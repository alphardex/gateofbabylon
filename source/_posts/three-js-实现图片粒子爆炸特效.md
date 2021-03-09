title: three.js 实现图片粒子爆炸特效
author: alphardex
abbrlink: 794
tags:
  - three.js
  - JavaScript
categories:
  - 前端
date: 2021-03-09 08:23:00
---
## 前言

大家好，这里是 CSS 魔法使——alphardex。

以下是最终实现的效果图

![explode.gif](https://i.loli.net/2021/03/09/behulLMfdQA8oRF.gif)

撒，哈吉马路由！

<!--more-->

## 准备工作

笔者的[three.js模板](https://codepen.io/alphardex/pen/yLaQdOq)：点击右下角的fork即可复制一份

## 世界同步

在我的[上一篇博文](https://juejin.cn/post/6937070693506875429)中，讲到了如何将HTML世界和webgl的世界同步起来，本文也是同样的思路，先同步好两个世界，再进行特效创作

首先搭建HTML和JS

```html
<div class="relative w-screen h-screen">
  <div class="absolute w-screen h-screen flex-center opacity-0">
    <img src="https://i.loli.net/2021/03/08/uYcvELkr4dqFj9w.jpg" class="w-60 cursor-pointer" alt="" crossorigin="anonymous" />
  </div>
  <div class="particle-explode w-full h-full bg-black"></div>
</div>
```

```ts
class ParticleExplode extends Base {
  // 初始化
  async init() {
    this.createScene();
    this.createPerspectiveCamera();
    this.createRenderer();
    this.createParticleExplodeMaterial();
    await preloadImages();
    this.createPoints();
    this.createClickEffect();
    this.createLight();
    this.trackMousePos();
    this.createOrbitControls();
    this.addListeners();
    this.setLoop();
  }
}

const start = () => {
  const particleExplode = new ParticleExplode(".particle-explode", true);
  particleExplode.init();
};

start();
```

### 单位同步

```ts
class ParticleExplode extends Base {
  constructor(sel: string, debug: boolean) {
    ...
    this.cameraPosition = new THREE.Vector3(0, 0, 1500);
    const fov = this.getScreenFov();
    this.perspectiveCameraParams = {
      fov,
      near: 0.1,
      far: 5000
    };
  }
  // 获取跟屏幕同像素的fov角度
  getScreenFov() {
    return ky.rad2deg(
      2 * Math.atan(window.innerHeight / 2 / this.cameraPosition.z)
    );
  }
}
```

### 预加载图片

```ts
import imagesLoaded from "https://cdn.skypack.dev/imagesloaded@4.1.4";

const preloadImages = (sel = "img") => {
  return new Promise((resolve) => {
    imagesLoaded(sel, { background: true }, resolve);
  });
};
```

### 数据同步

创建DOMMeshObject，用来同步HTML和webgl世界的数据

这里有一点要注意的是，由于本文创建的是粒子特效，因此不用`Mesh`，用的是`Points`，它能将`Geometry`以点阵的形式表示

```ts
class DOMMeshObject {
  el!: Element;
  rect!: DOMRect;
  mesh!: THREE.Mesh | THREE.Points;
  constructor(
    el: Element,
    scene: THREE.Scene,
    material: THREE.Material = new THREE.MeshBasicMaterial({ color: 0xff0000 }),
    isPoints = false
  ) {
    this.el = el;
    const rect = el.getBoundingClientRect();
    this.rect = rect;
    const { width, height } = rect;
    const geometry = new THREE.PlaneBufferGeometry(
      width,
      height,
      width,
      height
    );
    const mesh = isPoints
      ? new THREE.Points(geometry, material)
      : new THREE.Mesh(geometry, material);
    scene.add(mesh);
    this.mesh = mesh;
  }
  setPosition() {
    const { mesh, rect } = this;
    const { top, left, width, height } = rect;
    const x = left + width / 2 - window.innerWidth / 2;
    const y = -(top + height / 2 - window.innerHeight / 2) + window.scrollY;
    mesh.position.set(x, y, 0);
  }
}

class ParticleExplode extends Base {
  // 创建材质
  createParticleExplodeMaterial() {
    const particleExplodeMaterial = new THREE.ShaderMaterial({
      vertexShader: particleExplodeVertexShader,
      fragmentShader: particleExplodeFragmentShader,
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
        uProgress: {
          value: 0
        },
        uTexture: {
          value: null
        }
      }
    });
    this.particleExplodeMaterial = particleExplodeMaterial;
  }
  // 创建点
  createPoints() {
    const image = document.querySelector("img")!;
    this.image = image;
    const texture = new THREE.Texture(image);
    texture.needsUpdate = true;
    const material = this.particleExplodeMaterial.clone();
    material.uniforms.uTexture.value = texture;
    const imageDOMMeshObj = new DOMMeshObject(
      image,
      this.scene,
      material,
      true
    );
    imageDOMMeshObj.setPosition();
    this.imageDOMMeshObj = imageDOMMeshObj;
  }
}
```

创建完点阵后，画面上依旧一片黑，为什么呢？因为我们忘了在顶点着色器中设置点的大小了，在`particleExplodeVertexShader`中加入这一行

```glsl
gl_PointSize=2.;
```

![1](https://i.loli.net/2021/03/09/dZ9XwLsipInDq4z.png)

可以看到画面上总算有显示了，其实这不是一张平面，而是由成千上万的点组成的“平面”

现在，你可以将你喜欢的图片贴上去了，片元着色器`particleExplodeFragmentShader`代码如下

```glsl
uniform sampler2D uTexture;

varying vec2 vUv;

void main(){
    vec4 color=texture2D(uTexture,vUv);
    if(color.r<.1&&color.g<.1&&color.b<.1){
        discard;
    }
    gl_FragColor=color;
}
```

![2](https://i.loli.net/2021/03/09/VxjgyoQmi7dat1G.png)


## 爆炸特效

接下来又到了激动人心的时刻——爆炸特效的实现了！

### 噪声

爆炸，简而言之就是大量的微粒在一定空间内进行不规则的大幅运动而形成的奇观。说到“不规则”，我们首先就能想到一个词——“噪声”。

噪声有很多种，最常见的有`perlin noise`、`simplex noise`等，本文用的是基于`simplex noise`的`curl noise`，在google上搜索`curl noise glsl`，很容易就能将下面的[噪声代码](https://github.com/cabbibo/glsl-curl-noise)搞到手（谷歌：关键时刻还是得靠劳资）

```glsl
vec4 permute(vec4 x){return mod(((x*34.)+1.)*x,289.);}
vec4 taylorInvSqrt(vec4 r){return 1.79284291400159-.85373472095314*r;}

float snoise(vec3 v){
    const vec2 C=vec2(1./6.,1./3.);
    const vec4 D=vec4(0.,.5,1.,2.);
    
    // First corner
    vec3 i=floor(v+dot(v,C.yyy));
    vec3 x0=v-i+dot(i,C.xxx);
    
    // Other corners
    vec3 g=step(x0.yzx,x0.xyz);
    vec3 l=1.-g;
    vec3 i1=min(g.xyz,l.zxy);
    vec3 i2=max(g.xyz,l.zxy);
    
    //  x0 = x0 - 0. + 0.0 * C
    vec3 x1=x0-i1+1.*C.xxx;
    vec3 x2=x0-i2+2.*C.xxx;
    vec3 x3=x0-1.+3.*C.xxx;
    
    // Permutations
    i=mod(i,289.);
    vec4 p=permute(permute(permute(
                i.z+vec4(0.,i1.z,i2.z,1.))
                +i.y+vec4(0.,i1.y,i2.y,1.))
                +i.x+vec4(0.,i1.x,i2.x,1.));
                
                // Gradients
                // ( N*N points uniformly over a square, mapped onto an octahedron.)
                float n_=1./7.;// N=7
                vec3 ns=n_*D.wyz-D.xzx;
                
                vec4 j=p-49.*floor(p*ns.z*ns.z);//  mod(p,N*N)
                
                vec4 x_=floor(j*ns.z);
                vec4 y_=floor(j-7.*x_);// mod(j,N)
                
                vec4 x=x_*ns.x+ns.yyyy;
                vec4 y=y_*ns.x+ns.yyyy;
                vec4 h=1.-abs(x)-abs(y);
                
                vec4 b0=vec4(x.xy,y.xy);
                vec4 b1=vec4(x.zw,y.zw);
                
                vec4 s0=floor(b0)*2.+1.;
                vec4 s1=floor(b1)*2.+1.;
                vec4 sh=-step(h,vec4(0.));
                
                vec4 a0=b0.xzyw+s0.xzyw*sh.xxyy;
                vec4 a1=b1.xzyw+s1.xzyw*sh.zzww;
                
                vec3 p0=vec3(a0.xy,h.x);
                vec3 p1=vec3(a0.zw,h.y);
                vec3 p2=vec3(a1.xy,h.z);
                vec3 p3=vec3(a1.zw,h.w);
                
                //Normalise gradients
                vec4 norm=taylorInvSqrt(vec4(dot(p0,p0),dot(p1,p1),dot(p2,p2),dot(p3,p3)));
                p0*=norm.x;
                p1*=norm.y;
                p2*=norm.z;
                p3*=norm.w;
                
                // Mix final noise value
                vec4 m=max(.6-vec4(dot(x0,x0),dot(x1,x1),dot(x2,x2),dot(x3,x3)),0.);
                m=m*m;
                return 42.*dot(m*m,vec4(dot(p0,x0),dot(p1,x1),
                dot(p2,x2),dot(p3,x3)));
            }
            
            vec3 snoiseVec3(vec3 x){
                return vec3(snoise(vec3(x)*2.-1.),
                snoise(vec3(x.y-19.1,x.z+33.4,x.x+47.2))*2.-1.,
                snoise(vec3(x.z+74.2,x.x-124.5,x.y+99.4)*2.-1.)
            );
        }
        
        vec3 curlNoise(vec3 p){
            const float e=.1;
            vec3 dx=vec3(e,0.,0.);
            vec3 dy=vec3(0.,e,0.);
            vec3 dz=vec3(0.,0.,e);
            
            vec3 p_x0=snoiseVec3(p-dx);
            vec3 p_x1=snoiseVec3(p+dx);
            vec3 p_y0=snoiseVec3(p-dy);
            vec3 p_y1=snoiseVec3(p+dy);
            vec3 p_z0=snoiseVec3(p-dz);
            vec3 p_z1=snoiseVec3(p+dz);
            
            float x=p_y1.z-p_y0.z-p_z1.y+p_z0.y;
            float y=p_z1.x-p_z0.x-p_x1.z+p_x0.z;
            float z=p_x1.y-p_x0.y-p_y1.x+p_y0.x;
            
            const float divisor=1./(2.*e);
            return normalize(vec3(x,y,z)*divisor);
        }
```

### 应用噪声

将噪声函数弄来后，立马就能应用到我们的片元着色器里，思路也很简单粗暴：将位置信息传递给噪声，再给原先的位置加上噪声就OK了，由于噪声的值很大，需要慢慢地对值进行调试，这里花的时间相对较多一些

```glsl
uniform float uTime;
uniform float uProgress;
varying vec2 vUv;

void main(){
    vec3 noise=curlNoise(vec3(position.x*.02,position.y*.008,uTime*.05));
    vec3 distortion=vec3(position.x*2.,position.y,1.)*noise*uProgress;
    vec3 newPos=position+distortion;
    vec4 modelPosition=modelMatrix*vec4(newPos,1.);
    vec4 viewPosition=viewMatrix*modelPosition;
    vec4 projectedPosition=projectionMatrix*viewPosition;
    gl_Position=projectedPosition;
    gl_PointSize=2.;
    
    vUv=uv;
}
```

### 动起来

将噪声的值调成你预想中的效果后，监听好点击事件，用`gsap`来改变爆炸的过程值，这样爆炸效果就实现了

```ts
import gsap from "https://cdn.skypack.dev/gsap@3.6.0";

class ParticleExplode extends Base {
  // 创建点击效果
  createClickEffect() {
    const material = this.imageDOMMeshObj.mesh.material as any;
    const image = this.image;
    image.addEventListener("click", () => {
      if (!this.isOpen) {
        gsap.to(material.uniforms.uProgress, {
          value: 3,
          duration: 1
        });
        this.isOpen = true;
      } else {
        gsap.to(material.uniforms.uProgress, {
          value: 0,
          duration: 1
        });
        this.isOpen = false;
      }
    });
  }
  // 动画
  update() {
    const elapsedTime = this.clock.getElapsedTime();
    if (this.imageDOMMeshObj) {
      const material = this.imageDOMMeshObj.mesh.material as any;
      material.uniforms.uTime.value = elapsedTime;
    }
  }
}
```

![explode.gif](https://i.loli.net/2021/03/09/behulLMfdQA8oRF.gif)

## 项目地址

[Particle Explode](https://codepen.io/alphardex/full/vYyVxXO)