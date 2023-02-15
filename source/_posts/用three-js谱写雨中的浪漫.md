title: 用three.js谱写雨中的浪漫
author: alphardex
abbrlink: 5999
date: 2023-02-15 16:23:21
tags:
---
呀哈喽！这里是alphardex。

https://code.juejin.cn/pen/7200287096689393720

<!--more-->

## 建模

首先我们需要一些贴图素材

贴图素材一般可以在[3dtextures](https://3dtextures.me/)网站上找到，这里我找了2份，包含了墙的法线贴图和潮湿地面的法线、透明度、粗糙度贴图

通过`kokomi.AssetManager`将贴图素材一次性全部加载出来，将它们应用到`Mesh`上，加上基本的环境光照，即可完成最基本的建模

```js
// 光照
const pointLight1 = new THREE.PointLight(config.color, 0.5, 17, 0.8);
pointLight1.position.set(0, 2, 0);
this.scene.add(pointLight1);
...

// 网格
const aspTex = am.items["asphalt-normal"];
aspTex.rotation = THREE.MathUtils.degToRad(90);
aspTex.wrapS = aspTex.wrapT = THREE.RepeatWrapping;
aspTex.repeat.set(5, 8);

const wallMat = new THREE.MeshPhongMaterial({
  color: new THREE.Color("#111111"),
  normalMap: aspTex,
  normalScale: new THREE.Vector2(0.5, 0.5),
  shininess: 200,
});

const wall = new THREE.Mesh(new THREE.BoxGeometry(25, 20, 0.5), wallMat);
this.scene.add(wall);
wall.position.y = 10;
wall.position.z = -10.3;
...

// 文字
const t3d = new kokomi.Text3D(this, config.text, font, {
  size: 3,
  height: 0.2,
  curveSegments: 120,
  bevelEnabled: false,
});
t3d.mesh.geometry.center();

const tm = new THREE.Mesh(
  t3d.mesh.geometry,
  new THREE.MeshBasicMaterial({
    color: config.color,
  })
);
this.scene.add(tm);
tm.position.y = 1.54;
```

![1.jpg](https://s2.loli.net/2023/02/15/wMHvm2rUXRojbg8.jpg)

## 地面

![2.gif](https://s2.loli.net/2023/02/15/PRxHYhcEk7ydgCJ.gif)

## 雨

![3.gif](https://s2.loli.net/2023/02/15/oH2gnGqXftYeBFJ.gif)

## 灯光闪烁

```js
// flicker
const turnOffLight = () => {
  tm.material.color.copy(new THREE.Color("black"));
  pointLight1.color.copy(new THREE.Color("black"));
};

const turnOnLight = () => {
  tm.material.color.copy(new THREE.Color(config.color));
  pointLight1.color.copy(new THREE.Color(config.color));
};

let flickerTimer = null;

const flicker = () => {
  flickerTimer = setInterval(async () => {
    const rate = Math.random();
    if (rate < 0.5) {
      turnOffLight();
      await kokomi.sleep(200 * Math.random());
      turnOnLight();
      await kokomi.sleep(200 * Math.random());
      turnOffLight();
      await kokomi.sleep(200 * Math.random());
      turnOnLight();
    }
  }, 3000);
};

flicker();
```

![4.gif](https://s2.loli.net/2023/02/15/zm8vM9TA5eCS1b6.gif)

## 后期处理

```js
// postprocessing
const composer = new POSTPROCESSING.EffectComposer(this.renderer);
this.composer = composer;

composer.addPass(new POSTPROCESSING.RenderPass(this.scene, this.camera));

// bloom
const bloom = new POSTPROCESSING.BloomEffect({
  luminanceThreshold: 0.4,
  luminanceSmoothing: 0,
  mipmapBlur: true,
  intensity: 2,
  radius: 0.4,
});
composer.addPass(new POSTPROCESSING.EffectPass(this.camera, bloom));

// antialiasing
const smaa = new POSTPROCESSING.SMAAEffect();
composer.addPass(new POSTPROCESSING.EffectPass(this.camera, smaa));
```

![5.gif](https://s2.loli.net/2023/02/15/ObcQdI1wANFyk9J.gif)