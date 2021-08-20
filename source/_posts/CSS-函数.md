title: 有趣的 CSS 数学函数
author: alphardex
abbrlink: 38775
date: 2021-08-19 17:20:09
tags:

---

## 前言

大家好，这里是 CSS 兼 WebGL 魔法使——alphardex。

之前一直在玩 three.js ，接触了很多数学函数，用它们创造过很多特效。于是我思考：能否在 CSS 中也用上这些数学函数，但发现 CSS 目前还没有，据说以后的新规范会纳入，估计也要等很久。

然而，我们可以通过一些小技巧，来创作出一些属于自己的 CSS 数学函数。

让我们开始吧！

## CSS 数学函数

注意：以下的函数用原生CSS也都能实现，这里用SCSS函数只是为了方便封装，封装起来的话更方便调用

### 绝对值

绝对值就是正的还是正的，负的变为正的

可以创造2个数，其中一个数是另一个数的相反数，比较它们的最大值，即可获得这个数的绝对值

```scss
@function abs($v) {
  @return max(#{$v}, calc(-1 * #{$v}));
}
```

### 中位数

原数减1并乘以一半即可

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

## 例子

以下的几个动画特效演示了上面数学函数的作用

### 一维交错动画

#### 初始状态

创建一排元素，用内部阴影填充，准备好我们的数学函数

```html
<div class="list">
  <div class="list-item"></div>
  ...(此处省略14个)
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

// 绝对值
@function abs($v) {
  @return max(#{$v}, calc(-1 * #{$v}));
}

// 中位数
@function middle($v) {
  @return calc(0.5 * (#{$v} - 1));
}

// 数轴上两点距离
@function dist-1d($v1, $v2) {
  $v-delta: calc(#{$v1} - #{$v2});
  @return #{abs($v-delta)};
}

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

这里用了2个动画：grow负责将元素缩放出来；melt负责“融化”元素（即消除阴影的扩散半径）

```html
<div class="list grow-melt" style="--n:16">
  <div class="list-item" style="--i:1"></div>
  ...
  <div class="list-item" style="--i:16"></div>
</div>
```

```scss
.list {
	...

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

#### 中间交错

1. 计算出元素下标的中位数
2. 计算每个元素id到这个中位数的距离
3. 根据距离算出比例
4. 根据比例算出delay

```html
<div class="list grow-melt middle-stagger" style="--n:16">
  <div class="list-item" style="--i:1"></div>
  ...
  <div class="list-item" style="--i:16"></div>
</div>
```

```scss
.list {
  ...

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
  ...(此处省略62个)
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
  ...(此处省略62个)
  <div class="grid-item"></div>
</div>
```

```scss
.grid {
  ...

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

#### 中间交错

1. 计算出网格行列的中位数
2. 计算网格xy坐标到中位数的距离并求和
3. 根据距离算出比例
4. 根据比例算出delay

```html
<div class="grid grow-melt middle-stagger">
  <div class="grid-item"></div>
  ...(此处省略62个)
  <div class="grid-item"></div>
</div>
```

```scss
.grid {
  ...

  &.middle-stagger {
    .grid-item {
      --m: #{middle(var(--col))}; // 中位数，这里是7.5
      --x-m-dist: #{dist-1d(var(--x), var(--m))}; // 计算x坐标到中位数之间的距离
      --y-m-dist: #{dist-1d(var(--y), var(--m))}; // 计算y坐标到中位数之间的距离
      --dist-sum: calc(var(--x-m-dist) + var(--y-m-dist)); // 距离之和
      --ratio: calc(var(--dist-sum) / var(--m)); // 根据距离和计算比例
      --delay: calc(var(--ratio) * var(--t) * 0.5); // 根据比例算出delay
      --n-delay: calc((var(--ratio) - 2) * var(--t) * 0.5); // 负delay表示动画提前开始

      animation-delay: var(--n-delay);
    }
  }
}
```

[![fL2Ppt.gif](https://z3.ax1x.com/2021/08/20/fL2Ppt.gif)](https://imgtu.com/i/fL2Ppt)

地址：[Symmetric Grid Animation](https://codepen.io/alphardex/pen/zYwgdZO)