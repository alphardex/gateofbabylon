title: three.js实现反应堆游戏
author: alphardex
abbrlink: 8477
tags: []
categories: []
date: 2020-12-12 09:58:00
---
## 前言

大家好，这里是CSS魔法使——alphardex。

之前在appstore上有这样一个游戏，叫stack（中文译名为“反应堆”），游戏规则是这样的：在竖直方向上会不停地有方块出现并来回移动，点击屏幕能叠方块，而你的目的是尽量使它们保持重合，不重合就会被削掉，叠得越多分数越高。玩法虽简单但极其令人上瘾。

碰巧笔者最近在学习three.js——一个基于webgl的3d框架，于是乎就思索着能不能用three.js来实现这样的效果，以摸清那些3D游戏的套路。

<!--more-->

最终的效果图如下

![](https://i.loli.net/2020/12/12/hqIk8yf9GY763ol.gif)

## 技术栈

- [three.js](https://github.com/mrdoob/three.js)：本文主角
- [gsap](https://github.com/greensock/GSAP)：使数值动起来的工具人
- [kyouka](https://github.com/alphardex/kyouka)：我的TS工具库，名字来源于公主连结的冰川镜华（xcw）
- [aqua.css](https://github.com/alphardex/aqua.css)：我的CSS框架，名字来源于素晴的智慧女神阿库娅

## 规则解析

- 每一关创建一个方块，并使其在x轴或者z轴上来回移动，方块的高度和速度是递增的
- 点击时进行重叠的判定，将不重叠的部分削掉，重叠的部分固定在原先的位置，完全不重叠则游戏结束
- 方块的颜色随关卡数的增加进行有规律的变换

撒，哈吉马路油！

## 基础场景

首先先创建一个最简单的场景，也是three.js里的hello world

```ts
interface Cube {
  width?: number;
  height?: number;
  depth?: number;
  x?: number;
  y?: number;
  z?: number;
  color?: string | Color;
}

const calcAspect = (el: HTMLElement) => el.clientWidth / el.clientHeight;

class Base {
  debug: boolean;
  container: HTMLElement | null;
  scene!: Scene;
  camera!: PerspectiveCamera | OrthographicCamera;
  renderer!: WebGLRenderer;
  box!: Mesh;
  light!: PointLight | DirectionalLight;
  constructor(sel: string, debug = false) {
    this.debug = debug;
    this.container = document.querySelector(sel);
  }
  // 初始化
  init() {
    this.createScene();
    this.createCamera();
    this.createRenderer();
    const box = this.createBox({});
    this.box = box;
    this.createLight();
    this.addListeners();
    this.setLoop();
  }
  // 创建场景
  createScene() {
    const scene = new Scene();
    if (this.debug) {
      scene.add(new AxesHelper());
    }
    this.scene = scene;
  }
  // 创建透视相机
  createCamera() {
    const aspect = calcAspect(this.container!);
    const camera = new PerspectiveCamera(75, aspect, 0.1, 100);
    camera.position.set(0, 1, 10);
    this.camera = camera;
  }
  // 创建渲染
  createRenderer() {
    const renderer = new WebGLRenderer({
      alpha: true,
      antialias: true,
    });
    renderer.setSize(this.container!.clientWidth, this.container!.clientHeight);
    this.container?.appendChild(renderer.domElement);
    this.renderer = renderer;
    this.renderer.setClearColor(0x000000, 0);
  }
  // 创建方块
  createBox(cube: Cube) {
    const { width = 1, height = 1, depth = 1, color = new Color("#d9dfc8"), x = 0, y = 0, z = 0 } = cube;
    const geo = new BoxBufferGeometry(width, height, depth);
    const material = new MeshToonMaterial({ color, flatShading: true });
    const box = new Mesh(geo, material);
    box.position.x = x;
    box.position.y = y;
    box.position.z = z;
    this.scene.add(box);
    return box;
  }
  // 创建光源
  createLight() {
    const light = new DirectionalLight(new Color("#ffffff"), 0.5);
    light.position.set(0, 50, 0);
    this.scene.add(light);
    const ambientLight = new AmbientLight(new Color("#ffffff"), 0.4);
    this.scene.add(ambientLight);
    this.light = light;
  }
  // 监听事件
  addListeners() {
    this.onResize();
  }
  // 监听画面缩放
  onResize() {
    window.addEventListener("resize", (e) => {
      const aspect = calcAspect(this.container!);
      const camera = this.camera as PerspectiveCamera;
      camera.aspect = aspect;
      camera.updateProjectionMatrix();
      this.renderer.setSize(this.container!.clientWidth, this.container!.clientHeight);
    });
  }
  // 动画
  update() {
    console.log("animation");
  }
  // 渲染
  setLoop() {
    this.renderer.setAnimationLoop(() => {
      this.update();
      this.renderer.render(this.scene, this.camera);
    });
  }
}
```
这个场景把three.js最基本的要素都囊括在内了：场景、相机、渲染、物体、光源、事件、动画。效果图如下：

![](https://i.loli.net/2020/12/12/CZuzv4aAOw1bkHF.png)

## 游戏场景

### 初始化

首先，将本游戏所必要的参数全部设定好。

相机采用了正交相机（无论物体远近，大小始终不变）

创建了一个底座，高度设定为大约场景高的二分之一

```ts
class Stack extends Base {
  cameraParams: Record<string, any>; // 相机参数
  cameraPosition: Vector3; // 相机位置
  lookAtPosition: Vector3; // 视点
  colorOffset: number; // 颜色偏移量
  boxParams: Record<string, any>; // 方块属性参数
  level: number; // 关卡
  moveLimit: number; // 移动上限
  moveAxis: "x" | "z"; // 移动所沿的轴
  moveEdge: "width" | "depth"; // 移动的边
  currentY: number; // 当前的y轴高度
  state: string; // 状态：paused - 静止；running - 运动
  speed: number; // 移动速度
  speedInc: number; // 速度增量
  speedLimit: number; // 速度上限
  gamestart: boolean; // 游戏开始
  gameover: boolean; // 游戏结束
  constructor(sel: string, debug: boolean) {
    super(sel, debug);
    this.cameraParams = {};
    this.updateCameraParams();
    this.cameraPosition = new Vector3(2, 2, 2);
    this.lookAtPosition = new Vector3(0, 0, 0);
    this.colorOffset = ky.randomIntegerInRange(0, 255);
    this.boxParams = { width: 1, height: 0.2, depth: 1, x: 0, y: 0, z: 0, color: new Color("#d9dfc8") };
    this.level = 0;
    this.moveLimit = 1.2;
    this.moveAxis = "x";
    this.moveEdge = "width";
    this.currentY = 0;
    this.state = "paused";
    this.speed = 0.02;
    this.speedInc = 0.0005;
    this.speedLimit = 0.05;
    this.gamestart = false;
    this.gameover = false;
  }
  // 更新相机参数
  updateCameraParams() {
    const { container } = this;
    const aspect = calcAspect(container!);
    const zoom = 2;
    this.cameraParams = { left: -zoom * aspect, right: zoom * aspect, top: zoom, bottom: -zoom, near: -100, far: 1000 };
  }
  // 创建正交相机
  createCamera() {
    const { cameraParams, cameraPosition, lookAtPosition } = this;
    const { left, right, top, bottom, near, far } = cameraParams;
    const camera = new OrthographicCamera(left, right, top, bottom, near, far);
    camera.position.set(cameraPosition.x, cameraPosition.y, cameraPosition.z);
    camera.lookAt(lookAtPosition.x, lookAtPosition.y, lookAtPosition.z);
    this.camera = camera;
  }
  // 初始化
  init() {
    this.createScene();
    this.createCamera();
    this.createRenderer();
    this.updateColor(); // 这行先注释掉，下节再用
    const baseParams = { ...this.boxParams };
    const baseHeight = 2.5;
    baseParams.height = baseHeight;
    baseParams.y -= (baseHeight - this.boxParams.height) / 2;
    const base = this.createBox(baseParams);
    this.box = base;
    this.createLight();
    this.addListeners();
    this.setLoop();
  }
}
```

![](https://i.loli.net/2020/12/12/O3fwSkLpoyIn9Bl.png)

### 有规律地改变方块颜色

这里采用了正弦函数来周期性地改变方块的颜色

```ts
class Stack extends Base {
  ...
  // 更新颜色
  updateColor() {
    const { level, colorOffset } = this;
    const colorValue = (level + colorOffset) * 0.25;
    const r = (Math.sin(colorValue) * 55 + 200) / 255;
    const g = (Math.sin(colorValue + 2) * 55 + 200) / 255;
    const b = (Math.sin(colorValue + 4) * 55 + 200) / 255;
    this.boxParams.color = new Color(r, g, b);
  }
}
```

![](https://i.loli.net/2020/12/12/b5hlzSRy4UuVYkW.png)

### 创建并移动方块

每开始一个关卡，我们就要做以下的事情：

- 确定方块是在x轴还是z轴上移动
- 增加方块的高度和移动速度
- 更新方块颜色
- 创建方块
- 根据移动轴来确定方块的初始移动位置
- 更新相机和视角的高度
- 开始移动方块，当移动到最大距离时反转速度，形成来回移动的效果
- 用户点击时进行重叠判定

```ts
class Stack extends Base {
  ...
  // 开始游戏
  start() {
    this.gamestart = true;
    this.startNextLevel();
  }
  // 开始下一关
  startNextLevel() {
    this.level += 1;
    // 确定移动轴和移动边：奇数x；偶数z
    this.moveAxis = this.level % 2 ? "x" : "z";
    this.moveEdge = this.level % 2 ? "width" : "depth";
    // 增加方块生成的高度
    this.currentY += this.boxParams.height;
    // 增加方块的速度
    if (this.speed <= this.speedLimit) {
      this.speed += this.speedInc;
    }
    this.updateColor();
    const boxParams = { ...this.boxParams };
    boxParams.y = this.currentY;
    const box = this.createBox(boxParams);
    this.box = box;
    // 确定初始移动位置
    this.box.position[this.moveAxis] = this.moveLimit * -1;
    this.state = "running";
    if (this.level > 1) {
      this.updateCameraHeight();
    }
  }
  // 更新相机高度
  updateCameraHeight() {
    this.cameraPosition.y += this.boxParams.height;
    this.lookAtPosition.y += this.boxParams.height;
    gsap.to(this.camera.position, {
      y: this.cameraPosition.y,
      duration: 0.4,
    });
    gsap.to(this.camera.lookAt, {
      y: this.lookAtPosition.y,
      duration: 0.4,
    });
  }
  // 动画
  update() {
    if (this.state === "running") {
      const { moveAxis } = this;
      this.box.position[moveAxis] += this.speed;
      // 移到末端就反转方向
      if (Math.abs(this.box.position[moveAxis]) > this.moveLimit) {
        this.speed = this.speed * -1;
      }
    }
  }
  // 事件监听
  addListeners() {
    if (this.debug) {
      this.onKeyDown();  // 这行先注释掉，下节再用
    } else {
      this.onClick();
    }
  }
  // 监听点击
  onClick() {
    this.renderer.domElement.addEventListener("click", () => {
      if (this.level === 0) {
        this.start();
      } else {
        this.detectOverlap(); // 这行先注释掉，最后一节再用
      }
    });
  }
}
```
![](https://i.loli.net/2020/12/12/jl1SaeEOQkzmrxN.gif)

### 调试模式

由于某些数值的计算对本游戏来说很是关键，因此我们要弄一个调试模式，在这个模式下，我们能通过键盘来暂停方块的运动，并动态改变方块的位置，配合[three.js扩展程序](https://chrome.google.com/webstore/detail/threejs-developer-tools/ebpnegggocnnhleeicgljbedjkganaek?hl=zh-CN)来调试各个数值

```ts
class Stack extends Base {
  ...
  // 监听键盘（调试时使用：空格下一关；P键暂停；上下键控制移动）
  onKeyDown() {
    document.addEventListener("keydown", (e) => {
      const code = e.code;
      if (code === "KeyP") {
        this.state = this.state === "running" ? "paused" : "running";
      } else if (code === "Space") {
        if (this.level === 0) {
          this.start();
        } else {
          this.detectOverlap(); // 这行先注释掉，最后一节再用
        }
      } else if (code === "ArrowUp") {
        this.box.position[this.moveAxis] += this.speed / 2;
      } else if (code === "ArrowDown") {
        this.box.position[this.moveAxis] -= this.speed / 2;
      }
    });
  }
}
```

### 检测重叠部分

本游戏最难的部分来了，笔者调了很久才成功，一句话：耐心就是胜利。

方块切下来的效果是怎么实现的呢？其实这是一个障眼法：方块本身并没有被“切开”，而是在同一个位置创建了2个方块：一个就是重叠的方块，另一个就是不重叠的方块，即被“切开”的那个方块。

尽管我们现在知道了要创建这两个方块，但确定它俩的参数可绝非易事，建议拿一个草稿纸将方块移动的位置画下来，再动手计算那几个数值（想起我可怜的数学水平），如果实在是算不来，就直接CV笔者的公式吧:)

计算完后，一切就豁然开朗了，将那两个方块创建出来，并用gsap将不重叠的那个方块落下，本游戏就算正式完成了

```ts
class Stack extends Base {
  ...
  // 检测重叠部分
  // 难点：1. 重叠距离计算 2. 重叠方块位置计算 3. 切掉方块位置计算
  async detectOverlap() {
    const that = this;
    const { boxParams, moveEdge, box, moveAxis, currentY, camera } = this;
    const currentPosition = box.position[moveAxis];
    const prevPosition = boxParams[moveAxis];
    const direction = Math.sign(currentPosition - prevPosition);
    const edge = boxParams![moveEdge];
    // 重叠距离 = 上一个方块的边长 + 方向 * (上一个方块位置 - 当前方块位置)
    const overlap = edge + direction * (prevPosition - currentPosition);
    if (overlap <= 0) {
      this.state = "paused";
      this.dropBox(box);
      gsap.to(camera, {
        zoom: 0.6,
        duration: 1,
        ease: "Power1.easeOut",
        onUpdate() {
          camera.updateProjectionMatrix();
        },
        onComplete() {
          const score = that.level - 1;
          const prevHighScore = Number(localStorage.getItem('high-score')) || 0;
          if (score > prevHighScore) {
            localStorage.setItem('high-score', `${score}`)
          }
          that.gameover = true;
        },
      });
    } else {
      // 创建重叠部分的方块
      const overlapBoxParams = { ...boxParams };
      const overlapBoxPosition = currentPosition / 2 + prevPosition / 2;
      overlapBoxParams.y = currentY;
      overlapBoxParams[moveEdge] = overlap;
      overlapBoxParams[moveAxis] = overlapBoxPosition;
      this.createBox(overlapBoxParams);
      // 创建切掉部分的方块
      const slicedBoxParams = { ...boxParams };
      const slicedBoxEdge = edge - overlap;
      const slicedBoxPosition = direction * ((edge - overlap) / 2 + edge / 2 + direction * prevPosition);
      slicedBoxParams.y = currentY;
      slicedBoxParams[moveEdge] = slicedBoxEdge;
      slicedBoxParams[moveAxis] = slicedBoxPosition;
      const slicedBox = this.createBox(slicedBoxParams);
      this.dropBox(slicedBox);
      this.boxParams = overlapBoxParams;
      this.scene.remove(box);
      this.startNextLevel();
    }
  }
  // 使方块旋转下落
  dropBox(box: Mesh) {
    const { moveAxis } = this;
    const that = this;
    gsap.to(box.position, {
      y: "-=3.2",
      ease: "power1.easeIn",
      duration: 1.5,
      onComplete() {
        that.scene.remove(box);
      },
    });
    gsap.to(box.rotation, {
      delay: 0.1,
      x: moveAxis === "z" ? ky.randomNumberInRange(4, 5) : 0.1,
      y: 0.1,
      z: moveAxis === "x" ? ky.randomNumberInRange(4, 5) : 0.1,
      duration: 1.5,
    });
  }
}
```

![](https://i.loli.net/2020/12/12/hqIk8yf9GY763ol.gif)

## 在线游玩地址

[猛戳这里](https://codepen.io/alphardex/pen/YzGprbM)