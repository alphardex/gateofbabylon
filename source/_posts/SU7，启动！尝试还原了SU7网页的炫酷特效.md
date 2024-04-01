title: SU7，启动！尝试还原了 SU7 网页的炫酷特效
author: alphardex
abbrlink: 17935
date: 2024-03-28 08:24:49
tags:

---

这里是 alphardex。

最近看到有小伙伴发了一个小米 SU7 的展示网页，那是相当的酷炫：

![intro](https://s2.loli.net/2024/03/28/NBucKfVrZM3FD71.png)

光看上面的图可能看不出来什么，请直接访问原网页：https://gamemcu.com/su7/

当你体验一遍后，会看到各种炫酷的效果，包括隧道穿梭、波浪动画等，并且还有些细节也值得注意，如地面的反射效果。

那么这个网页是用哪个技术实现的呢？答案是`three.js`，不论是从控制台，还是一些技术解析插件都能得知。

但是，这里我要研究的重点却并非`three.js`本身。仅仅用`three.js`展示一个`3D`模型，相信对大家来说并不是很难。真正的难点，是那些炫酷的效果是如何实现的。想要实现那些效果，就得靠`three.js`背后的功臣——`Shader`，也就是着色器。

为了进一步地加强`three.js`和`Shader`的学习，我决定爆肝几天，亲手把那些炫酷的效果给还原出来。

经过一定的努力，总算是写出来了。不多说，放地址：https://su7-replica.netlify.app/

对整体流程做了一个简化：入场动画过后，无需长按，点击汽车即可直接进行冲刺（方便直接录制动图或视频），再次点击汽车即可取消冲刺，返回原来的状态。

<!--more-->

## 用到的库

除了最基本的`three.js`外，我还用到了这些库：

- [kokomi.js](https://github.com/alphardex/kokomi.js)：我基于`three.js`二次封装的框架。
- [lygia](https://github.com/patriciogonzalezvivo/lygia)：一个有着很多实用`Shader`函数的库。
- [postprocessing](https://github.com/pmndrs/postprocessing)：一个有着很多后期处理（滤镜）效果的框架。
- [gsap](https://github.com/greensock/GSAP)：一个很常用的`JS`缓动动画库。

## 基础场景

最初的场景是这样的。

![1](https://s2.loli.net/2024/03/28/VGTbQeoHCNUW8kM.png)

小米 SU7 的模型在`Sketchfab`上有：https://sketchfab.com/3d-models/su7-7296a91633d74c6eb113010e2ed75eda

## 环境贴图切换

当你刚进入页面时，会看到车子从夜间色调转变为白天色调的过程。

![2-3](https://s2.loli.net/2024/03/28/hkuNWoriE17Fz2e.png)

这里并没有用到任何光照，而是用的环境贴图。

如果你用过`three.js`，就知道它的环境贴图默认只支持一张，要两张或者更多张之间切换的话，就要用到`FBO`。

`FBO`是什么？简言之就是一个可以作为贴图的场景。由于场景是可以动态渲染的，因此贴图本身也能随时改变，具有非常大的灵活性。

创建一个`FBO`，并且在里面放一个占满全屏的平面，在这个平面里对环境贴图进行采样，再用`mix`插值函数负责切换，将`FBO`的渲染结果设为场景的`environment`即可。

以下是`JS`部分的逻辑实现：

```ts
class DynamicEnv extends kokomi.Component {
  constructor(base: Experience, config: Partial<DynamicEnvConfig> = {}) {
    super(base);

    const { envmap1, envmap2 } = config;

    const envData = envmap1?.source.data;

    const fbo = new kokomi.FBO(this.base, {
      width: envData.width,
      height: envData.height,
    });
    this.fbo = fbo;

    this.envmap.mapping = THREE.CubeUVReflectionMapping;

    const material = new THREE.ShaderMaterial({
      vertexShader: dynamicEnvVertexShader,
      fragmentShader: dynamicEnvFragmentShader,
      uniforms: {
        uEnvmap1: {
          value: envmap1,
        },
        uEnvmap2: {
          value: envmap2,
        },
        uWeight: {
          value: 0,
        },
        uIntensity: {
          value: 1,
        },
      },
    });
    this.material = material;

    const quad = new kokomi.FullScreenQuad(material);
    this.quad = quad;
  }
  update() {
    this.base.renderer.setRenderTarget(this.fbo.rt);
    this.quad.render(this.base.renderer);
    this.base.renderer.setRenderTarget(null);
  }
}
```

`FBO`里的全屏平面的片元着色器：

```glsl
uniform sampler2D uEnvmap1;
uniform sampler2D uEnvmap2;
uniform float uWeight;
uniform float uIntensity;

void main(){
    vec2 uv=vUv;
    vec3 envmap1=texture(uEnvmap1,uv).xyz;
    vec3 envmap2=texture(uEnvmap2,uv).xyz;
    vec3 col=mix(envmap1,envmap2,uWeight)*uIntensity;
    gl_FragColor=vec4(col,1.);
}
```

过渡效果如下：

![3](https://s2.loli.net/2024/04/01/yQBf1Y5vi762AsL.gif)

## 反光地面

一开始我想偷懒，直接用`kokomi.js`的[反射材质](https://github.com/alphardex/kokomi.js/blob/main/src/materials/meshReflectorMaterial.ts)来实现。

![4](https://s2.loli.net/2024/03/28/mQPhOqILWaos28Y.png)

但这并不是我真正想要的效果，甚至连原来的地面贴图都丢掉了，因此只好自己在`kokomi.js`反射材质的基础上实现一个自定义的反射材质。

这里我捣鼓了很久，发现在原`geometry`的基础上始终达不到很好的反射效果，只好把原网站上反射向量运算的逻辑给搬了过来，挺复杂的，大家看看就好。

![carbon](https://s2.loli.net/2024/03/30/UDkrQNKeZgISCuP.png)

我们主要关心 2 个变量：反射矩阵`_reflectMatrix`和反射贴图`_renderTexture`，利用这两个变量就能产生反光的效果。

用自定义材质`kokomi.CustomShaderMaterial`在之前的地面材质的基础上进行自定义`Shader`，并把上面提到的那两个变量放进去。

> 自定义材质借用了这个库：https://github.com/FarazzShaikh/THREE-CustomShaderMaterial

顶点着色器中计算好`vWorldPosition`并作为`varying`传给片元着色器。

```glsl
uniform float iTime;
uniform vec2 iResolution;
uniform vec2 iMouse;

varying vec2 vUv_;
varying vec4 vWorldPosition;

void main(){
    vec3 p=position;

    csm_Position=p;

    vUv_=uv;
    vWorldPosition=modelMatrix*vec4(p,1);
}
```

片元着色器中直接在原来颜色的基础上加上反光的颜色。

```glsl
uniform float iTime;
uniform vec2 iResolution;
uniform vec2 iMouse;

varying vec2 vUv_;
varying vec4 vWorldPosition;

uniform vec3 uColor;
uniform mat4 uReflectMatrix;
uniform sampler2D uReflectTexture;
uniform float uReflectIntensity;

void main(){
    vec2 p=vUv_;

    vec4 reflectPoint=uReflectMatrix*vWorldPosition;
    reflectPoint=reflectPoint/reflectPoint.w;
    vec3 reflectionSample=texture(uReflectTexture,reflectPoint.xy).xyz;
    reflectionSample*=uReflectIntensity;

    vec3 col=uColor;
    col+=reflectionSample;

    csm_DiffuseColor=vec4(col,1.);
}
```

![5](https://s2.loli.net/2024/03/29/VuCrBpJ3GRIlwL2.png)

Cool！

尽管挺酷，但还缺少一种粗糙的质感。

我们需要给它加上法线贴图，先解包贴图吧。

```glsl
vec3 surfaceNormal=texture(normalMap,vWorldPosition.xz).rgb*2.-1.;
surfaceNormal=normalize(surfaceNormal);

...

col=surfaceNormal;
```

![6](https://s2.loli.net/2024/03/29/jk8MCz3QqFdeiyx.png)

很明显法线的方向不对（显示的是蓝色，对应`RGB`中的`B`通道，但`y`轴理应是`G`通道），让我们调换下`G`通道和`B`通道吧。

```glsl
vec3 surfaceNormal=texture(normalMap,vWorldPosition.xz).rgb*2.-1.;
surfaceNormal=surfaceNormal.rbg;
surfaceNormal=normalize(surfaceNormal);
```

![7](https://s2.loli.net/2024/03/29/Ah53BsRYrt8ofOS.png)

这下法线的方向对了，我们可以给反射`UV`增加一点偏移量来扭曲它。

```glsl
vec3 viewDir=vViewPosition;
float d=length(viewDir);
viewDir=normalize(viewDir);

vec2 distortion=surfaceNormal.xz*(.001+1./d);

...
vec3 reflectionSample=texture(uReflectTexture,reflectPoint.xy+distortion).xyz;
```

![8](https://s2.loli.net/2024/03/29/EfUMrVPQp8CGRBN.png)

尽管变化不算很大，但看上去也不错。

这里有个小问题：当你把相机往上调时，会发现反光变得更明显了，有点不怎么真实。

![9](https://s2.loli.net/2024/03/29/TOkYzC9Lsc6pDVt.png)

我们可以用菲涅尔反射来作为原颜色与反光色的混合因子。

```glsl
#include "/node_modules/lygia/lighting/fresnel.glsl"

vec3 col=uColor;
// col+=reflectionSample;
vec3 fres=fresnel(vec3(0.),vNormal,viewDir);
col=mix(col,reflectionSample,fres);
```

![10](https://s2.loli.net/2024/03/29/T9wJHdIa8cOu4sF.png)

这样反光就显得自然多了。

尽管如此，我觉得在初始的视觉下，反光还是太明显，可以用一些手段来将它模糊化一下。

我在之前写的一篇[模拟真实的下雨效果的文章](https://juejin.cn/post/7200443454567137336)里介绍过这么一种方法——`自定义mipmap`，在这里我们也可以将它应用到反射材质上。

```glsl
#include "/node_modules/lygia/sample/bicubic.glsl"

// 这里省略了packedTexture2DLOD函数，原文章中有，直接复制过来即可

...
uniform vec2 uMipmapTextureSize;

void main(){
    ...

    // vec3 reflectionSample=texture(uReflectTexture,reflectPoint.xy+distortion).xyz;
    float roughnessValue=texture(roughnessMap,vWorldPosition.xz).r;
    roughnessValue=roughnessValue*(1.7-.7*roughnessValue);
    roughnessValue*=4.;
    float level=roughnessValue;
    vec2 finalUv=reflectPoint.xy+distortion;
    vec3 reflectionSample=packedTexture2DLOD(uReflectTexture,finalUv,level,uMipmapTextureSize).rgb;
    reflectionSample*=uReflectIntensity;

    vec3 col=uColor;
    // col+=reflectionSample;
    col*=3.;
    vec3 fres=fresnel(vec3(0.),vNormal,viewDir);
    col=mix(col,reflectionSample,fres);

    csm_DiffuseColor=vec4(col,1.);
}
```

我们通过粗糙度来控制`mipmap`的`level`，这样就能达成一种带粗糙的模糊效果。

顺便也将原来的颜色调亮了一点。

![11](https://s2.loli.net/2024/03/29/StPBakTlFwcM6bR.png)

到这里，我觉得反光地面效果基本算是完成了。

## 隧道穿梭

这个网站最吸引我的，无非就是长按鼠标时的隧道穿梭效果，可以说是最惊艳的一个效果了。

初见我还以为这个效果用到了[实例化网格 InstancedMesh](https://threejs.org/docs/#api/en/objects/InstancedMesh)，因为有大量的线条在运动，后来我仔细一看才发现这些线条是紧贴在一个圆柱的表面上运动的，并且是扁平的，不像是实体的网格`Mesh`，就排除了这种可能。

既然不是实例化网格，那么想要模拟出这种效果，只有可能是像`Shadertoy`上那种的纯片元着色器动画了。

OK，让我们先从彩色的线条开始吧，由于这些线条的位置和颜色都是随机的，因此必定会用到随机函数，并且它们的分布也有一定的混沌性，估计噪声函数应该也用到了。

`iq`曾在 Shadertoy 上发过一个很常用的`Simplex`噪声函数，我们把它直接复制过来吧。

```glsl
// https://www.shadertoy.com/view/Msf3WH
vec2 hash(vec2 p)// replace this by something better
{
    p=vec2(dot(p,vec2(127.1,311.7)),dot(p,vec2(269.5,183.3)));
    return-1.+2.*fract(sin(p)*43758.5453123);
}

float noise(in vec2 p)
{
    const float K1=.366025404;// (sqrt(3)-1)/2;
    const float K2=.211324865;// (3-sqrt(3))/6;

    vec2 i=floor(p+(p.x+p.y)*K1);
    vec2 a=p-i+(i.x+i.y)*K2;
    float m=step(a.y,a.x);
    vec2 o=vec2(m,1.-m);
    vec2 b=a-o+K2;
    vec2 c=a-1.+2.*K2;
    vec3 h=max(.5-vec3(dot(a,a),dot(b,b),dot(c,c)),0.);
    vec3 n=h*h*h*h*vec3(dot(a,hash(i+0.)),dot(b,hash(i+o)),dot(c,hash(i+1.)));
    return dot(n,vec3(70.));
}
```

从最简单的纯色开始。

```glsl
void main(){
    vec2 uv=vUv;

    vec3 col=vec3(1.);

    float mask=1.;

    gl_FragColor=vec4(col,mask);
}
```

![12](https://s2.loli.net/2024/03/29/3UPl5kvuCNxi4Xm.png)

先给遮罩直接应用噪声试试。

```glsl
vec2 noiseUv=uv;
float noiseValue=noise(noiseUv*vec2(100.));
mask=noiseValue;
```

![13](https://s2.loli.net/2024/03/29/OsJ2BgPHVTZ91C5.png)

图案过于点状了，如果想要线条状的话，就要把点给“拉长”，我们把`x`的值设的稍微小一点吧。

```glsl
vec2 noiseUv=uv;
float noiseValue=noise(noiseUv*vec2(3.,100.));
mask=noiseValue;
```

![14](https://s2.loli.net/2024/03/29/ma1bru7qPjIYtDo.png)

目前噪声的值域是`[-1,1]`，为了方便后续的计算，我们用`map`函数把它的值域映射到`[0,1]`。

```glsl
#include "/node_modules/lygia/math/map.glsl"

...
mask=map(mask,-1.,1.,0.,1.);
```

![15](https://s2.loli.net/2024/03/29/eIHOo92x1rYgXuA.png)

每个线条看上去都很大，我们可以用`pow`函数来让它们在大小上产生指数级的变化。

```glsl
mask=pow(mask,5.);
```

![16](https://s2.loli.net/2024/03/29/zYZqcxl8kT7p46P.png)

此时的线看上去比较模糊，我们用`smoothstep`函数来把它的实体给勾画出来吧。

```glsl
mask=smoothstep(0.,.04,mask);
```

![17](https://s2.loli.net/2024/03/29/7dQ64I2B5qfpasb.png)

线条太粗了，我们加大`pow`的力度，再顺便减一点值，就能让它变细。

```glsl
#include "/node_modules/lygia/math/saturate.glsl"

...
mask=pow(saturate(mask-.1),11.);
mask=smoothstep(0.,.04,mask);
```

![18](https://s2.loli.net/2024/03/29/HCfc4kRyFP9wuS7.png)

是不是已经有速度线的感觉了呢？

形状是有了，接下来我们开始让颜色也变得随机化起来。

我们可以先实验一下`Shadertoy`里面常用的一个颜色方案[cos Palette](https://iquilezles.org/articles/palettes/)，这里我选择了彩色。

![19](https://s2.loli.net/2024/03/30/z3Sf1hUq2ajrEOg.png)

```glsl
col=palette(mask,vec3(.5),vec3(.5),vec3(1.),vec3(0.,.33,.67));
```

![20](https://s2.loli.net/2024/03/30/zDcxn2VdW7oeATk.png)

感觉不太好看，再调整下颜色的通道值吧。

```glsl
col*=vec3(1.5,1.,400.);
```

![21](https://s2.loli.net/2024/03/30/up3bqQhcZwen4y1.png)

有点那味了！但跟原效果比还是有一点视觉上的差距。

把这一行注释掉吧。

```glsl
// col=palette(mask,vec3(.5),vec3(.5),vec3(1.),vec3(0.,.33,.67));
```

那么不用`cos Palette`方案，能不能让颜色也随机化呢？当然可以！

让我们引入`Shader`常用的随机函数`random`，并在此基础上封装这样一个函数：它能在 3 个颜色通道上生成各不相同的随机值，从而产生随机的颜色。

```glsl
#include "/node_modules/lygia/generative/random.glsl"

vec3 pos2col(vec2 i){
    i+=vec2(9.,0.);

    float r=random(i+vec2(12.,2.));
    float g=random(i+vec2(7.,5.));
    float b=random(i);

    vec3 col=vec3(r,g,b);
    return col;
}
```

光随机函数还不够，我们需要一个噪声函数。

由于并没有能够生成随机颜色的噪声函数，我们需要用双线性插值法来自己封装一个。

什么是双线性插值法呢？其实就是我们采样纹理时`GPU`所使用的一个方法。

假设有四个随机生成的值，你想从它们的“包围圈”中取某一个值，怎么做呢？

![22](https://s2.loli.net/2024/03/30/HKNPUIlXJrTSzOw.png)

先在上面的两个值之间用插值法求出一个值，再在下面的两个值之间用插值法求出一个值，最后在这两个值之间再用一次插值法即可求出最终的值。

利用这个方法，我们就可以封装出能生成随机颜色的噪声函数。

```glsl
vec3 colorNoise(vec2 uv){
    vec2 size=vec2(1.);
    vec2 pc=uv*size;
    vec2 base=floor(pc);

    vec3 v1=pos2col((base+vec2(0.,0.))/size);
    vec3 v2=pos2col((base+vec2(1.,0.))/size);
    vec3 v3=pos2col((base+vec2(0.,1.))/size);
    vec3 v4=pos2col((base+vec2(1.,1.))/size);

    vec2 f=fract(pc);

    f=smoothstep(0.,1.,f);

    vec3 px1=mix(v1,v2,f.x);
    vec3 px2=mix(v3,v4,f.x);
    vec3 v=mix(px1,px2,f.y);
    return v;
}
```

来调用下看看结果吧。（这里顺便把遮罩调成 1）

```glsl
col=colorNoise(noiseUv*vec2(10.,100.));
col*=vec3(1.5,1.,400.);
mask=1.;
```

![23](https://s2.loli.net/2024/03/30/VvxDqiwHShO8IjG.png)

随机色有了，我们用`smoothstep`函数把某些地方给虚化掉。

```glsl
mask*=smoothstep(.02,.5,uv.x)*smoothstep(.02,.5,1.-uv.x);
mask*=smoothstep(.01,.1,uv.y)*smoothstep(.01,.1,1.-uv.y);
```

![24](https://s2.loli.net/2024/03/30/FWs72NGMgjp48Kk.png)

上图中红圈圈出来的是虚化掉的部分，由于隧道有两头和两侧，`smoothstep`时我们要乘以`uv`取反后的值`1.-uv`。

把`mask=1.;`这一行给注释掉吧。

![25](https://s2.loli.net/2024/03/30/KsbWtkc9NuUCvBM.png)

感觉线条还是不够多，我给画面加上了`Bloom`滤镜，能使值超过 1 的颜色显示得更多。

![26](https://s2.loli.net/2024/03/30/Tr5BR9zngEKGWNJ.png)

顺便把之前的反光地面给整回来吧，把顶部的灯给隐藏掉，再把地面的主色调`uColor`改成黑色。

![27](https://s2.loli.net/2024/03/30/x6en84BGtrPqKvw.png)

要让这些线条动起来，给`noiseUv`的`x`轴加上时间偏移量即可。

```glsl
vec2 noiseUv=uv;
noiseUv.x+=-iTime*.5;
```

同时让地面的纹理也动起来吧。

```glsl
vec2 surfaceNormalUv=vWorldPosition.xz;
surfaceNormalUv.x+=iTime*10.;
vec3 surfaceNormal=texture(normalMap,surfaceNormalUv).rgb*2.-1.;

...

vec2 roughnessUv=vWorldPosition.xz;
roughnessUv.x+=iTime*10.;
float roughnessValue=texture(roughnessMap,roughnessUv).r;
```

再优化下，环境要变暗些，车身的`envMapIntensity`要降低。

别忘了让轮子也动起来~

![28](https://s2.loli.net/2024/03/30/IULayh367ruWCAO.gif)

如果想控制速度，可以弄一个统一的速度变量`uSpeed`，再分别应用到位移的部分即可。

到这里效果就算基本完成了，但总感觉还是少了些什么。没错！是流光特效以及相机晃动的效果，没有了它们感觉这个效果就少了灵魂。

### 流光特效

乍一看有点难实现，但其实是最简单的一个。

创建一个立方体`FBO`和一个`CubeCamera`，把当前场景直接放到`CubeCamera`中渲染，再将环境贴图设为立方体`FBO`的渲染结果贴图即可。

`kokomi.js`封装了一个[kokomi.Environment](https://github.com/alphardex/kokomi.js/blob/main/src/lights/environment.ts)类，正好可以用在这里。

```ts
const environment = new kokomi.Environment(this.base, {
  resolution: 512,
  scene: this.base.scene,
  options: {
    minFilter: THREE.LinearMipMapLinearFilter,
    anisotropy: 0,
    depthBuffer: false,
    generateMipmaps: true,
  },
  textureType: THREE.UnsignedByteType,
});
this.environment = environment;
this.base.scene.environment = this.environment.texture;
```

![29](https://s2.loli.net/2024/04/01/2K7LWtGcEyb5DIl.png)

### 相机晃动

想要晃动相机，我们就要实时地给相机的位置加上某个动态生成的偏移量。

这个偏移量可以是一个`xyz`都是随机值的向量，我决定用噪声来实现，在`JS`里比较常用的噪声库是[simplex-noise.js](https://github.com/jwagner/simplex-noise.js)。

```ts
import { createNoise2D } from "simplex-noise";

const noise2d = createNoise2D();
```

为了让结果更加混沌，可以使用分形布朗运动`FBM`。

`FBM`简言之就是将多个噪声叠加起来，并且叠加的同时升高频率，降低振幅。

```ts
const fbm = ({
  octave = 3,
  frequency = 2,
  amplitude = 0.5,
  lacunarity = 2,
  persistance = 0.5,
} = {}) => {
  let value = 0;
  for (let i = 0; i < octave; i++) {
    const noiseValue = noise2d(frequency, frequency);
    value += noiseValue * amplitude;
    frequency *= lacunarity;
    amplitude *= persistance;
  }
  return value;
};
```

晃动效果的主要实现逻辑如下：

```ts
class CameraShake extends kokomi.Component {
  constructor(base: Experience, config: Partial<CameraShakeConfig> = {}) {
    super(base);

    const { intensity = 1 } = config;
    this.intensity = intensity;

    const tweenedPosOffset = new THREE.Vector3(0, 0, 0);
    this.tweenedPosOffset = tweenedPosOffset;
  }
  update(): void {
    const t = this.base.clock.elapsedTime;
    const posOffset = new THREE.Vector3(
      fbm({
        frequency: t * 0.5 + THREE.MathUtils.randFloat(-10000, 0),
        amplitude: 2,
      }),
      fbm({
        frequency: t * 0.5 + THREE.MathUtils.randFloat(-10000, 0),
        amplitude: 2,
      }),
      fbm({
        frequency: t * 0.5 + THREE.MathUtils.randFloat(-10000, 0),
        amplitude: 2,
      })
    );
    posOffset.multiplyScalar(0.1 * this.intensity);
    gsap.to(this.tweenedPosOffset, {
      x: posOffset.x,
      y: posOffset.y,
      z: posOffset.z,
      duration: 1.2,
    });
    this.base.camera.position.add(this.tweenedPosOffset);
  }
}
```

用`FBM`在 3 个轴上分别产生一个随机值，并且这些值的频率也是随机错开的，默认生成的值会很大，要适当缩小一点，再给最后的目标值加上一个缓动的变化，就能得到一个相当不错的相机晃动效果了。

## 最终效果

![final](https://s2.loli.net/2024/03/31/OzfpV5HNl8W6CJv.gif)

## 源码

Github 地址：https://github.com/alphardex/su7-replica

## 最后

如果还有其他疑问或者想说的话，随时欢迎在微信上与我联系，我的微信号：`Blacklurker`。

我是`alphardex`，我们下次再见！

_Thank you._
