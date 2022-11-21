title: 用three.js创造时空裂缝
author: alphardex
abbrlink: 11844
date: 2022-11-21 14:48:05
tags:
---
呀哈喽！这里是alphardex。

最近受到轮回系作品（例如寒蝉等）中时空裂缝场景的启发，我用three.js来实现了一个实时渲染的时空裂缝场景。本文将简要地介绍下实现该效果的要点。

以下特效全屏观看效果最佳~

https://code.juejin.cn/pen/7168065415069827124

<!--more-->

## 建模

### 多边形形状

首先，创造一个最初始的平面

![1.png](https://s2.loli.net/2022/11/21/4tHm6XEWSLZBaJO.png)

建模，也就是定制化`geometry`

要想创建玻璃碎片一般的形状的话，也就是要创造一个多边形的形状

这就要用到kokomi.js的这2个函数[createPolygonShape](https://github.com/alphardex/kokomi.js/blob/main/src/utils/misc.ts#L146)和[polySort](https://github.com/alphardex/kokomi.js/blob/main/src/utils/math.ts#L16)：前者能接收一系列的点来创造一个多边形`Shape`，后者能给无序的点进行排序以符合多边形的描画

创建形状`Shape`后，再传进`ExtrudeGeometry`将其3D化成`geometry`即可，这里`depth`等值故意设得很小，是为了模拟玻璃碎片的纤细程度

```js
let points = [
  { x: 0, y: 0 },
  { x: 25, y: 0 },
  { x: 45, y: 45 },
  { x: 0, y: 25 },
];
points = kokomi.polySort(points);
const shape = kokomi.createPolygonShape(points, {
  scale: 0.01,
});
const geometry = new THREE.ExtrudeGeometry(shape, {
  steps: 1,
  depth: 0.0001,
  bevelEnabled: true,
  bevelThickness: 0.0005,
  bevelSize: 0.0005,
  bevelSegments: 1,
});
geometry.center();
```

![2.png](https://s2.loli.net/2022/11/21/Ik6m2Kzr3C1b7d8.png)

### 随机多边形

为了创建随机的多边形，我特意设计了一套算法，大致是这样的：

1. 多边形是按二维网格排布的，这样就能尽可能避免有重合的情况出现
2. 多边形的边数`edgeCount`按个人喜好用随机概率来控制
3. 多边形的第一个点决定了它在网格上的位置，其他的点是以它为圆心延伸出来的随机角度的点（跟圆有关因此用到了极坐标公式）

```js
const generatePolygons = (config = {}) => {
  const { gridX = 10, gridY = 20, maxX = 9, maxY = 9 } = config;

  const polygons = [];

  for (let i = 0; i < gridX; i++) {
    for (let j = 0; j < gridY; j++) {
      const points = [];
      let edgeCount = 3;
      const randEdgePossibility = Math.random();
      if (randEdgePossibility > 0 && randEdgePossibility <= 0.2) {
        edgeCount = 3;
      } else if (randEdgePossibility > 0.2 && randEdgePossibility <= 0.55) {
        edgeCount = 4;
      } else if (randEdgePossibility > 0.55 && randEdgePossibility <= 0.9) {
        edgeCount = 5;
      } else if (randEdgePossibility > 0.9 && randEdgePossibility <= 0.95) {
        edgeCount = 6;
      } else if (randEdgePossibility > 0.95 && randEdgePossibility <= 1) {
        edgeCount = 7;
      }
      let firstPoint = {
        x: 0,
        y: 0,
      };
      let angle = THREE.MathUtils.randFloat(0, 2 * Math.PI);
      for (let k = 0; k < edgeCount; k++) {
        if (k === 0) {
          firstPoint = {
            x: (i % maxX) * 10,
            y: (j % maxY) * 10,
          };
          points.push(firstPoint);
        } else {
          // random polar
          const r = 10;
          angle += THREE.MathUtils.randFloat(0, Math.PI / 2);
          const anotherPoint = {
            x: firstPoint.x + r * Math.cos(angle),
            y: firstPoint.y + r * Math.sin(angle),
          };
          points.push(anotherPoint);
        }
      }
      polygons.push(points);
    }
  }

  return polygons;
};
```

用该算法来创建多边形组，再调整下相机和多边形组的位置和缩放，就有了下图的效果

![3.png](https://s2.loli.net/2022/11/21/rKUQM7spOq52Ddj.png)

## 漂浮动画

将多边形组整体向上偏移，超出界限则重置高度

![4.gif](https://s2.loli.net/2022/11/21/SNEALzRVBmT4hbt.gif)

```js
let floatDistance = 0;
let floatSpeed = 1;
let floatMaxDistance = 1;

this.update(() => {
  floatDistance += floatSpeed;

  const y = floatDistance * 0.001;
  if (y > floatMaxDistance) {
    floatDistance = 0;
  }

  totalG.position.y = y;
});
```

将相机靠近，你就会觉得像是每个多边形在上升（其实是整体的容器在上升）

![5.gif](https://s2.loli.net/2022/11/21/9mLvnPFzCrVMQaw.gif)

要想达成无限上升的动画“假象”，我们需要拷贝一份多边形组，将它和之前的那组错开，这样动画就能无限衔接了

## 光照

这里可以自由表现，可以尝试以下几种手法：

1. 漫反射光和镜面反射光相结合
2. 扭曲顶点、法线和uv
3. 根据光线动态计算透明度，以形成玻璃般的效果

## 后期处理

同样也可以自由表现，可以尝试以下几种手法：

1. RGB扭曲（该特效所采用的）
2. 色差
3. 景深效果
4. 噪声点阵

## 最后

希望本文能给你创作新特效的灵感，keep creating~