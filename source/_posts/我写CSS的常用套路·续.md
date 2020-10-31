title: 我写 CSS 的常用套路·续
author: alphardex
abbrlink: 32261
tags:
  - CSS
categories:
  - 前端
date: 2020-10-09 15:00:00
---
# 前言

前篇传送门：[猛戳这里](https://juejin.im/post/6844904033405108232)

其实大多数的技巧前篇都已经讲完了，本文算是具有补充性质的番外篇吧0.0

<!--more-->

# 3D 方块

如何在 CSS 中创建立体的方块呢？在SCSS中用以下的 mixin 即可

方块的长度、高度、深度都可以通过 CSS 变量自由调节

```scss
@mixin cube($width, $height, $depth) {
  &__front {
    @include cube-front($width, $height, $depth);
  }
  &__back {
    @include cube-back($width, $height, $depth);
  }
  &__right {
    @include cube-right($width, $height, $depth);
  }
  &__left {
    @include cube-left($width, $height, $depth);
  }
  &__top {
    @include cube-top($width, $height, $depth);
  }
  &__bottom {
    @include cube-bottom($width, $height, $depth);
  }
  .face {
    position: absolute;
  }
}

@mixin cube-front($width, $height, $depth) {
  width: var($width);
  height: var($height);
  transform-origin: bottom left;
  transform: rotateX(-90deg) translateZ(calc(calc(var(#{$depth}) * 2) - var(#{$height})));
}

@mixin cube-back($width, $height, $depth) {
  width: var($width);
  height: var($height);
  transform-origin: top left;
  transform: rotateX(-90deg) rotateY(180deg) translateX(calc(var(#{$width}) * -1)) translateY(
      calc(var(#{$height}) * -1)
    );
}

@mixin cube-right($width, $height, $depth) {
  width: calc(var(#{$depth}) * 2);
  height: var($height);
  transform-origin: top left;
  transform: rotateY(90deg) rotateZ(-90deg) translateZ(var(#{$width})) translateX(calc(var(#{$depth}) * -2)) translateY(calc(var(
            #{$height}
          ) * -1));
}

@mixin cube-left($width, $height, $depth) {
  width: calc(var(#{$depth}) * 2);
  height: var($height);
  transform-origin: top left;
  transform: rotateY(-90deg) rotateZ(90deg) translateY(calc(var(#{$height}) * -1));
}

@mixin cube-top($width, $height, $depth) {
  width: var($width);
  height: calc(var(#{$depth}) * 2);
  transform-origin: top left;
  transform: translateZ(var($height));
}

@mixin cube-bottom($width, $height, $depth) {
  width: var($width);
  height: calc(var(#{$depth}) * 2);
  transform-origin: top left;
  transform: rotateY(180deg) translateX(calc(var(#{$width}) * -1));
}

.cube {
  --cube-width: 3rem;
  --cube-height: 3rem;
  --cube-depth: 1.5rem;

  @include cube(--cube-width, --cube-height, --cube-depth);
  width: 3rem;
  height: 3rem;
}
```

## 交错旋转

给多个方块应用交错动画会产生如下效果

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9b079bad28b4971a438db1960436d43~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)

```scss
.spiral-tower {
  display: grid;
  grid-auto-flow: row;
  transform: rotateX(-30deg) rotateY(45deg);

  .cube {
    @for $i from 1 through 48 {
      &:nth-child(#{$i}) {
        animation-delay: 0.015s * ($i - 1);
      }
    }
  }
}

@keyframes spin {
  0%,
  15% {
    transform: rotateY(0);
  }

  85%,
  100% {
    transform: rotateY(1turn);
  }
}
```

本 demo 地址：[Spiral Tower](https://codepen.io/alphardex/pen/mdPjLGm)

## 伸缩长度

在 CSS 动画中，我们无法直接使变量动起来（其实能动，但很生硬）

这时我们就得求助于 CSS Houdini，将变量声明为长度单位类型即可，因为长度单位是可以动起来的

```js
CSS.registerProperty({
  name: "--cube-width",
  syntax: "<length>",
  initialValue: 0,
  inherits: true,
});

CSS.registerProperty({
  name: "--cube-height",
  syntax: "<length>",
  initialValue: 0,
  inherits: true,
});

CSS.registerProperty({
  name: "--cube-depth",
  syntax: "<length>",
  initialValue: 0,
  inherits: true,
});
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/734029e9456449db98f013dde3d8dcfe~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)
本 demo 地址：[3D Stair Loading](https://codepen.io/alphardex/pen/YzqrGXb)

# 文本分割

在上一篇我们提到了如何用 JS 来分割文本，本篇将介绍一种更简洁的实现方法——gsap 的 SplitText 插件，利用它我们能用更少的代码来实现下图的效果

```html
<div class="staggered-land-in font-bold text-2xl">Fushigi no Monogatari</div>
```

```js
const t1 = gsap.timeline();
const staggeredLandInText = new SplitText(".staggered-land-in", {
  type: "chars",
});
t1.from(staggeredLandInText.chars, {
  duration: 0.8,
  opacity: 0,
  y: "-20%",
  stagger: 0.05,
});
```

<img src="https://user-gold-cdn.xitu.io/2020/2/8/1702507ba8f9dbf3?imageslim" referrerpolicy="no-referrer"/>

简化版 demo 地址：[SplitText Starter](https://codepen.io/alphardex/pen/ZEWRBJp)

## 关键帧

简单的动画固然可以实现，那么相对复杂一点的动画呢？这时候还是要依靠强大的@keyframes 和 CSS 变量

注：尽管 gsap 目前也支持 keyframes，但是无法和交错动画结合起来，因此用@keyframes 作为替代方案

```html
<div class="staggered-scale-in font-bold text-6xl">Never Never Give Up</div>
```

```css
.scale-in-bounce {
  animation: scale-in-bounce 0.4s both;
  animation-delay: calc(0.1s * var(--i));
}

@keyframes scale-in-bounce {
  0% {
    opacity: 0;
    transform: scale(2);
  }

  40% {
    opacity: 1;
    transform: scale(0.8);
  }

  100% {
    opacity: 1;
    transform: scale(1);
  }
}
```

```js
const t1 = gsap.timeline();
const staggeredScaleInText = new SplitText(".staggered-scale-in", {
  type: "chars",
});
const staggeredScaleInChars = staggeredScaleInText.chars;
staggeredScaleInChars.forEach((item, i) => {
  item.style.setProperty("--i", `${i}`);
});
t1.to(staggeredScaleInChars, {
  className: "scale-in-bounce",
});
```

![](https://i.loli.net/2020/09/17/fFhUwdnP1GuObrK.gif)

本 demo 地址：[Staggered Scale In Text](https://codepen.io/alphardex/pen/eYZLJqw)

# SVG 滤镜

CSS 的滤镜其实都是 SVG 滤镜的封装版本，方便我们使用而已

SVG 滤镜则更加灵活强大，以下是几个常见的滤镜使用场景

附在线调试 SVG 滤镜的网站：[SVG Filters](https://yoksel.github.io/svg-filters/#/)

## 粘滞效果

```html
<svg width="0" height="0" class="absolute">
  <filter id="goo">
    <feGaussianBlur stdDeviation="10 10" in="SourceGraphic" result="blur" />
    <feColorMatrix
      type="matrix"
      values="1 0 0 0 0
    0 1 0 0 0
    0 0 1 0 0
    0 0 0 18 -7"
      in="blur"
      result="colormatrix"
    />
    <feComposite in="SourceGraphic" in2="colormatrix" operator="over" result="composite" />
  </filter>
</svg>
```

```css
.gooey {
  filter: url("#goo");
}
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc85f2aef03c49d2bcf28f0e04b21625~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)

本 demo 地址：[SVG Filter Gooey Menu](https://codepen.io/alphardex/pen/GRZOvwJ)

## 故障效果

```html
<svg width="0" height="0" class="absolute">
  <filter id="glitch">
    <feTurbulence type="fractalNoise" baseFrequency="0.00001 0.000001" numOctaves="1" result="turbulence1">
      <animate
        attributeName="baseFrequency"
        from="0.00001 0.000001"
        to="0.00001 0.4"
        dur="0.4s"
        id="glitch1"
        fill="freeze"
        repeatCount="indefinite"
      ></animate>
      <animate
        attributeName="baseFrequency"
        from="0.00001 0.4"
        to="0.00001 0.2"
        dur="0.2s"
        begin="glitch1.end"
        fill="freeze"
        repeatCount="indefinite"
      ></animate>
    </feTurbulence>
    <feDisplacementMap
      in="SourceGraphic"
      in2="turbulence1"
      scale="30"
      xChannelSelector="R"
      yChannelSelector="G"
      result="displacementMap"
    />
  </filter>
</svg>
```

```css
.glitch {
  filter: url("#glitch");
}
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45881c14da48420b87f70a0b03296eb1~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)

本 demo 地址：[SVG Filter Glitch Button](https://codepen.io/alphardex/pen/mdPqmaM)

## 动态模糊

CSS 滤镜的 blur 是全方位模糊，而 SVG 滤镜的 blur 可以控制单方向的模糊

```html
<svg width="0" height="0" class="absolute">
  <filter id="motion-blur" filterUnits="userSpaceOnUse">
    <feGaussianBlur stdDeviation="100 0" in="SourceGraphic" result="blur">
      <animate dur="0.6s" attributeName="stdDeviation" from="100 0" to="0 0" fill="freeze"></animate>
    </feGaussianBlur>
  </filter>
</svg>
```

```css
.motion-blur {
  filter: url("#motion-blur");
}
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc2320c3f44b4d29891eedfc5c09a94b~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)

本 demo 地址：[SVG Filter Motion Blur](https://codepen.io/alphardex/pen/KKzeLVO?editors=0110)

# mask 遮罩

有时候我们想做出一种过渡式的半透明效果，类似下图这样的

![](https://i.loli.net/2020/09/14/VXrtQnbhZDAwiLc.png)

这时候就得借助 mask 属性了，因为图片与 mask 生成的渐变的 transparent 的重叠部分会变透明

```css
.divider-grad-mask {
  background: linear-gradient(90deg, var(--blue-color) 0 50%, transparent 0 100%) 0 0 / 2rem 1rem;
  mask: linear-gradient(-90deg, black, transparent);
}
```

demo 地址：[Gradient Mask Divider](https://codepen.io/alphardex/pen/WNwyjqw)

和 clip-path 结合也会相当有意思，如下图所示的加载特效

<img src="https://user-gold-cdn.xitu.io/2020/7/7/17327c58fc62bf7d?imageView2/2/w/800/q/85" referrerpolicy="no-referrer">

demo 地址：[Mask Loader](https://codepen.io/alphardex/pen/LYGdxXa)

# CSS 变量

## 鼠标跟踪

上篇提到了利用 Web Animations API 实现鼠标悬浮跟踪的效果，但其实 CSS 变量也能实现，而且更加简洁高效

在 CSS 中定义 x 和 y 变量，然后在 JS 中监听鼠标移动事件并获取鼠标坐标，更新对应的 x 和 y 变量即可

```css
:root {
  --mouse-x: 0;
  --mouse-y: 0;
}

.target {
  transform: translate(var(--mouse-x), var(--mouse-y));
}
```

```js
let mouseX = 0;
let mouseY = 0;
let x = 0;
let y = 0;
let offset = 50; // center
let windowWidth = window.innerWidth;
let windowHeight = window.innerHeight;
const percentage = (value, total) => (value / total) * 100;

window.addEventListener("mousemove", (e) => {
  mouseX = e.clientX;
  mouseY = e.clientY;
  x = percentage(mouseX, windowWidth) - offset;
  y = percentage(mouseY, windowHeight) - offset;
  document.documentElement.style.setProperty("--mouse-x", `${x}%`);
  document.documentElement.style.setProperty("--mouse-y", `${y}%`);
});

window.addEventListener("resize", () => {
  windowWidth = window.innerWidth;
  windowHeight = window.innerHeight;
});
```

<img src="https://user-gold-cdn.xitu.io/2020/7/2/1730ef4ff4d4837f?imageView2/2/w/800/q/85" referrerpolicy="no-referrer"/>

简化版地址：[Mousemove Starter](https://codepen.io/alphardex/pen/abNKBdJ)

### 残影效果

如果将鼠标跟踪和交错动画结合起来，再加点模糊滤镜，就能创作出帅气的残影效果

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abe7f7ba794f4e759b7758e4ff25b64b~tplv-k3u1fbpfcp-zoom-1.image?imageslim)

本 demo 地址：[Motion Table - Delay](https://codepen.io/alphardex/pen/xxVMEYp?editors=0110)

## 图片分割

为了做出一个图片碎片运动相关的动画，或者是一个拼图游戏，我们就要对一张图片进行分割，且块数、大小等都能随意控制，这时CSS变量就能发挥它的用场了

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64639f4345124481abf3f088eee194a8~tplv-k3u1fbpfcp-zoom-1.image)

``` scss
.puzzle {
  --puzzle-width: 16rem;
  --puzzle-height: 24rem;
  --puzzle-row: 3;
  --puzzle-col: 4;
  --puzzle-gap: 1px;
  --puzzle-frag-width: calc(var(--puzzle-width) / var(--puzzle-col));
  --puzzle-frag-height: calc(var(--puzzle-height) / var(--puzzle-row));
  --puzzle-img: url(...);

  display: flex;
  flex-wrap: wrap;
  width: calc(var(--puzzle-width) + calc(var(--puzzle-col) * var(--puzzle-gap) * 2));
  height: calc(var(--puzzle-height) + calc(var(--puzzle-row) * var(--puzzle-gap) * 2));

  .fragment {
    --x-offset: calc(var(--x) * var(--puzzle-frag-width) * -1);
    --y-offset: calc(var(--y) * var(--puzzle-frag-height) * -1);
 
    width: var(--puzzle-frag-width);
    height: var(--puzzle-frag-height);
    margin: var(--puzzle-gap);
    background: var(--puzzle-img) var(--x-offset) var(--y-offset) / var(--puzzle-width) var(--puzzle-height) no-repeat;
  }
}
```

1. 设定好分割的行列，根据行列来动态计算切片的大小
2. 拼图的总宽|高=拼图宽|高+列|行数 \* 间隙 \* 2
3. 切片的显示利用背景定位的xy轴偏移，偏移量的计算方式：x|y坐标 \* 切片宽|高 \* -1

在JS中，设定好变量值并动态生成切片的xy坐标，即可完成图片的分割

```js
class Puzzle {
  constructor(el, width = 16, height = 24, row = 3, col = 3, gap = 1) {
    this.el = el;
    this.fragments = el.children;
    this.width = width;
    this.height = height;
    this.row = row;
    this.col = col;
    this.gap = gap;
  }

  create() {
    this.ids = [...Array(this.row * this.col).keys()];
    const puzzle = this.el;
    const fragments = this.fragments;
    if (fragments.length) {
      Array.from(fragments).forEach((item) => item.remove());
    }
    puzzle.style.setProperty("--puzzle-width", this.width + "rem");
    puzzle.style.setProperty("--puzzle-height", this.height + "rem");
    puzzle.style.setProperty("--puzzle-row", this.row);
    puzzle.style.setProperty("--puzzle-col", this.col);
    puzzle.style.setProperty("--puzzle-gap", this.gap + "px");
    for (let i = 0; i < this.row; i++) {
      for (let j = 0; j < this.col; j++) {
        const fragment = document.createElement("div");
        fragment.className = "fragment";
        fragment.style.setProperty("--x", j);
        fragment.style.setProperty("--y", i);
        fragment.style.setProperty("--i", j + i * this.col);
        puzzle.appendChild(fragment);
      }
    }
  }
}

const puzzle = new Puzzle(document.querySelector(".puzzle"));
```

本demo地址：[Split Image With CSS Variable](https://codepen.io/alphardex/pen/XWdypva)

# 复杂动画

## 案例1

<img src="https://user-gold-cdn.xitu.io/2020/2/28/1708a99b931467d7?imageView2/2/w/800/q/85">

本demo地址：[Elastic Love](https://codepen.io/alphardex/pen/gOpWpjq)

## 案例2

![](https://i.loli.net/2020/10/09/APNQgjZLomvqVJ2.gif)

本demo地址：[Infinite Line Animation](https://codepen.io/alphardex/pen/YzqOZOW)

## 案例3

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/135abf85ba5c40a9a5b2d6dec69c0d7c~tplv-k3u1fbpfcp-zoom-1.image)

本demo地址：[Orbit Reverse](https://codepen.io/alphardex/pen/jOqgyRm)

## 案例4

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0d528f8b4634bd4acd3422723d8f772~tplv-k3u1fbpfcp-zoom-1.image)

本demo地址：[Motion Table - Solid Rotation](https://codepen.io/alphardex/pen/ExKzwwX)

## 案例5

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dee916d09aab4f2ea6d60aa482627b1d~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)

本demo地址：[Motion Table - Symmetric Move](https://codepen.io/alphardex/pen/jOqQgzy)

## 小结

以上几个复杂的动画或多或少都有以下的特征：

1. `div`很多，对布局的要求很高
2. `@keyframes`很多，对动画的要求很高
3. 有的动画有较多的3d变换

案例5的教程已经写在之前的博文“[画物语——CSS动画之美](https://juejin.im/post/6878192439732109326)”里了，其余案例亦可以用此文提到的方法进行研究

笔者的CSS动画作品全放在这个集合里了：[CSS Animation Collection](https://codepen.io/collection/DrPkOq?cursor=ZD0xJm89MSZwPTEmdj0xNg==)

# 彩蛋

螺旋阶梯动画（灵感来自灰色的果实OP）

![](https://user-gold-cdn.xitu.io/2020/6/14/172b14fc0e9d22c1?imageView2/1/w/460/h/316/q/85/format/jpg/interlace/1)

本demo地址：[Spiral Stair Loading](https://codepen.io/alphardex/pen/OJMXOVR)