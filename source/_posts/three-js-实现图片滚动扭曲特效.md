title: three.js 实现图片滚动扭曲特效
author: alphardex
abbrlink: 7601
tags: []
categories: []
date: 2021-03-06 21:14:00

---

## 前言

大家好，这里是 CSS 魔法使——alphardex。

平时我们见过很多的图片悬浮和滚动特效，大部分是用 CSS 和 SVG 实现的，但是有一种特效它们绝对实现不了——扭曲特效。为何？CSS 擅长直线型变换，而 SVG 擅长曲线型变换。扭曲特效则两者都不是，它是像素级变换，能做到像素级变换的只有 canvas，而 webgl 的片元着色器其实特别擅长这一点，我们可以利用它来实现各种酷炫的扭曲特效。以下是最终实现的效果图

![scroll.gif](https://i.loli.net/2021/03/06/EgYx4XAb29tOIRG.gif)

撒，哈吉马路由！

<!--more-->

## 准备工作

笔者的[three.js 模板](https://codepen.io/alphardex/pen/yLaQdOq)：点击右下角的 fork 即可复制一份

## 实现思路

实现本文效果最重要的一点是：将 HTML 世界与 webgl 世界同步起来！

一旦两个世界同步了，各种炫酷的效果都可以应用在原生 HTML 元素上，且不影响基本的交互！

## 世界同步

### 搭建 HTML

第一步：将你想展示的所有图片搭成一个简单的 HTML 页面

```html
<main class="overflow-hidden">
  <div data-scroll>
    <div class="relative w-screen h-screen flex-center">
      <img
        class="w-240 h-120"
        src="https://i.loli.net/2019/11/16/cqyJiYlRwnTeHmj.jpg"
        alt=""
        crossorigin="anonymous"
      />
    </div>
    <div class="relative w-screen h-screen flex-center">
      <img
        class="w-240 h-120"
        src="https://i.loli.net/2019/10/18/Ujf6n75o8TtIsWX.jpg"
        alt=""
        crossorigin="anonymous"
      />
    </div>
    <div class="relative w-screen h-screen flex-center">
      <img
        class="w-240 h-120"
        src="https://i.loli.net/2019/10/18/buDT4YS6zUMfHst.jpg"
        alt=""
        crossorigin="anonymous"
      />
    </div>
    <div class="relative w-screen h-screen flex-center">
      <img
        class="w-240 h-120"
        src="https://i.loli.net/2019/10/18/uXF1Kx7lzELB6wf.jpg"
        alt=""
        crossorigin="anonymous"
      />
    </div>
    <div class="relative w-screen h-screen flex-center">
      <img
        class="w-240 h-120"
        src="https://i.loli.net/2019/11/03/RtVq2wxQYySDb8L.jpg"
        alt=""
        crossorigin="anonymous"
      />
    </div>
    <div class="relative w-screen h-screen flex-center">
      <img
        class="w-240 h-120"
        src="https://i.loli.net/2019/11/16/FLnzi5Kq4tkRZSm.jpg"
        alt=""
        crossorigin="anonymous"
      />
    </div>
  </div>
</main>
<div class="twisted-gallery fixed -z-1 inset-0 w-screen h-screen"></div>
```

```css
.w-240 {
  width: 60rem;
}

.h-120 {
  height: 30rem;
}
```

看到图片整齐的排列后，可以把它们隐藏起来了

```css
img {
  opacity: 0;
}
```

在我们的主类中，先创建一个总函数，里面未实现的函数可以先注释掉，下文会慢慢地实现它们

```ts
class TwistedGallery extends Base {
  ...
  async init() {
    this.createScene();
    this.createPerspectiveCamera();
    this.createRenderer();
    this.createPlane();
    // await preloadImages();
    // this.createDistortImageMaterial();
    // this.createImageDOMMeshObjs();
    // this.setImagesPosition();
    // this.listenScroll();
    // this.createPostprocessingEffect()
    this.createLight();
    this.createOrbitControls();
    this.addListeners();
    this.setLoop();
  }
  ...
}

const start = () => {
  const twistedGallery = new TwistedGallery(".twisted-gallery", true);
  twistedGallery.init();
};
```

### 将 HTML 和 webgl 的单位同步

为了在 webgl 中渲染出 HTML 里的所有 img，我们需要把 HTML 里的 img 的像素信息同步给 webgl

问题来了，如何使得 webgl 里的 1 单位长度===HTML 里元素的 1px？

这时我们就要搬上那张经典的透视示意图了，我们需要根据公式来计算出视场角 fov

![fov](https://i.loli.net/2021/03/06/btZ8eK2jYV3Cp9n.png)

```ts
class TwistedGallery extends Base {
  constructor(sel: string, debug: boolean) {
    ...
    this.cameraPosition = new THREE.Vector3(0, 0, 600);
    const fov = this.getScreenFov();
    this.perspectiveCameraParams = {
      fov,
      near: 100,
      far: 2000,
    };
  }
  getScreenFov() {
    return ky.rad2deg(
      2 * Math.atan(window.innerHeight / 2 / this.cameraPosition.z)
    );
  }
}
```

为了验证，我们将 createPlane 函数里的 PlaneBufferGeometry 的宽高都调为 100

![1](https://i.loli.net/2021/03/06/V7Dah5HZGkCLucl.png)

画面中的平面宽高皆为 100px，和 HTML 完全一致！

`createPlane`函数完成了它的测试使命，可以删掉了

### 确保图片加载完毕

在显示图片之前，我们要确保图片全部加载完毕，故用[imagesLoaded](https://github.com/desandro/imagesloaded)库来进行判断

```ts
import imagesLoaded from "https://cdn.skypack.dev/imagesloaded@4.1.4";

const preloadImages = (sel = "img") => {
  return new Promise((resolve) => {
    imagesLoaded(sel, { background: true }, resolve);
  });
};
```

### 将图片显示在 webgl 上

我们创建一个 DOMMeshObject 类，该类是一座桥梁，负责将 DOM 里的信息同步至 webgl 世界

首先获取 DOM 元素的长宽和位置，再根据这些数据计算出它们在 webgl 中对应的位置和大小

```ts
class DOMMeshObject {
  constructor(
    el: Element,
    scene: THREE.Scene,
    material: THREE.Material = new THREE.MeshBasicMaterial({ color: 0xff0000 })
  ) {
    this.el = el;
    const rect = el.getBoundingClientRect();
    this.rect = rect;
    const { width, height } = rect;
    const geometry = new THREE.PlaneBufferGeometry(width, height, 10, 10);
    const mesh = new THREE.Mesh(geometry, material);
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
```

在 webgl 世界中创建出图片材质，并将 HTML 里的 img 元素作为贴图传给着色器，再同步好位置即可

```ts
class TwistedGallery extends Base {
  constructor(sel: string, debug: boolean) {
    ...
    this.images = [...document.querySelectorAll("img")];
    this.imageDOMMeshObjs = [];
  }
  // 创建材质
  createDistortImageMaterial() {
    const distortImageMaterial = new THREE.ShaderMaterial({
      vertexShader: twistedGalleryMainVertexShader,
      fragmentShader: twistedGalleryMainFragmentShader,
      side: THREE.DoubleSide,
      uniforms: {
        uTexture: {
          value: 0
        }
      }
    });
    this.distortImageMaterial = distortImageMaterial;
  }
  // 创建图片DOM物体
  createImageDOMMeshObjs() {
    const { images, scene, distortImageMaterial } = this;
    const imageDOMMeshObjs = images.map((image) => {
      const texture = new THREE.Texture(image);
      texture.needsUpdate = true;
      const material = distortImageMaterial.clone();
      material.uniforms.uTexture.value = texture;
      const imageDOMMeshObj = new DOMMeshObject(image, scene, material);
      return imageDOMMeshObj;
    });
    this.imageDOMMeshObjs = imageDOMMeshObjs;
  }
  // 设置图片位置
  setImagesPosition() {
    const { imageDOMMeshObjs } = this;
    imageDOMMeshObjs.forEach((obj) => {
      obj.setPosition();
    });
  }
}
```

顶点着色器`twistedGalleryMainVertexShader`跟模板一致

```glsl
varying vec2 vUv;

void main(){
    vec4 modelPosition=modelMatrix*vec4(position,1.);
    vec4 viewPosition=viewMatrix*modelPosition;
    vec4 projectedPosition=projectionMatrix*viewPosition;
    gl_Position=projectedPosition;

    vUv=uv;
}
```

片元着色器`twistedGalleryMainFragmentShader`将贴图作为颜色显示了出来

```glsl
uniform sampler2D uTexture;

varying vec2 vUv;

void main(){
    vec2 newUv=vUv;
    vec4 texture=texture2D(uTexture,newUv);
    vec3 color=texture.rgb;
    gl_FragColor=vec4(color,1.);
}
```

![2](https://i.loli.net/2021/03/06/r3DiHp6PmAayNoI.png)

经过一番努力，图片总算显示在画面上了，接下来我们就要开始让它跟随画面滚动起来了

### 滚动起来

这里的滚动监听原生的`scroll`事件即可实现，但本文并不会这么做，为什么呢？因为原生的`scroll`事件只能获取滚动的位置，而无法获取滚动的速度。如果用户滚得快，我们也要在我们的特效上体现出来。因此，我们将采用[locomotive-scroll](https://github.com/locomotivemtl/locomotive-scroll)这个库，它能捕获到用户滚动的位置和速度

```ts
import LocomotiveScroll from "https://cdn.skypack.dev/locomotive-scroll@4.1.0";

class TwistedGallery extends Base {
  // 监听滚动
  listenScroll() {
    const scroll = new LocomotiveScroll({
      getSpeed: true,
    });
    scroll.on("scroll", () => {
      this.setImagesPosition();
    });
    this.scroll = scroll;
  }
}
```

![3](https://i.loli.net/2021/03/06/jnmSkbsa3hRX7I2.gif)

图片终于在 WEBGL 的世界里滚动起来了，两个世界同步完成~

接下来开始我们的重头戏——扭曲特效！

## 扭曲特效

开头的那个特效可以看出是全屏幕上对图片进行扭曲的，因此我们将采用后期处理`postprocessing`来实现，它提供了屏幕`tDiffuse`

```ts
import { RenderPass } from "https://cdn.skypack.dev/three@0.124.0/examples/jsm/postprocessing/RenderPass.js";
import { ShaderPass } from "https://cdn.skypack.dev/three@0.124.0/examples/jsm/postprocessing/ShaderPass.js";
import gsap from "https://cdn.skypack.dev/gsap@3.6.0";

class TwistedGallery extends Base {
  constructor(sel: string, debug: boolean) {
    ...
    this.scrollSpeed = 0;
  }
  // 创建后期处理特效
  createPostprocessingEffect() {
    const composer = new EffectComposer(this.renderer);
    const renderPass = new RenderPass(this.scene, this.camera);
    composer.addPass(renderPass);
    const customPass = new ShaderPass({
      vertexShader: twistedGalleryPostprocessingVertexShader,
      fragmentShader: twistedGalleryPostprocessingFragmentShader,
      uniforms: {
        tDiffuse: {
          value: null
        },
        uRadius: {
          value: 0.75
        },
        uPower: {
          value: 0
        }
      }
    });
    customPass.renderToScreen = true;
    composer.addPass(customPass);
    this.composer = composer;
    this.customPass = customPass;
  }
  // 设置滚动速度
  setScrollSpeed() {
    const scrollSpeed = this.scroll.scroll.instance.speed || 0;
    gsap.to(this, {
      scrollSpeed: Math.min(Math.abs(scrollSpeed) * 1.25, 2),
      duration: 1
    });
  }
  // 动画
  update() {
    if (this.customPass) {
      this.setScrollSpeed();
      this.customPass.uniforms.uPower.value = this.scrollSpeed;
    }
  }
}
```

可以看出我们将滚动速度作为`uniform`传入了着色器，并且用[gsap](https://github.com/greensock/GSAP)来实现缓动效果，接下来就是着色器部分了

顶点着色器`twistedGalleryPostprocessingVertexShader`跟上文所说的模板一致，故略过

片元着色器`twistedGalleryPostprocessingFragmentShader`负责动态计算 uv 坐标，实现起来稍有难度，关键是多调，里面的值用`gl_FragColor`显示出来，这样就容易理解值的变化

```glsl
uniform sampler2D tDiffuse;
uniform float uRadius;
uniform float uPower;

varying vec2 vUv;

void main(){
    vec2 pivot=vec2(.5);
    vec2 d=vUv-pivot;
    float rDist=length(d);
    float gr=pow(rDist/uRadius,uPower);
    float mag=2.-cos(gr-1.);
    vec2 uvR=pivot+d*mag;
    vec4 color=texture2D(tDiffuse,uvR);
    gl_FragColor=color;
}
```

![scroll.gif](https://i.loli.net/2021/03/06/EgYx4XAb29tOIRG.gif)

## 项目地址

[Twisted Gallery](https://codepen.io/alphardex/full/eYBLWOY)

## 最后

本文所实现的特效仅仅是众多特效中的一种，但真正重要的是如何将 HTML 与 webgl 世界所同步起来的这个过程，一旦掌握了这一方法，想做出一个非常酷炫的网站将不再是一个难事。
