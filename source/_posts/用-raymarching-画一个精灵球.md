title: 用 raymarching 画一个精灵球
author: alphardex
abbrlink: 58379
tags: []
categories: []
date: 2022-04-13 15:46:00
---
## 前言

大家好，这里是 CSS 兼 WebGL 魔法使——alphardex。本文就让我们来用[kokomi.js](https://github.com/alphardex/kokomi.js)的[raymarching组件](https://github.com/alphardex/marcher.js)来画一个精灵球吧~

泼给莫，给托打贼！

[![LKobMF.gif](https://s1.ax1x.com/2022/04/13/LKobMF.gif)](https://imgtu.com/i/LKobMF)

<!--more-->

## 绘画开始

首先fork下这个模板

https://codesandbox.io/s/kokomi-js-raymarching-starter-lk17vs

看到“从这开始吧~”的代码块，我们将从那里开始

### 材质

精灵球有3种颜色：黑色、白色和红色

首先定义好3种颜色的材质

```ts
// 材质
const BLACK_MAT = "1.0";
const WHITE_MAT = "2.0";
const RED_MAT = "3.0";

const mat = new marcher.SDFMaterial();
mat.addColorMaterial(BLACK_MAT, 0, 0, 0);
mat.addColorMaterial(WHITE_MAT, 255, 255, 255);
mat.addColorMaterial(RED_MAT, 255, 0, 0);
mar.setMaterial(mat);
```

### 球内

精灵球的内部是一个小黑球

```ts
const sphere = new marcher.SphereSDF({
  sdfVarName: "d1",
  materialId: BLACK_MAT,
});
layer.addPrimitive(sphere);
```

[![LK4vIP.png](https://s1.ax1x.com/2022/04/13/LK4vIP.png)](https://imgtu.com/i/LK4vIP)

### 按钮

中间有个白色的按钮，其实就是个圆柱体

```ts
const button = new marcher.CylinderSDF({
  sdfVarName: "d2",
  materialId: WHITE_MAT,
  radius: 0.1,
  height: 0.54,
});
button.rotate(90, "x");
layer.addPrimitive(button);
```

[![LK5GIx.png](https://s1.ax1x.com/2022/04/13/LK5GIx.png)](https://imgtu.com/i/LK5GIx)

### 上下球壳

raymarching里只能创造整个球体，那如果只想要球体的一半呢？

创建一个临时用的方块，将其移动到球的半侧，再和球体求交集，就能获取一半的球体了

```ts
// 球壳（上）
const shellUpper = new marcher.SphereSDF({
  sdfVarName: "d3",
  materialId: "3",
  radius: 0.55,
});

const clipBoxUpper = new marcher.BoxSDF({
  sdfVarName: "d4",
  width: 0.55,
  height: 0.55,
  depth: 0.55,
});
clipBoxUpper.hide();
clipBoxUpper.translate(0, -0.6, 0);

layer.addPrimitive(clipBoxUpper);
layer.addPrimitive(shellUpper);

shellUpper.intersect(clipBoxUpper);

// 球壳（下）
const shellLower = new marcher.SphereSDF({
  sdfVarName: "d6",
  materialId: WHITE_MAT,
  radius: 0.55,
});

const clipBoxLower = new marcher.BoxSDF({
  sdfVarName: "d5",
  width: 0.55,
  height: 0.55,
  depth: 0.55,
});
clipBoxLower.hide();
clipBoxLower.translate(0, 0.6, 0);

layer.addPrimitive(clipBoxLower);
layer.addPrimitive(shellLower);

shellLower.intersect(clipBoxLower);
```

[![LKI0pT.png](https://s1.ax1x.com/2022/04/13/LKI0pT.png)](https://imgtu.com/i/LKI0pT)

这里法线中间的按钮还是被球壳给挡住了，我们就要把挡住的部分给挖空

### 中间镂空

创建2个圆柱体（类似之前的按钮），半径和高度稍微调大点，用它们分别对上下的球壳进行扣除操作就能起到挖空中间的效果

```ts
// 球壳（上）：挖除中间镂空部分后
const clipCylinderCenter1 = new marcher.CylinderSDF({
  sdfVarName: "d7",
  radius: 0.15,
  height: 0.6,
  materialId: RED_MAT,
});
clipCylinderCenter1.rotate(90, "x");

layer.addPrimitive(clipCylinderCenter1);

clipCylinderCenter1.subtract(shellUpper);

shellUpper.hide();

// 球壳下：挖除中间镂空部分后
const clipCylinderCenter2 = new marcher.CylinderSDF({
  sdfVarName: "d8",
  radius: 0.15,
  height: 0.6,
  materialId: WHITE_MAT,
});
clipCylinderCenter2.rotate(90, "x");

layer.addPrimitive(clipCylinderCenter2);

clipCylinderCenter2.subtract(shellLower);

shellLower.hide();
```

[![LKoECV.png](https://s1.ax1x.com/2022/04/13/LKoECV.png)](https://imgtu.com/i/LKoECV)

### 美化

这里可以自由发挥，不论是光照、相机视角还是材质等要素都可以优化

笔者优化后的结果如下

[![LKobMF.gif](https://s1.ax1x.com/2022/04/13/LKobMF.gif)](https://imgtu.com/i/LKobMF)

## 项目地址

https://code.juejin.cn/pen/7085983190216605700