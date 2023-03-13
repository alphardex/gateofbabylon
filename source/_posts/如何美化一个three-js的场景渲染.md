title: 如何美化一个three.js的场景渲染
author: alphardex
abbrlink: 44091
date: 2023-03-13 13:20:58
tags:
---
呀哈喽！这里是alphardex。

在使用three.js开发时，有时会感觉场景的渲染不是非常美观，本文就来一步一步地将一个不是很美观的场景重新焕发它的生机。

场景美化前：
![before.png](https://s2.loli.net/2023/03/13/V1T6srfquQcExZH.png)

场景美化后：
![after.png](https://s2.loli.net/2023/03/13/BXtsu21gZnMKS56.png)

<!--more-->

## 准备工作

首先在码上掘金上打开并fork这个初始场景

https://code.juejin.cn/pen/7209892060004876344

## 环境贴图

尽管已经有了2个光照，但模型还是显得非常暗，第一个任务就是要增加模型的亮度

该模型的主要材质是`MeshStandardMaterial`，它是一种基于[PBR](https://en.wikipedia.org/wiki/Physically_based_rendering)的材质，PBR材质比较容易受光照的影响，但效果最显著的还是要靠环境贴图

环境贴图往往就是360°的全景照片，通常可以在[polyhaven](https://polyhaven.com/hdris)网站上找到，这里选用了最常见的[potsdamer_platz](https://polyhaven.com/a/potsdamer_platz)来模拟城市的环境光

在[kokomi.AssetManager](https://github.com/alphardex/kokomi.js/blob/main/src/components/assetManager.ts)中加载HDR素材（加载的背后用到了[RGBELoader](https://threejs.org/examples/#webgl_loader_texture_hdr)）

```js
const am = new kokomi.AssetManager(
  this,
  [
    {
      name: "hdr",
      type: "hdrTexture",
      path: "https://kokomi-demo-1259280366.cos.ap-nanjing.myqcloud.com/potsdamer_platz_1k.hdr",
    },
    ...
  ],
  {
    useDracoLoader: true,
  }
);
```

用[THREE.PMREMGenerator](https://threejs.org/docs/#api/en/extras/PMREMGenerator)提取出envmap，应用到`scene.environment`即可

```js
const getEnvmapFromHDRTexture = (
  renderer,
  texture
) => {
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  pmremGenerator.compileEquirectangularShader();
  const envmap = pmremGenerator.fromEquirectangular(texture).texture;
  pmremGenerator.dispose();
  return envmap;
};
```

```js
const envMap = getEnvmapFromHDRTexture(this.renderer, am.items["hdr"]);
this.scene.environment = envMap;
```

![1.png](https://s2.loli.net/2023/03/13/nIUis375TL1BCDb.png)

## 输出编码

three.js默认的输出编码是线性编码（`THREE.LinearEncoding`），这种编码会使场景看上去偏暗淡一点

为了获得一个色彩更加明艳的场景，我们需要将编码改为`THREE.sRGBEncoding`

```js
this.renderer.outputEncoding = THREE.sRGBEncoding;
```

![2.png](https://s2.loli.net/2023/03/13/58XFutrPl9KDhqm.png)

## 后期处理

如果说以上的两步是必经之路的话，那这一步可以算是画龙点睛了

`Bloom`滤镜是最常用的滤镜之一，它可以照亮整个场景

`SMAA`滤镜也是很常用的滤镜，它能消除场景中的锯齿，尤其是对这种顶点很多的模型颇为有效

```js
this.scene.background = this.scene.background.convertSRGBToLinear();

const composer = new POSTPROCESSING.EffectComposer(this.renderer, {
  frameBufferType: THREE.HalfFloatType,
  multisampling: 8,
});
this.composer = composer;

composer.addPass(new POSTPROCESSING.RenderPass(this.scene, this.camera));

const bloom = new POSTPROCESSING.BloomEffect({
  blendFunction: POSTPROCESSING.BlendFunction.ADD,
  mipmapBlur: true,
  luminanceThreshold: 1,
});

const smaa = new POSTPROCESSING.SMAAEffect();

const effectPass = new POSTPROCESSING.EffectPass(this.camera, bloom, smaa);
composer.addPass(effectPass);

this.renderer.autoClear = true;
```

使用了它们后，会发现场景除了无锯齿外并没有什么明显的变化，别急，好戏还在后头~

在这个场景中，车灯、轮胎灯、尾灯都用到了自发光贴图`emissive map`，而这个贴图的显示效果跟材质的某个参数息息相关，这个参数是[emissiveIntensity](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial.emissiveIntensity)，将它调高就会使自发光的效果更加显著

此外，我们也要将材质的[tonemapping](https://threejs.org/docs/?q=material#api/en/materials/Material.toneMapped)给关闭，因为tonemapping会自动把颜色的RGB值限制在0-1内，达不到自发光的要求，解除了这个限制后，我们就会发现那些地方亮起来了

还有一个注意点：在`Bloom`滤镜中，我们把`luminanceThreshold`设为了1，这是为了防止场景的其他部分发亮，仅仅让那些有`emissive map`的材质发亮

![3.png](https://s2.loli.net/2023/03/13/AudFyfhP5OYM2qC.png)

## 完成代码

https://code.juejin.cn/pen/7209889563153481765