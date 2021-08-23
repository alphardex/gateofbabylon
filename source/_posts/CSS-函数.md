title: 有趣的 CSS 数学函数
author: alphardex
abbrlink: 38775
date: 2021-08-19 17:20:09
tags:

---

## 前言

大家好，这里是 CSS 兼 WebGL 魔法使——alphardex。

之前一直在玩 three.js ，接触了很多数学函数，用它们创造过很多特效。于是我思考：能否在 CSS 中也用上这些数学函数，但发现 CSS 目前还没有，据说以后的新规范会纳入，估计也要等很久。

然而，我们可以通过一些小技巧，来创作出一些属于自己的 CSS 数学函数，从而实现一些有趣的动画效果。

让我们开始吧！

<!--more-->

## CSS 数学函数

注意：以下的函数用原生 CSS 也都能实现，这里用 SCSS 函数只是为了方便封装，封装起来的话更方便调用

### 绝对值

绝对值就是正的还是正的，负的变为正的

可以创造 2 个数，其中一个数是另一个数的相反数，比较它们的最大值，即可获得这个数的绝对值

```scss
@function abs($v) {
  @return max(#{$v}, calc(-1 * #{$v}));
}
```

### 中位数

原数减 1 并乘以一半即可

```scss
@function middle($v) {
  @return calc(0.5 * (#{$v} - 1));
}
```

### 数轴上两点距离

数轴上两点距离就是两点所表示数字之差的绝对值，有了上面的绝对值公式就可以直接写出来

```scss
@function dist-1d($v1, $v2) {
  $v-delta: calc(#{$v1} - #{$v2});
  @return #{abs($v-delta)};
}
```

### 三角函数

其实这个笔者也不会实现~不过之前看到过[好友 chokcoco 的一篇文章](https://github.com/chokcoco/iCSS/issues/72)写到了如何在 CSS 中实现三角函数，在此表示感谢

```scss
@function fact($number) {
  $value: 1;
  @if $number>0 {
    @for $i from 1 through $number {
      $value: $value * $i;
    }
  }
  @return $value;
}

@function pow($number, $exp) {
  $value: 1;
  @if $exp>0 {
    @for $i from 1 through $exp {
      $value: $value * $number;
    }
  } @else if $exp < 0 {
    @for $i from 1 through -$exp {
      $value: $value / $number;
    }
  }
  @return $value;
}

@function rad($angle) {
  $unit: unit($angle);
  $unitless: $angle / ($angle * 0 + 1);
  @if $unit==deg {
    $unitless: $unitless / 180 * pi();
  }
  @return $unitless;
}

@function pi() {
  @return 3.14159265359;
}

@function sin($angle) {
  $sin: 0;
  $angle: rad($angle);
  // Iterate a bunch of times.
  @for $i from 0 through 20 {
    $sin: $sin + pow(-1, $i) * pow($angle, (2 * $i + 1)) / fact(2 * $i + 1);
  }
  @return $sin;
}

@function cos($angle) {
  $cos: 0;
  $angle: rad($angle);
  // Iterate a bunch of times.
  @for $i from 0 through 20 {
    $cos: $cos + pow(-1, $i) * pow($angle, 2 * $i) / fact(2 * $i);
  }
  @return $cos;
}

@function tan($angle) {
  @return sin($angle) / cos($angle);
}
```

## 例子

以下的几个动画特效演示了上面数学函数的作用

### 一维交错动画

#### 初始状态

创建一排元素，用内部阴影填充，准备好我们的数学函数

```html
<div class="list">
  <div class="list-item"></div>
  ...(此处省略14个 list-item)
  <div class="list-item"></div>
</div>
```

```scss
body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
  background: #222;
}

:root {
  --blue-color-1: #6ee1f5;
}

(这里复制粘贴上文所有的数学公式)

.list {
  --n: 16;

  display: flex;
  flex-wrap: wrap;
  justify-content: space-evenly;

  &-item {
    --p: 2vw;
    --gap: 1vw;
    --bg: var(--blue-color-1);

    @for $i from 1 through 16 {
      &:nth-child(#{$i}) {
        --i: #{$i};
      }
    }

    padding: var(--p);
    margin: var(--gap);
    box-shadow: inset 0 0 0 var(--p) var(--bg);
  }
}
```

[![fb7wZV.png](https://z3.ax1x.com/2021/08/19/fb7wZV.png)](https://imgtu.com/i/fb7wZV)

#### 应用动画

这里用了 2 个动画：grow 负责将元素缩放出来；melt 负责“融化”元素（即消除阴影的扩散半径）

```html
<div class="list grow-melt">
  <div class="list-item"></div>
  ...(此处省略14个 list-item)
  <div class="list-item"></div>
</div>
```

```scss
.list {
  &.grow-melt {
    .list-item {
      --t: 2s;

      animation-name: grow, melt;
      animation-duration: var(--t);
      animation-iteration-count: infinite;
    }
  }
}

@keyframes grow {
  0% {
    transform: scale(0);
  }

  50%,
  100% {
    transform: scale(1);
  }
}

@keyframes melt {
  0%,
  50% {
    box-shadow: inset 0 0 0 var(--p) var(--bg);
  }

  100% {
    box-shadow: inset 0 0 0 0 var(--bg);
  }
}
```

[![fqkIkF.gif](https://z3.ax1x.com/2021/08/19/fqkIkF.gif)](https://imgtu.com/i/fqkIkF)

#### 交错动画

1. 计算出元素下标的中位数
2. 计算每个元素 id 到这个中位数的距离
3. 根据距离算出比例
4. 根据比例算出 delay

```html
<div class="list grow-melt middle-stagger">
  <div class="list-item"></div>
  ...(此处省略14个 list-item)
  <div class="list-item"></div>
</div>
```

```scss
.list {
  &.middle-stagger {
    .list-item {
      --m: #{middle(var(--n))}; // 中位数，这里是7.5
      --i-m-dist: #{dist-1d(var(--i), var(--m))}; // 计算每个id到中位数之间的距离
      --ratio: calc(var(--i-m-dist) / var(--m)); // 根据距离算出比例
      --delay: calc(var(--ratio) * var(--t)); // 根据比例算出delay
      --n-delay: calc((var(--ratio) - 2) * var(--t)); // 负delay表示动画提前开始

      animation-delay: var(--n-delay);
    }
  }
}
```

[![fqkzkD.gif](https://z3.ax1x.com/2021/08/19/fqkzkD.gif)](https://imgtu.com/i/fqkzkD)

地址：[Symmetric Line Animation](https://codepen.io/alphardex/pen/vYmqvpe)

### 二维交错动画

#### 初始状态

如何将一维的升成二维？应用网格系统即可

```html
<div class="grid">
  <div class="grid-item"></div>
  ...(此处省略62个 grid-item)
  <div class="grid-item"></div>
</div>
```

```scss
.grid {
  $row: 8;
  $col: 8;
  --row: #{$row};
  --col: #{$col};
  --gap: 0.25vw;

  display: grid;
  gap: var(--gap);
  grid-template-rows: repeat(var(--row), 1fr);
  grid-template-columns: repeat(var(--col), 1fr);

  &-item {
    --p: 2vw;
    --bg: var(--blue-color-1);

    @for $y from 1 through $row {
      @for $x from 1 through $col {
        $k: $col * ($y - 1) + $x;
        &:nth-child(#{$k}) {
          --x: #{$x};
          --y: #{$y};
        }
      }
    }

    padding: var(--p);
    box-shadow: inset 0 0 0 var(--p) var(--bg);
  }
}
```

[![fLsvPx.png](https://z3.ax1x.com/2021/08/20/fLsvPx.png)](https://imgtu.com/i/fLsvPx)

#### 应用动画

跟上面的动画一模一样

```html
<div class="grid grow-melt">
  <div class="grid-item"></div>
  ...(此处省略62个 grid-item)
  <div class="grid-item"></div>
</div>
```

```scss
.grid {
  &.grow-melt {
    .grid-item {
      --t: 2s;

      animation-name: grow, melt;
      animation-duration: var(--t);
      animation-iteration-count: infinite;
    }
  }
}
```

[![fLsGvD.gif](https://z3.ax1x.com/2021/08/20/fLsGvD.gif)](https://imgtu.com/i/fLsGvD)

#### 交错动画

1. 计算出网格行列的中位数
2. 计算网格 xy 坐标到中位数的距离并求和
3. 根据距离算出比例
4. 根据比例算出 delay

```html
<div class="grid grow-melt middle-stagger">
  <div class="grid-item"></div>
  ...(此处省略62个 grid-item)
  <div class="grid-item"></div>
</div>
```

```scss
.grid {
  &.middle-stagger {
    .grid-item {
      --m: #{middle(var(--col))}; // 中位数，这里是7.5
      --x-m-dist: #{dist-1d(var(--x), var(--m))}; // 计算x坐标到中位数之间的距离
      --y-m-dist: #{dist-1d(var(--y), var(--m))}; // 计算y坐标到中位数之间的距离
      --dist-sum: calc(var(--x-m-dist) + var(--y-m-dist)); // 距离之和
      --ratio: calc(var(--dist-sum) / var(--m)); // 根据距离和计算比例
      --delay: calc(var(--ratio) * var(--t) * 0.5); // 根据比例算出delay
      --n-delay: calc(
        (var(--ratio) - 2) * var(--t) * 0.5
      ); // 负delay表示动画提前开始

      animation-delay: var(--n-delay);
    }
  }
}
```

[![fL2Ppt.gif](https://z3.ax1x.com/2021/08/20/fL2Ppt.gif)](https://imgtu.com/i/fL2Ppt)

地址：[Symmetric Grid Animation](https://codepen.io/alphardex/pen/zYwgdZO)

#### 另一种动画

可以换一种动画 shuffle（穿梭），会产生另一种奇特的效果

```html
<div class="grid shuffle middle-stagger">
  <div class="grid-item"></div>
  ...(此处省略254个 grid-item )
  <div class="grid-item"></div>
</div>
```

```scss
.grid {
  $row: 16;
  $col: 16;
  --row: #{$row};
  --col: #{$col};
  --gap: 0.25vw;

  &-item {
    --p: 1vw;

    transform-origin: bottom;
    transform: scaleY(0.1);
  }

  &.shuffle {
    .grid-item {
      --t: 2s;

      animation: shuffle var(--t) infinite ease-in-out alternate;
    }
  }
}

@keyframes shuffle {
  0% {
    transform: scaleY(0.1);
  }

  50% {
    transform: scaleY(1);
    transform-origin: bottom;
  }

  50.01% {
    transform-origin: top;
  }

  100% {
    transform-origin: top;
    transform: scaleY(0.1);
  }
}
```

[![fOJSZ8.gif](https://z3.ax1x.com/2021/08/20/fOJSZ8.gif)](https://imgtu.com/i/fOJSZ8)

地址：[Shuffle Grid Animation](https://codepen.io/alphardex/pen/YzVmYaV)

### 余弦波动动画

#### 初始状态

创建 7 个不同颜色的（这里直接选了彩虹色）列表，每个列表有 40 个子元素，每个子元素是一个小圆点

让这 7 个列表排列在一条线上，且 z 轴上距离错开，设置好基本的 delay

```html
<div class="lists">
  <div class="list">
    <div class="list-item"></div>
    ...(此处省略39个 list-item)
  </div>
  ...(此处省略6个 list)
</div>
```

```scss
.lists {
  $list-count: 7;
  $colors: red, orange, yellow, green, cyan, blue, purple;

  position: relative;
  width: 34vw;
  height: 2vw;
  transform-style: preserve-3d;
  perspective: 800px;

  .list {
    position: absolute;
    top: 0;
    left: 0;
    display: flex;
    transform: translateZ(var(--z));

    @for $i from 1 through $list-count {
      &:nth-child(#{$i}) {
        --bg: #{nth($colors, $i)};
        --z: #{$i * -1vw};
        --basic-delay-ratio: #{$i / $list-count};
      }
    }

    &-item {
      --w: 0.6vw;
      --gap: 0.15vw;

      width: var(--w);
      height: var(--w);
      margin: var(--gap);
      background: var(--bg);
      border-radius: 50%;
    }
  }
}
```

[![hSdtfI.png](https://z3.ax1x.com/2021/08/22/hSdtfI.png)](https://imgtu.com/i/hSdtfI)

#### 余弦排列

运用上文的三角函数公式，让这些小圆点以余弦的一部分形状进行排列

```scss
.lists {
  .list {
    &-item {
      $item-count: 40;
      $offset: pi() * 0.5;
      --wave-length: 21vw;

      @for $i from 1 through $item-count {
        &:nth-child(#{$i}) {
          --i: #{$i};
          $ratio: ($i - 1) / ($item-count - 1);
          $angle-unit: pi() * $ratio;
          $wave: cos($angle-unit + $offset);
          --single-wave-length: calc(#{$wave} * var(--wave-length));
          --n-single-wave-length: calc(var(--single-wave-length) * -1);
        }
      }

      transform: translateY(var(--n-single-wave-length));
    }
  }
}
```

[![hSwuNj.png](https://z3.ax1x.com/2021/08/22/hSwuNj.png)](https://imgtu.com/i/hSwuNj)

#### 波动动画

对每个小圆点应用上下平移动画，平移的距离就是余弦的波动距离

```scss
.lists {
  .list {
    &-item {
      --t: 2s;

      animation: wave var(--t) infinite ease-in-out alternate;
    }
  }
}

@keyframes wave {
  from {
    transform: translateY(var(--n-single-wave-length));
  }

  to {
    transform: translateY(var(--single-wave-length));
  }
}
```

[![hSwfPA.gif](https://z3.ax1x.com/2021/08/22/hSwfPA.gif)](https://imgtu.com/i/hSwfPA)

#### 交错动画

跟上面一个套路，计算从中间开始的 delay，再应用到动画上即可

```scss
.lists {
  .list {
    &-item {
      --n: #{$item-count + 1};
      --m: #{middle(var(--n))};
      --i-m-dist: #{dist-1d(var(--i), var(--m))};
      --ratio: calc(var(--i-m-dist) / var(--m));
      --square: calc(var(--ratio) * var(--ratio));
      --delay: calc(
        calc(var(--square) + var(--basic-delay-ratio) + 1) * var(--t)
      );
      --n-delay: calc(var(--delay) * -1);

      animation-delay: var(--n-delay);
    }
  }
}
```

[![hSwqaQ.gif](https://z3.ax1x.com/2021/08/22/hSwqaQ.gif)](https://imgtu.com/i/hSwqaQ)

地址：[Rainbow Sine](https://codepen.io/alphardex/pen/GREKJbL)

## 最后

CSS 数学函数能实现的特效远不止于此，希望通过本文能激起大家创作特效的灵感~
