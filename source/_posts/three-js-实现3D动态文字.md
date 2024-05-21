title: three.js 实现 3D 动态文字
author: alphardex
abbrlink: 23487
tags: []
categories: []
date: 2021-02-21 16:38:00

---

## 前言

大家好，这里是 CSS 魔法使——alphardex。

之前在逛国外网站的时候，发现有些网站的文字是刻在 3D 图形上的，并且能在图形上运动，视觉效果相当不错，于是笔者就也想用 three.js 来尝试复现出这种效果

![kinetic-text.gif](https://i.loli.net/2021/02/21/UnEqc5uBYvofiKt.gif)

上图只是所有效果的其中之一，接下来让我们一起开干吧~

<!--more-->

## 准备工作

笔者自行封装的 three.js 模板：[Three.js Starter](https://codepen.io/alphardex/pen/yLaQdOq)

读者可以点击右下角 fork 一份后再开始本项目

本项目需要用到位图字体，可以直接复制[demo](https://codepen.io/alphardex/pen/wvoqjer)的 HTML 里的 font 字体代码

一个注意点：`three-bmfont-text`这个库依赖全局的 three.js，因此要在 JS 里额外引入一次 three.js，如下图

![Snipaste_2021-02-21_16-53-54.png](https://i.loli.net/2021/02/21/XK95mTdu8BozYch.png)

## 实现思路

1. 加载位图字体文件，将其转化为文字对象所需要的形状和材质
2. 创建文字对象
3. 创建渲染目标，可以理解为 canvas 中的 canvas，因为接下来我们要将文字对象本身当做贴图
4. 创建承载字体的容器，将文字对象作为贴图贴上去
5. 动画

## 正片

### 搭好架子

```html
<div class="relative w-screen h-screen">
  <div class="kinetic-text w-full h-full bg-blue-1"></div>
  <div class="font">
    <font> 一坨从demo里CV而来的字体代码 </font>
  </div>
</div>
```

```css
:root {
  --blue-color-1: #2c3e50;
}

.bg-blue-1 {
  background: var(--blue-color-1);
}
```

```ts
import createGeometry from "https://cdn.skypack.dev/three-bmfont-text@3.0.1";
import MSDFShader from "https://cdn.skypack.dev/three-bmfont-text@3.0.1/shaders/msdf";
import parseBmfontXml from "https://cdn.skypack.dev/parse-bmfont-xml@1.1.4";

const font = parseBmfontXml(document.querySelector(".font").innerHTML);
const fontAtlas = "https://i.loli.net/2021/02/20/DcEhuYNjxCgeU42.png";

const kineticTextTorusKnotVertexShader = `（顶点着色器代码，先空着，具体见下文）`;

const kineticTextTorusKnotFragmentShader = `（片元着色器代码，先空着，具体见下文）`;

class KineticText extends Base {
  constructor(sel: string, debug: boolean) {
    super(sel, debug);
    this.cameraPosition = new THREE.Vector3(0, 0, 4);
    this.clock = new THREE.Clock();
    this.meshConfig = {
      torusKnot: {
        vertexShader: kineticTextTorusKnotVertexShader,
        fragmentShader: kineticTextTorusKnotFragmentShader,
        geometry: new THREE.TorusKnotGeometry(9, 3, 768, 3, 4, 3),
      },
    };
    this.meshNames = Object.keys(this.meshConfig);
    this.params = {
      meshName: "torusKnot",
      velocity: 0.5,
      shadow: 5,
      color: "#000000",
      frequency: 0.5,
      text: "ALPHARDEX",
      cameraZ: 2.5,
    };
  }
  // 初始化
  async init() {
    this.createScene();
    this.createPerspectiveCamera();
    this.createRenderer(true);
    await this.createKineticText(this.params.text);
    this.createLight();
    this.createOrbitControls();
    this.addListeners();
    this.setLoop();
  }
  // 创建动态文字
  async createKineticText(text: string) {
    await this.createFontText(text);
    this.createRenderTarget();
    this.createTextContainer();
  }
}
```

### 加载和创建字体

首先加载字体文件，并创建出形状和材质，有了这两样就能创建出字体对象了

```ts
class KineticText extends Base {
  loadFontText(text: string): any {
    return new Promise((resolve) => {
      const fontGeo = createGeometry({
        font,
        text,
      });
      const loader = new THREE.TextureLoader();
      loader.load(fontAtlas, (texture) => {
        const fontMat = new THREE.RawShaderMaterial(
          MSDFShader({
            map: texture,
            side: THREE.DoubleSide,
            transparent: true,
            negate: false,
            color: 0xffffff,
          })
        );
        resolve({ fontGeo, fontMat });
      });
    });
  }
  async createFontText(text: string) {
    const { fontGeo, fontMat } = await this.loadFontText(text);
    const textMesh = this.createMesh({
      geometry: fontGeo,
      material: fontMat,
    });
    textMesh.position.set(-0.965, -0.525, 0);
    textMesh.rotation.set(ky.deg2rad(180), 0, 0);
    textMesh.scale.set(0.008, 0.025, 1);
    this.textMesh = textMesh;
  }
}
```

### 着色器

#### 顶点着色器

通用模板，直接 CV 即可

```glsl
varying vec2 vUv;
varying vec3 vPosition;

void main(){
    vec4 modelPosition=modelMatrix*vec4(position,1.);
    vec4 viewPosition=viewMatrix*modelPosition;
    vec4 projectedPosition=projectionMatrix*viewPosition;
    gl_Position=projectedPosition;

    vUv=uv;
    vPosition=position;
}
```

#### 片元着色器

利用`fract`函数创建重复的贴图，加上位移距离`displacement`使得贴图能随着时间的增加而动起来，再用`clamp`函数来根据 z 轴大小限定阴影的范围，意思是离画面越远则阴影越重，反之离画面越近则阴影越轻

```glsl
uniform sampler2D uTexture;
uniform float uTime;
uniform float uVelocity;
uniform float uShadow;

varying vec2 vUv;
varying vec3 vPosition;

void main(){
    vec2 repeat=vec2(12.,3.);
    vec2 repeatedUv=vUv*repeat;
    vec2 displacement=vec2(uTime*uVelocity,0.);
    vec2 uv=fract(repeatedUv+displacement);
    vec3 texture=texture2D(uTexture,uv).rgb;
    // texture*=vec3(uv.x,uv.y,1.);
    float shadow=clamp(vPosition.z/uShadow,0.,1.);// farther darker (to 0).
    vec3 color=vec3(texture*shadow);
    gl_FragColor=vec4(color,1.);
}
```

此时文本显示到了屏幕上

![Snipaste_2021-02-21_17-01-56.png](https://i.loli.net/2021/02/21/r6bfS5ytGkjHqUl.png)

### 创建渲染目标

为了将字体对象本身作为贴图，创建了一个渲染目标

```ts
class KineticText extends Base {
  createRenderTarget() {
    const rt = new THREE.WebGLRenderTarget(
      window.innerWidth,
      window.innerHeight
    );
    this.rt = rt;
    const rtCamera = new THREE.PerspectiveCamera(45, 1, 0.1, 1000);
    rtCamera.position.z = this.params.cameraZ;
    this.rtCamera = rtCamera;
    const rtScene = new THREE.Scene();
    rtScene.add(this.textMesh);
    this.rtScene = rtScene;
  }
}
```

### 创建字体容器

创建一个容器，并将字体对象本身作为贴图贴上去，再应用动画即可完成

```ts
class KineticText extends Base {
  createTextContainer() {
    if (this.mesh) {
      this.scene.remove(this.mesh);
      this.mesh = null;
      this.material!.dispose();
      this.material = null;
    }
    this.rtScene.background = new THREE.Color(this.params.color);
    const meshConfig = this.meshConfig[this.params.meshName];
    const geometry = meshConfig.geometry;
    const material = new THREE.ShaderMaterial({
      vertexShader: meshConfig.vertexShader,
      fragmentShader: meshConfig.fragmentShader,
      uniforms: {
        uTime: {
          value: 0,
        },
        uVelocity: {
          value: this.params.velocity,
        },
        uTexture: {
          value: this.rt.texture,
        },
        uShadow: {
          value: this.params.shadow,
        },
        uFrequency: {
          value: this.params.frequency,
        },
      },
    });
    this.material = material;
    const mesh = this.createMesh({
      geometry,
      material,
    });
    this.mesh = mesh;
  }
  update() {
    if (this.rtScene) {
      this.renderer.setRenderTarget(this.rt);
      this.renderer.render(this.rtScene, this.rtCamera);
      this.renderer.setRenderTarget(null);
    }
    const elapsedTime = this.clock.getElapsedTime();
    if (this.material) {
      this.material.uniforms.uTime.value = elapsedTime;
    }
  }
}
```

别忘了把相机调远一些

```ts
this.cameraPosition = new THREE.Vector3(0, 0, 40);
```

风骚的动态文字出现了:)

![kinetic-text.gif](https://i.loli.net/2021/02/21/UnEqc5uBYvofiKt.gif)

## 项目地址

[Kinetic Text](https://codepen.io/alphardex/pen/wvoqjer)

demo 里不止本文创建的这一种形状，大家可以随意把玩。
