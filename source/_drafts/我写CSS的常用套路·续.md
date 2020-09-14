title: 我写CSS的常用套路·续
author: alphardex
abbrlink: 32261
date: 2020-09-14 10:09:09
tags:
---
# 前言

前篇传送门：[猛戳这里](https://juejin.im/post/6844904033405108232)

<!--more-->

# 3D方块

如何在CSS中创建立体的方块呢？用以下的mixin即可

方块的长度、高度、深度都可以通过CSS变量自由调节

```scss
@mixin cube($width, $height, $depth) {
  width: var($width);
  height: var($height);

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
  transform: rotateX(-90deg) rotateY(180deg) translateX(calc(var(#{$width}) * -1))
    translateY(calc(var(#{$height}) * -1));
}

@mixin cube-right($width, $height, $depth) {
  width: calc(var(#{$depth}) * 2);
  height: var($height);
  transform-origin: top left;
  transform: rotateY(90deg) rotateZ(-90deg) translateZ(var(#{$width}))
    translateX(calc(var(#{$depth}) * -2)) translateY(calc(var(#{$height}) * -1));
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
}
```
利用CSS Houdini，我们可以使方块的长度变量动起来
``` js
CSS.registerProperty({
  name: '--cube-width',
  syntax: '<length>',
  initialValue: 0,
  inherits: true
})

CSS.registerProperty({
  name: '--cube-height',
  syntax: '<length>',
  initialValue: 0,
  inherits: true
})

CSS.registerProperty({
  name: '--cube-depth',
  syntax: '<length>',
  initialValue: 0,
  inherits: true
})
```
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/734029e9456449db98f013dde3d8dcfe~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)
本demo地址：[3D Stair Loading](https://codepen.io/alphardex/pen/YzqrGXb)

用gsap来控制方块的运动也会相当有趣

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e87bc25a2ce426993a84cfa0503419a~tplv-k3u1fbpfcp-zoom-1.image)

该动画的灵感来源：[凭物语第3集开头](https://www.acfun.cn/bangumi/aa6003168_36123_1739764)

本demo地址：[3D Puzzle Animation](https://codepen.io/alphardex/pen/wvGXwgL)

# 文本分割

在上一篇我们提到了如何用JS来分割文本，本篇将介绍一种更简洁的实现方法——gsap的SplitText插件，利用它我们能用更少的代码来实现下图的效果

``` html
<div class="land-in font-bold text-2xl">Fushigi no Monogatari</div>
```

```js
const t1 = gsap.timeline();
const splitText = new SplitText('.land-in', { type: 'chars' });
const chars = splitText.chars;
t1.from(chars, { duration: 0.8, ease: 'power1.out', opacity: 0, y: "-20%", stagger: 0.05 })
```

<img src="https://user-gold-cdn.xitu.io/2020/2/8/1702507ba8f9dbf3?imageslim" referrerpolicy="no-referrer"/>

简化版demo地址：[SplitText Starter](https://codepen.io/alphardex/pen/ZEWRBJp)

# SVG滤镜

CSS的滤镜其实都是SVG滤镜的封装版本，方便我们使用而已

SVG滤镜则更加灵活强大，以下是几个常见的滤镜使用场景

推荐一个在线调SVG滤镜的网站：[SVG Filters](https://yoksel.github.io/svg-filters/#/)

## 粘滞效果

```html
<svg width="0" height="0" class="absolute">
  <filter id="goo">
    <feGaussianBlur stdDeviation="10 10" in="SourceGraphic" result="blur" />
    <feColorMatrix type="matrix" values="1 0 0 0 0
    0 1 0 0 0
    0 0 1 0 0
    0 0 0 18 -7" in="blur" result="colormatrix" />
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

本demo地址：[SVG Filter Gooey Menu](https://codepen.io/alphardex/pen/GRZOvwJ)

## 故障效果

```html
<svg width="0" height="0" class="absolute">
  <filter id="glitch">
    <feTurbulence type="fractalNoise" baseFrequency="0.00001 0.000001" numOctaves="1" result="turbulence1">
      <animate attributeName="baseFrequency" from="0.00001 0.000001" to="0.00001 0.4" dur="0.4s" id="glitch1" fill="freeze" repeatCount="indefinite"></animate>
      <animate attributeName="baseFrequency" from="0.00001 0.4" to="0.00001 0.2" dur="0.2s" begin="glitch1.end" fill="freeze" repeatCount="indefinite"></animate>
    </feTurbulence>
    <feDisplacementMap in="SourceGraphic" in2="turbulence1" scale="30" xChannelSelector="R" yChannelSelector="G" result="displacementMap" />
  </filter>
</svg>
```

```css
.glitch {
  filter: url('#glitch');
}
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/45881c14da48420b87f70a0b03296eb1~tplv-k3u1fbpfcp-zoom-1.image?imageView2/2/w/800/q/85)

本demo地址：[SVG Filter Glitch Button](https://codepen.io/alphardex/pen/mdPqmaM)

# 鼠标跟踪

上篇提到了利用Web Animations API实现鼠标悬浮跟踪的效果，但其实CSS变量也能实现，而且更加简洁高效

在CSS中定义x和y变量，然后在JS中监听鼠标移动事件并获取鼠标坐标，更新对应的x和y变量即可

```css
:root {
  --mouse-x: 0;
  --mouse-y: 0;
}

.target {
  transform: translate(var(--mouse-x), var(--mouse-y));
}
```

```ts
let mouseX = 0;
let mouseY = 0;
let x = 0;
let y = 0;
let offset = 50;  // center
let windowWidth = window.innerWidth;
let windowHeight = window.innerHeight;
const percentage = (value: number, total: number) => (value / total) * 100;

window.addEventListener('mousemove', (e) => {
  mouseX = e.clientX;
  mouseY = e.clientY;
  x = percentage(mouseX, windowWidth) - offset;
  y = percentage(mouseY, windowHeight) - offset;
  document.documentElement.style.setProperty('--mouse-x', `${x}%`);
  document.documentElement.style.setProperty('--mouse-y', `${y}%`);
})

window.addEventListener('resize', () => {
  windowWidth = window.innerWidth;
  windowHeight = window.innerHeight;
})
```

<img src="https://user-gold-cdn.xitu.io/2020/7/2/1730ef4ff4d4837f?imageView2/2/w/800/q/85" referrerpolicy="no-referrer"/>

本demo地址：[Cursor Hover Magnetize](https://codepen.io/alphardex/pen/NWxXGKb)

简化版地址：[Mousemove Starter](https://codepen.io/alphardex/pen/abNKBdJ)

# 遮罩

有时候我们想做出一种过渡式的半透明效果，类似下图这样的

![](https://i.loli.net/2020/09/14/VXrtQnbhZDAwiLc.png)

这时候就得借助mask属性了，因为图片与mask生成的渐变的transparent的重叠部分会变透明

```css
.divider-grad-mask {
  background: linear-gradient(90deg, var(--blue-color) 0 50%, transparent 0 100%) 0 0 / 2rem 1rem;
  mask: linear-gradient(-90deg, black, transparent);
}
```

demo地址：[Gradient Mask Divider](https://codepen.io/alphardex/pen/WNwyjqw)

和clip-path结合也会相当有意思，如下图所示的加载特效

<img src="https://user-gold-cdn.xitu.io/2020/7/7/17327c58fc62bf7d?imageView2/2/w/800/q/85">

demo地址：[Mask Loader](https://codepen.io/alphardex/pen/LYGdxXa)