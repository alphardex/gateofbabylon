title: 文字与气泡与粒子特效——玩转three.js的表面采样
author: alphardex
abbrlink: 8412
date: 2022-11-10 09:51:54
tags:
---
呀哈喽！这里是alphardex。

[MeshSurfaceSampler](https://threejs.org/docs/#examples/en/math/MeshSurfaceSampler)，是three.js的表面采样器，通过它，我们可以在一个`Mesh`的表面拾取一定数量的随机点位，从而实现一些炫酷的粒子特效，请看以下的效果（全屏最佳）：

https://code.juejin.cn/pen/7164008029766025223

本文将简要地介绍一下实现这个效果的思路

PS：本文标题neta了[《孤独摇滚》](https://bangumi.tv/subject/328609)的live曲[《吉他与孤独与蓝色星球》](https://www.bilibili.com/video/BV1sv4y1U7j6/?spm_id_from=333.337.search-card.all.click)

<!--more-->

## 表面采样

首先用kokomi.js的[Text3D](https://github.com/alphardex/kokomi.js/blob/main/src/shapes/text3D.ts)类来创建3D文字

```js
const font = await kokomi.loadFont();

const text = "J U E J I N";

const t3d = new kokomi.Text3D(
  this,
  text,
  font,
  {
    size: 0.6,
    height: 0.2,
    curveSegments: 120,
    bevelEnabled: true,
    bevelThickness: 0.03,
    bevelSize: 0.02,
    bevelOffset: 0,
    bevelSegments: 5,
  },
  {
    baseMaterial: new THREE.ShaderMaterial(),
    vertexShader,
    fragmentShader,
    materialParams: {
      side: THREE.DoubleSide,
    },
  }
);
t3d.mesh.geometry.center();
t3d.addExisting();
```

![1.png](https://s2.loli.net/2022/11/10/dSoEIYFzmL6k9ta.png)

利用kokomi.js封装好的函数[sampleParticlesPositionFromMesh](https://github.com/alphardex/kokomi.js/blob/main/src/utils/misc.ts#L48)来采样文字表面的点位数据

```js
const sampledPos = kokomi.sampleParticlesPositionFromMesh(
  t3d.mesh.geometry,
  8000
);
```

同时也获取下每个点位的下标

```js
const pIndexs = kokomi.makeBuffer(sampledPos.length / 3, (v, k) => v);
```

将点位位置数据和下标数据存储至`BufferGeometry`里

```js
const geometry = new THREE.BufferGeometry();
geometry.setAttribute("position", new THREE.BufferAttribute(sampledPos, 3));
geometry.setAttribute("pIndex", new THREE.BufferAttribute(pIndexs, 1));
```

新建一个微粒对象`THREE.Points`，并定义好`ShaderMaterial`

```js
const uj = new kokomi.UniformInjector(this);

const material = new THREE.ShaderMaterial({
  vertexShader,
  fragmentShader,
  side: THREE.DoubleSide,
  transparent: true,
  blending: THREE.AdditiveBlending,
  depthWrite: false,
  uniforms: {
    ...uj.shadertoyUniforms,
    uColor: {
      value: new THREE.Color("#d1a657"),
    },
    uPointSize: {
      value: 4,
    },
  },
});

this.update(() => {
  uj.injectShadertoyUniforms(material.uniforms);
});

const im = new THREE.Points(geometry, material);
this.scene.add(im);
```

顶点着色器里传入微粒大小

```glsl
uniform float iTime;
uniform vec2 iResolution;
uniform vec2 iMouse;

varying vec2 vUv;

uniform float uPointSize;

attribute float pIndex;

varying float vOpacity;
varying vec3 vPosition;

void main(){
    vec3 p=position;
    gl_Position=projectionMatrix*modelViewMatrix*vec4(p,1.);
    
    vUv=uv;
    gl_PointSize=uPointSize;
}
```

片元着色器里传入微粒颜色

```glsl
uniform float iTime;
uniform vec2 iResolution;
uniform vec2 iMouse;

varying vec2 vUv;

uniform vec3 uColor;

varying float vOpacity;
varying vec3 vPosition;

void main(){
    vec2 p=gl_PointCoord;
    
    vec3 col=uColor;
    
    gl_FragColor=vec4(col,1.);
}
```

![2.png](https://s2.loli.net/2022/11/10/MDue8s4O7Yw6Iqp.png)

可以看到被采样的点均匀地分布在了文字之上，cool！让它动起来吧！

## 气泡效果

首先编写片元着色器，利用`spot`函数来把微粒变成气泡状

```glsl
float shape=spot(p,.1,2.5);

gl_FragColor=vec4(col,shape);
```

将uPointSize微粒大小略微调大点，我们会看到下面的效果

![3.png](https://s2.loli.net/2022/11/10/wlEMPFgGsRKbydC.png)

接下来我们要在顶点着色器中变换顶点的y轴坐标，用到了随机函数和噪声函数，这里贴一下随机函数的实现吧，噪声函数请自行查看[Github链接](https://github.com/hughsk/glsl-noise/blob/master/classic/2d.glsl)

```glsl
float random(float n){
    return fract(sin(n)*43758.5453123);
}
```

```glsl
vec3 distort(vec3 p){
    float t=iTime*.25;
    float rndz=(random(pIndex)+cnoise(vec2(pIndex*.1,t*.5)));
    p.y+=fract((t+rndz)*.5);
    return p;
}

void main(){
    vec3 p=position;
    p=distort(p);
    gl_Position=projectionMatrix*modelViewMatrix*vec4(p,1.);

    vOpacity=random(pIndex);
    vPosition=p;
}
```

![4.gif](https://s2.loli.net/2022/11/10/rP6RMqnJZvkUyS8.gif)

在片元着色器中能够根据顶点着色器传递的`vPosition`和`vOpacity`来实现气泡的随机大小和透明度效果

```glsl
float saturate(float a){
    return clamp(a,0.,1.);
}

void main(){
    ...
    float alpha=1.-saturate(abs(vPosition.y*1.2));
    shape*=alpha;
    
    col*=vOpacity;
    ...
}
```

![5.gif](https://s2.loli.net/2022/11/10/KlVspSCXja78wUt.gif)

如此，气泡效果就基本实现了，有几点可以自行优化：

1. 将气泡动画和文字有机结合在一起
2. 给气泡赋予随机的颜色
3. 让文字变成可编辑的（提示：`contenteditable`）

## 最后

希望本文能给你创作新特效的灵感，keep creating~