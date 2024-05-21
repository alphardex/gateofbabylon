title: 纯 CSS 实现常见的 UI 效果
author: alphardex
abbrlink: 24238
tags: []
categories: []
date: 2021-01-04 14:28:00

---

## 前言

大家好，这里是 CSS 魔法使——alphardex。

切图仔，是大多数前端用来自嘲的称呼。相信很多人平时写页面的时候，大部分时间是在切图和排图，如此往复。这里并不是要否定切图本身，而是在质疑：一直切图到底对自己的功力增长有何好处？想想 UI 丢给你一套好看的界面，你却只需一个 img 标签，或者一个 background-image 属性即可搞定了它，但日后某个地方需要调整某些外观（颜色、文字等），你还不是会让 UI 再修改之前的素材，然后替换上去完事？这样就完全受制于 UI，而无法发挥自己的能动性。

那么，如何打破这个僵局？很简单，如果你 CSS 玩的够溜，你就无需再进行那枯燥无比的切图工作，那些界面、元素都是通过你双手亲自缔造而成的，尽管创作它们可能会花一些功夫，但带来的回报也是巨大的，你不仅能够自由掌控你所创造出来的元素，而且能大幅提高自己的 CSS 功力。

<!--more-->

## 在此之前

在用纯 CSS 实现这些效果之前，笔者先介绍几个常用的 SCSS Mixin 和一个得力武器，用它们来进行创作将会事半功倍

### 覆盖 - cover

```scss
@mixin cover($top: 0, $left: 0, $width: 100%, $height: 100%) {
  position: absolute;
  top: $top;
  left: $left;
  width: $width;
  height: $height;
}
```

当你想在原先元素的基础上再“复制”一个元素，并将其覆盖在它身上时，你将会用到它

![1.png](https://i.loli.net/2021/01/05/VvBJf73FAjHXDiq.png)

demo 地址：[Blob Button](https://codepen.io/alphardex/pen/GRjEoBZ)

### 嵌入 - inset

```scss
@mixin inset($inset: 0) {
  position: absolute;
  top: $inset;
  left: $inset;
  right: $inset;
  bottom: $inset;
}
```

同样地，这也是在原先元素基础上复制出一个元素，只不过这个元素位置和原先的元素相同，大小会基于原先的元素而增减。

举个例子，倘若你想创建多个半径不同的同心圆，这个 Mixin 将会很有帮助

### aqua.css

[aqua.css](https://aquacss.netlify.app/)是笔者开源的一个优雅的、轻量级的 CSS 框架。里面有很多常用的组件以及常用的样式类，用它来写 CSS 体验将会非常爽

在 codepen 上，笔者准备了一个[aqua.css 模版](https://codepen.io/alphardex/pen/LYVXwWd)，大家可以用它来进行 CSS 的创作

## 常见 UI 效果

### 条纹效果

![2.png](https://i.loli.net/2021/01/05/F7cPMJte9aNqEx2.png)

首先，我们要抓住“边框”这个词，如何创作出一个特殊的边框呢？如果一般的 CSS 属性实现不了的话，可以考虑用伪元素来实现，思路如下：在原先的元素下方创建一个有条纹背景的伪元素，并保证原先元素覆盖住它就行，这样就模拟了边框的效果。

那么如何创建条纹背景呢？这里我们将使用`repeating-linear-gradient`来实现它

```html
<div class="card w-80">
  <div class="border-stripe rounded-xl">Lorem ipsum...</div>
</div>
```

```scss
.border-stripe {
  --stripe-width: 0.5rem;
  --stripe-deg: -45deg;
  --stripe-color-1: var(--grey-color-1);
  --stripe-offset-1: 2px;
  --stripe-color-2: var(--skin-color-2);
  --stripe-offset-2: 1rem;
  --stripe-radius: 15px;
  --stripe-inset: calc(var(--stripe-width) * -1);

  &::before {
    @include inset(var(--stripe-inset));
    content: "";
    z-index: -1;
    background: repeating-linear-gradient(
      var(--stripe-deg),
      var(--stripe-color-1) 0 var(--stripe-offset-1),
      var(--stripe-color-2) 0 var(--stripe-offset-2)
    );
    border-radius: var(--stripe-radius);
  }
}
```

为了保证复用性，这里将其抽象成了`border-stripe`类，里面的值都可以通过 CSS 变量来动态调节

demo 地址：[Stripe Border](https://codepen.io/alphardex/pen/VwKWvdG)

### 光泽效果

![3.png](https://i.loli.net/2021/01/05/uiJOdb4fzLNP7ZU.png)

一看到光泽，相信你可能会想到一个关键角色——径向渐变，通过它，我们可以创作出放射状的图案，而光泽也恰好是放射状的，再根据背景可以叠加的特性，光泽效果就能轻松实现了

```html
<div class="flex flex-col space-y-4">
  <span class="btn btn-primary btn-round inline-flex">
    <span class="font-bold text-grad">Shine Button 1</span>
  </span>
  <span class="btn btn-info btn-round btn-depth inline-flex">
    <span class="font-bold">Shine Button 2</span>
  </span>
</div>
```

```scss
:root {
  --blue-color-1: #08123d;
  --gold-color-1: #dcb687;
  --brown-color-1: #50301f;
  --brown-color-2: #936237;
  --gold-grad-1: radial-gradient(
      circle at 50% 5%,
      #{transparentize(white, 0.5)},
      #eba262
    ), #eba262;
  --gold-grad-2: linear-gradient(88deg, #e7924e 0%, #f8ffee 50%, #e7924e 100%);
  --blue-grad-1: radial-gradient(
      circle at 50% 5%,
      #{transparentize(white, 0.8)},
      #091344
    ), #091344;
  --primary-color: var(--blue-grad-1);
  --info-color: var(--gold-grad-1);
}

.btn {
  &-primary {
    border: 4px solid var(--gold-color-1);

    span {
      background-image: var(--gold-grad-2);
    }
  }

  &-info {
    color: var(--brown-color-1);
    border: none;
  }

  &-depth {
    box-shadow: 0 -5px 0 var(--brown-color-2);
  }
}
```

demo 地址：[Shine Button](https://codepen.io/alphardex/details/vYXZNez)

### 不规则形状

![4.png](https://i.loli.net/2021/01/06/AnUpghKQa7y89Tt.png)

首先，让我们先观察一下上图的缎带形状是由哪些基本形状组成的：中间是一个矩形，矩形下方有 2 个三角形，左右 2 侧各有一个被裁切过的矩形。一提裁切，就能想到`clip-path`这个属性，于是问题也就很好解决了

```html
<div class="ribbon">
  Pure CSS Ribbon
  <div class="block"></div>
  <div class="block"></div>
  <div class="block"></div>
  <div class="block"></div>
</div>
```

```scss
.ribbon {
  --ribbon-color-1: var(--yellow-color-1);
  --ribbon-color-2: var(--yellow-color-2);
  --ribbon-color-3: var(--yellow-color-3);

  position: relative;
  padding: 0.5rem 1rem;
  color: white;
  background: var(--ribbon-color-1);

  .block {
    &:nth-child(1),
    &:nth-child(2) {
      position: absolute;
      bottom: -20%;
      width: 20%;
      height: 20%;
      background: var(--ribbon-color-2);
      clip-path: polygon(0 0, 100% 100%, 100% 0);
    }

    &:nth-child(1) {
      left: 0;
    }

    &:nth-child(2) {
      right: 0;
      transform: scaleX(-1);
    }

    &:nth-child(3),
    &:nth-child(4) {
      position: absolute;
      z-index: -1;
      top: 20%;
      width: 40%;
      height: 100%;
      background: var(--ribbon-color-3);
      clip-path: polygon(0 0, 25% 50%, 0 100%, 100% 100%, 100% 0);
    }

    &:nth-child(3) {
      left: -20%;
    }

    &:nth-child(4) {
      right: -20%;
      transform: scaleX(-1);
    }
  }
}
```

注意到有一行代码`transform: scaleX(-1);`，这起到了水平翻转的作用，它可以防止再写一遍`clip-path`

demo 地址：[Ribbon](https://codepen.io/alphardex/pen/OJRvaaR)

### 浮雕效果

![5.png](https://i.loli.net/2021/01/06/mt5CahWAGYIRL86.png)

通过仔细观察，你会发现这是由 2 个同心的元素组成的，于是自然就想到了`inset`这个 Mixin。

创建了 2 个同心元素后，就要想办法来创建它们的浮雕光泽了。这里的光泽可以用`box-shadow`来实现，通过叠加多重阴影，我们就能模拟出浮雕的效果了

```html
<div
  class="px-6 py-2 text-xl embossed cursor-pointer"
  data-text="浮雕按钮"
  style="--emboss-radius: 1.5rem"
>
  浮雕按钮
</div>
```

```scss
:root {
  --red-color-1: #af2222;
  --red-color-2: #c1423e;
  --red-color-3: #c62a2a;
  --red-color-4: #951110;
  --green-color-1: #486433;
  --green-color-2: #2b361a;
  --red-grad-1: linear-gradient(
    to right,
    var(--red-color-1) 50%,
    var(--red-color-2) 0
  );
}

.embossed {
  --emboss-radius: 1rem;
  --emboss-out: 6px;
  --emboss-out-minus: calc(var(--emboss-out) * -1);
  --emboss-inset: 2px;
  --emboss-inset-minus: calc(var(--emboss-inset) * -1);
  --emboss-blur: 1px;
  --emboss-bg-1: var(--red-color-3);
  --emboss-bg-2: var(--green-color-1);
  --emboss-color-1: white;
  --emboss-color-2: var(--red-color-4);
  --emboss-color-3: var(--green-color-2);

  position: relative;
  box-sizing: border-box;
  white-space: nowrap;

  &::before {
    @include inset(var(--emboss-out-minus));
    content: "";
    background: var(--emboss-bg-1);
    box-shadow: inset var(--emboss-inset-minus) var(--emboss-inset-minus) var(
          --emboss-blur
        ) var(--emboss-color-1), inset var(--emboss-inset) var(--emboss-inset)
        var(--emboss-blur) var(--emboss-color-2);
    border-radius: calc(var(--emboss-radius) + var(--emboss-out));
  }

  &::after {
    @include inset;
    @include flex-center;
    content: attr(data-text);
    color: white;
    font-weight: bold;
    background: var(--emboss-bg-2);
    box-shadow: inset var(--emboss-inset) var(--emboss-inset) var(--emboss-blur)
        var(--emboss-color-1), inset var(--emboss-inset-minus) var(
          --emboss-inset-minus
        )
        var(--emboss-blur) var(--emboss-color-3);
    border-radius: var(--emboss-radius);
  }
}
```

demo 地址：[Emboss Button](https://codepen.io/alphardex/pen/poEEERM?editors=0110)

## 课后作业

尝试用纯 CSS 来实现下图的效果，不准切图哦~

![6.png](https://i.loli.net/2021/01/06/CcD4mAPWZMHdSfr.png)

我的方案：[做完再点哦](https://codepen.io/alphardex/pen/gOweBBE)
