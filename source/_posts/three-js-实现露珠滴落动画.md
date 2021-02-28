title: three.js 实现露珠滴落动画
author: alphardex
abbrlink: 9615
tags:
  - three.js
  - JavaScript
categories:
  - 前端
date: 2021-02-28 09:17:00
---
## 前言

大家好，这里是 CSS 魔法使——alphardex。

本文我们将用three.js来实现一种很酷的光学效果——露珠滴落。我们知道，在露珠从一个物体表面滴落的时候，会产生一种粘着的效果。2D平面中，这种粘着效果其实用css滤镜就可以轻松实现。但是到了3D世界，就没那么简单了，这时我们就得依靠光照来实现，其中涉及到了一个关键算法——光线步进（Ray Marching）。以下是最终实现的效果图

![ray-marching.gif](https://i.loli.net/2021/02/28/1Zs4rTinGw2PbtU.gif)

撒，哈吉马路由！

<!--more-->

## 准备工作

笔者的[three.js模板](https://codepen.io/alphardex/pen/yLaQdOq)：点击右下角的fork即可复制一份

## 正片

### 全屏相机

首先将相机换成正交相机，再将平面的长度调整为2，使其填满屏幕

```ts
class RayMarching extends Base {
  constructor(sel: string, debug: boolean) {
    super(sel, debug);
    this.clock = new THREE.Clock();
    this.cameraPosition = new THREE.Vector3(0, 0, 0);
    this.orthographicCameraParams = {
      left: -1,
      right: 1,
      top: 1,
      bottom: -1,
      near: 0,
      far: 1,
      zoom: 1
    };
  }
  // 初始化
  init() {
    this.createScene();
    this.createOrthographicCamera();
    this.createRenderer();
    this.createRayMarchingMaterial();
    this.createPlane();
    this.createLight();
    this.trackMousePos();
    this.addListeners();
    this.setLoop();
  }
  // 创建平面
  createPlane() {
    const geometry = new THREE.PlaneBufferGeometry(2, 2, 100, 100);
    const material = this.rayMarchingMaterial;
    this.createMesh({
      geometry,
      material
    });
  }
}
```

![1](https://i.loli.net/2021/02/28/dXi7IsblwHpJgvu.png)

### 创建材质

创建好着色器材质，里面定义好所有要传递给着色器的参数

```ts
const matcapTextureUrl = "https://i.loli.net/2021/02/27/7zhBySIYxEqUFW3.png";

class RayMarching extends Base {
  // 创建光线追踪材质
  createRayMarchingMaterial() {
    const loader = new THREE.TextureLoader();
    const texture = loader.load(matcapTextureUrl);
    const rayMarchingMaterial = new THREE.ShaderMaterial({
      vertexShader: rayMarchingVertexShader,
      fragmentShader: rayMarchingFragmentShader,
      side: THREE.DoubleSide,
      uniforms: {
        uTime: {
          value: 0
        },
        uMouse: {
          value: new THREE.Vector2(0, 0)
        },
        uResolution: {
          value: new THREE.Vector2(window.innerWidth, window.innerHeight)
        },
        uTexture: {
          value: texture
        },
        uProgress: {
          value: 1
        },
        uVelocityBox: {
          value: 0.25
        },
        uVelocitySphere: {
          value: 0.5
        },
        uAngle: {
          value: 1.5
        },
        uDistance: {
          value: 1.2
        }
      }
    });
    this.rayMarchingMaterial = rayMarchingMaterial;
  }
}
```

顶点着色器`rayMarchingVertexShader`，这个只要用模板现成的就可以了

重点是片元着色器`rayMarchingFragmentShader`

### 片元着色器

#### 背景

作为热身运动，先创建一个辐射状的背景吧

```glsl
varying vec2 vUv;

vec3 background(vec2 uv){
    float dist=length(uv-vec2(.5));
    vec3 bg=mix(vec3(.3),vec3(.0),dist);
    return bg;
}

void main(){
    vec3 bg=background(vUv);
    vec3 color=bg;
    gl_FragColor=vec4(color,1.);
}
```

![2](https://i.loli.net/2021/02/28/BfCUSgAJVoj1Riy.png)

#### sdf

如何在光照模型中创建物体呢？我们需要sdf。

sdf的意思是符号距离函数：若传递给函数空间中的某个坐标，则返回那个点与某些平面之间的最短距离，返回值的符号表示点在平面的内部还是外部，故称符号距离函数。

如果我们要创建一个球，就得用球的sdf来创建。球体方程可以用如下的glsl代码来表示

```glsl
float sdSphere(vec3 p,float r)
{
    return length(p)-r;
}
```

方块的代码如下

```glsl
float sdBox(vec3 p,vec3 b)
{
    vec3 q=abs(p)-b;
    return length(max(q,0.))+min(max(q.x,max(q.y,q.z)),0.);
}
```

看不懂怎么办？没关系，国外已经有大牛把[常用的sdf公式](https://www.iquilezles.org/www/articles/distfunctions/distfunctions.htm)都整理出来了

在sdf里先创建一个方块

```glsl
float sdf(vec3 p){
    float box=sdBox(p,vec3(.3));
    return box;
}
```

画面上仍旧一片空白，因为我们的嘉宾——光线还尚未入场。

#### 光线步进酱

接下来就是本文的头号人物——光线步进酱了。在介绍光线步进酱之前，我们先来看看她的好姬友光线追踪吧。

![raytrace.png](https://i.loli.net/2021/02/28/8hrikXo9eTRtHcw.png)

首先，我们需要知道光线追踪是如何进行的：给相机一个位置`eye`，在前面放一个网格，从相机的位置发射一束射线`ray`，穿过网格打在物体上，所成的像的每一个像素对应着网格上的每一个点。

而在光线步进中，整个场景会由一系列的sdf的角度定义。为了找到场景和视线之间的边界，我们会从相机的位置开始，沿着射线，一点一点地移动每个点，每一步都会判断这个点在不在场景的某个表面内部，如果在则完成，表示光线击中了某东西，如果不在则光线继续步进。

![spheretrace.jpg](https://i.loli.net/2021/02/28/523sMyGvTu4BLYd.jpg)

上图中，p0是相机位置，蓝色的线代表射线。可以看出光线的第一步p0p1就迈的非常大，它也恰好是这一步到场景表面的最近距离。表面上的点尽管是最近距离，但并没有沿着视线的方向，因此要继续检测到p4这个点

以下是光线步进酱的glsl代码实现

```glsl
const float EPSILON=.0001;

float rayMarch(vec3 eye,vec3 ray,float end,int maxIter){
    float depth=0.;
    for(int i=0;i<maxIter;i++){
        vec3 pos=eye+depth*ray;
        float dist=sdf(pos);
        depth+=dist;
        if(dist<EPSILON||dist>=end){
            break;
        }
    }
    return depth;
}
```

在主函数中创建一条射线，将其投喂给光线步进酱，即可获得光线行走的距离啦

```glsl
void main(){
    ...
    vec3 eye=vec3(0.,0.,2.5);
    vec3 ray=normalize(vec3(vUv,-eye.z));
    float end=5.;
    int maxIter=256;
    float depth=rayMarch(eye,ray,end,maxIter);
    if(depth<end){
      vec3 pos=eye+depth*ray;
      color=pos;
    }
    ...
}
```

![3](https://i.loli.net/2021/02/28/pTNQr8qglxGuVLm.png)

在光线步进酱的引诱下，野生的方块出现了！

#### 居中材质

目前的方块有2个问题：1. 没有居中 2. x轴方向上被拉伸

居中+拉伸素质2连走起

```glsl
vec2 centerUv(vec2 uv){
    uv=2.*uv-1.;
    float aspect=uResolution.x/uResolution.y;
    uv.x*=aspect;
    return uv;
}

void main(){
    ...
    vec2 cUv=centerUv(vUv);
    vec3 ray=normalize(vec3(cUv,-eye.z));
    ...
}
```

![4](https://i.loli.net/2021/02/28/STcN24xr1lspnmJ.png)

方块瞬间飘到了画面的正中央，但此时的她还没有颜色

#### 计算表面法线

在光照模型中，我们需要[计算出表面法线](https://www.iquilezles.org/www/articles/normalsSDF/normalsSDF.htm)，才能给材质赋予颜色

```glsl
vec3 calcNormal(in vec3 p)
{
    const float eps=.0001;
    const vec2 h=vec2(eps,0);
    return normalize(vec3(sdf(p+h.xyy)-sdf(p-h.xyy),
    sdf(p+h.yxy)-sdf(p-h.yxy),
    sdf(p+h.yyx)-sdf(p-h.yyx)));
}

void main(){
    ...
    if(depth<end){
      vec3 pos=eye+depth*ray;
      vec3 normal=calcNormal(pos);
      color=normal;
    }
    ...
}
```

![5.png](https://i.loli.net/2021/02/28/oi5VcHN3sOqkm4P.png)

此时方块被赋予了蓝色，但我们还看不出她是个立体图形

#### 动起来

让方块360°旋转起来吧，3D旋转函数直接在[gist](https://gist.github.com/yiwenl/3f804e80d0930e34a0b33359259b556c)上搜一下就有了

```glsl
uniform float uVelocityBox;

mat4 rotationMatrix(vec3 axis,float angle){
    axis=normalize(axis);
    float s=sin(angle);
    float c=cos(angle);
    float oc=1.-c;
    
    return mat4(oc*axis.x*axis.x+c,oc*axis.x*axis.y-axis.z*s,oc*axis.z*axis.x+axis.y*s,0.,
        oc*axis.x*axis.y+axis.z*s,oc*axis.y*axis.y+c,oc*axis.y*axis.z-axis.x*s,0.,
        oc*axis.z*axis.x-axis.y*s,oc*axis.y*axis.z+axis.x*s,oc*axis.z*axis.z+c,0.,
    0.,0.,0.,1.);
}

vec3 rotate(vec3 v,vec3 axis,float angle){
    mat4 m=rotationMatrix(axis,angle);
    return(m*vec4(v,1.)).xyz;
}

float sdf(vec3 p){
    vec3 p1=rotate(p,vec3(1.),uTime*uVelocityBox);
    float box=sdBox(p1,vec3(.3));
    return box;
}
```

![6.gif](https://i.loli.net/2021/02/28/Zw47fDA6gdKkVYq.gif)

#### 融合效果

单单一个方块太孤单了，创建一个球来陪陪她吧

如何让球和方块贴在一起呢，你需要[smin](https://www.iquilezles.org/www/articles/smin/smin.htm)这个函数

```glsl
uniform float uProgress;

float smin(float a,float b,float k)
{
    float h=clamp(.5+.5*(b-a)/k,0.,1.);
    return mix(b,a,h)-k*h*(1.-h);
}

float sdf(vec3 p){
    vec3 p1=rotate(p,vec3(1.),uTime*uVelocityBox);
    float box=sdBox(p1,vec3(.3));
    float sphere=sdSphere(p,.3);
    float sBox=smin(box,sphere,.3);
    float mixedBox=mix(sBox,box,uProgress);
    return mixedBox;
}
```

把`uProgress`的值设为0，她们成功地贴在了一起

![7.png](https://i.loli.net/2021/02/28/9ZUyXY7STNcQtsf.png)

把`uProgress`的值调回1，她们又分开了

#### 动态融合

接下来就是露珠滴落的动画实现了，其实就是对融合图形应用了一个位移变换

```glsl
uniform float uAngle;
uniform float uDistance;
uniform float uVelocitySphere;

const float PI=3.14159265359;

float movingSphere(vec3 p,float shape){
    float rad=uAngle*PI;
    vec3 pos=vec3(cos(rad),sin(rad),0.)*uDistance;
    vec3 displacement=pos*fract(uTime*uVelocitySphere);
    float gotoCenter=sdSphere(p-displacement,.1);
    return smin(shape,gotoCenter,.3);
}

float sdf(vec3 p){
    vec3 p1=rotate(p,vec3(1.),uTime*uVelocityBox);
    float box=sdBox(p1,vec3(.3));
    float sphere=sdSphere(p,.3);
    float sBox=smin(box,sphere,.3);
    float mixedBox=mix(sBox,box,uProgress);
    mixedBox=movingSphere(p,mixedBox);
    return mixedBox;
}
```

![8.gif](https://i.loli.net/2021/02/28/zJXiRHpIhdZ1SQf.gif)

#### matcap贴图

默认的材质太土了？我们有帅气的matcap贴图来助阵

```glsl
uniform sampler2D uTexture;

vec2 matcap(vec3 eye,vec3 normal){
    vec3 reflected=reflect(eye,normal);
    float m=2.8284271247461903*sqrt(reflected.z+1.);
    return reflected.xy/m+.5;
}

float fresnel(float bias,float scale,float power,vec3 I,vec3 N)
{
    return bias+scale*pow(1.+dot(I,N),power);
}

void main(){
    ...
    if(depth<end){
      vec3 pos=eye+depth*ray;
      vec3 normal=calcNormal(pos);
        vec2 matcapUv=matcap(ray,normal);
        color=texture2D(uTexture,matcapUv).rgb;
        float F=fresnel(0.,.4,3.2,ray,normal);
        color=mix(color,bg,F);
    }
    ...
}
```

![ray-marching.gif](https://i.loli.net/2021/02/28/1Zs4rTinGw2PbtU.gif)

安排上了matcap和菲涅尔公式后，瞬间cool了有没有？！

## 项目地址

[Ray Marching Gooey Effect](https://codepen.io/alphardex/pen/BaQYXvy)