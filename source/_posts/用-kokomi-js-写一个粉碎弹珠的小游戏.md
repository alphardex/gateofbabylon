title: 用 kokomi.js 写一个粉碎弹珠的小游戏
author: alphardex
abbrlink: 17777
date: 2022-04-05 14:14:02
tags:

---

## 前言

大家好，这里是 CSS 兼 WebGL 魔法使——alphardex。本文就让我们来用[kokomi.js](https://github.com/alphardex/kokomi.js)实现一个非常经典的物理游戏——粉碎弹珠（[Smash Hit](https://baike.baidu.com/item/Smash%20Hit/13212932?fr=aladdin)）。

游戏中你将成为一颗弹珠，通过弹射来粉碎面前的一切障碍物，特别适合用来解压。

<!--more-->

## 游戏地址

https://r76xf5.csb.app

## kokomi.js

如果还不认识她，没关系，下面这篇文章会带你初步认识 kokomi.js

https://juejin.cn/post/7078578317053394975

## 场景搭建

首先，我们 fork[这个模板](https://codesandbox.io/s/kokomi-js-starter-tjh29w)来创建一个最简单的场景

创建 4 个场景要素：环境、地面、方块、弹球

每个要素都是一个单独的 class，保持自己的状态和函数，互不干扰

### 环境

里面设定了光照：一个环境光和一个日光

components/environment.ts

```ts
import * as THREE from "three";
import * as kokomi from "kokomi.js";

class Environment extends kokomi.Component {
  ambiLight: THREE.AmbientLight;
  directionalLight: THREE.DirectionalLight;
  constructor(base: kokomi.Base) {
    super(base);

    const ambiLight = new THREE.AmbientLight(0xffffff, 0.7);
    this.ambiLight = ambiLight;

    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.2);
    this.directionalLight = directionalLight;
  }
  addExisting(): void {
    const scene = this.base.scene;
    scene.add(this.ambiLight);
    scene.add(this.directionalLight);
  }
}

export default Environment;
```

### 地面

用来承载一切物理物体的大地

创建一个包含物理属性的 3D 物体分 3 步：

1. 创建渲染世界中的网格`mesh`
2. 创建物理世界中的刚体`body`
3. 将所创建的网格和刚体添加到`kokomi`的`physics`对象中，它会自动地将物理世界的运算状态同步到渲染世界上

注意这里的平面默认是竖着的，我们要将其事先旋转 90 度

`kokomi.convertGeometryToShape`是一个快捷函数，能自动将 three.js 的 geometry 转化为 cannon.js 所需的 shape，不用手动地来定义 shape

components/plane.ts

```ts
import * as THREE from "three";
import * as kokomi from "kokomi.js";
import * as CANNON from "cannon-es";

class Floor extends kokomi.Component {
  mesh: THREE.Mesh;
  body: CANNON.Body;
  constructor(base: kokomi.Base) {
    super(base);

    const geometry = new THREE.PlaneGeometry(10, 10);
    const material = new THREE.MeshStandardMaterial({
      color: new THREE.Color("#777777"),
    });
    const mesh = new THREE.Mesh(geometry, material);
    mesh.rotation.x = THREE.MathUtils.degToRad(-90);
    this.mesh = mesh;

    const shape = kokomi.convertGeometryToShape(geometry);
    const body = new CANNON.Body({
      mass: 0,
      shape,
    });
    body.quaternion.setFromAxisAngle(
      new CANNON.Vec3(1, 0, 0),
      THREE.MathUtils.degToRad(-90)
    );
    this.body = body;
  }
  addExisting(): void {
    const { base, mesh, body } = this;
    const { scene, physics } = base;

    scene.add(mesh);
    physics.add({ mesh, body });
  }
}

export default Floor;
```

### 方块

用来承受弹珠撞击的小方块，也可以换成其他形状

这里的材质用了 kokomi.js 自带的玻璃材质`GlassMaterial`

components/box.ts

```ts
import * as THREE from "three";
import * as kokomi from "kokomi.js";
import * as CANNON from "cannon-es";

class Box extends kokomi.Component {
  mesh: THREE.Mesh;
  body: CANNON.Body;
  constructor(base: kokomi.Base) {
    super(base);

    const geometry = new THREE.BoxGeometry(2, 2, 0.5);
    const material = new kokomi.GlassMaterial({});
    const mesh = new THREE.Mesh(geometry, material);
    this.mesh = mesh;

    const shape = kokomi.convertGeometryToShape(geometry);
    const body = new CANNON.Body({
      mass: 1,
      shape,
      position: new CANNON.Vec3(0, 1, 0),
    });
    this.body = body;
  }
  addExisting(): void {
    const { base, mesh, body } = this;
    const { scene, physics } = base;

    scene.add(mesh);
    physics.add({ mesh, body });
  }
}

export default Box;
```

### 弹珠

我们的主角，它将用来撞碎一切障碍！

components/ball.ts

```ts
import * as THREE from "three";
import * as kokomi from "kokomi.js";
import * as CANNON from "cannon-es";

class Ball extends kokomi.Component {
  mesh: THREE.Mesh;
  body: CANNON.Body;
  constructor(base: kokomi.Base) {
    super(base);

    const geometry = new THREE.SphereGeometry(0.25, 64, 64);
    const material = new kokomi.GlassMaterial({});
    const mesh = new THREE.Mesh(geometry, material);
    this.mesh = mesh;

    const shape = kokomi.convertGeometryToShape(geometry);
    const body = new CANNON.Body({
      mass: 1,
      shape,
      position: new CANNON.Vec3(0, 1, 2),
    });
    this.body = body;
  }
  addExisting(): void {
    const { base, mesh, body } = this;
    const { scene, physics } = base;

    scene.add(mesh);
    physics.add({ mesh, body });
  }
}

export default Ball;
```

### 演员就位

将我们创建的 4 个要素类统统实例化到 Sketch 中，我们就能看到她们了

app.ts

```ts
import * as kokomi from "kokomi.js";
import Floor from "./components/floor";
import Environment from "./components/environment";
import Box from "./components/box";
import Ball from "./components/ball";

class Sketch extends kokomi.Base {
  create() {
    new kokomi.OrbitControls(this);

    this.camera.position.set(-3, 3, 4);

    const environment = new Environment(this);
    environment.addExisting();

    const floor = new Floor(this);
    floor.addExisting();

    const box = new Box(this);
    box.addExisting();

    const ball = new Ball(this);
    ball.addExisting();
  }
}

const createSketch = () => {
  const sketch = new Sketch();
  sketch.create();
  return sketch;
};

export default createSketch;
```

![qOpPXQ.png](https://s2.loli.net/2024/05/22/eL2OWUHT1lvoXhB.png)

### 本阶段地址

https://codesandbox.io/s/1-le69n3?file=/src/app.ts

## 发射弹珠

用户点击鼠标时，能自动从点击的位置发射出一个弹珠

### 创建 Shooter 类

Shooter 类主要负责鼠标点击发射弹珠这个功能

components/shooter.ts

```ts
import * as THREE from "three";
import * as kokomi from "kokomi.js";
import Ball from "./ball";

class Shooter extends kokomi.Component {
  constructor(base: kokomi.Base) {
    super(base);
  }
  addExisting(): void {
    window.addEventListener("click", () => {
      this.shootBall();
    });
  }
  // 发射弹珠
  shootBall() {}
}

export default Shooter;
```

在 Sketch 中实例化 Shooter 类，绑定点击事件

app.ts

```ts
...
import Shooter from "./components/shooter";

class Sketch extends kokomi.Base {
  create() {
    ...
    const shooter = new Shooter(this);
    shooter.addExisting();
  }
}

...
```

### 追踪鼠标

创建弹珠很简单，实例化`Ball`类即可

那如何让它随着鼠标的位置生成呢，可以利用`kokomi`内置的`interactionManager`来获取鼠标的位置

components/shooter.ts

```ts
class Shooter extends kokomi.Component {
  ...
  shootBall() {
    const ball = new Ball(this.base);
    ball.addExisting();

    // 追踪鼠标位置
    const p = new THREE.Vector3(0, 0, 0);
    p.copy(this.base.interactionManager.raycaster.ray.direction);
    p.add(this.base.interactionManager.raycaster.ray.origin);
    ball.body.position.set(p.x, p.y, p.z);
  }
}
```

### 给予速度

这时，弹珠确实会从我们鼠标点击的位置生成了，但都是从原地下落的，我们还要给它们一个沿着鼠标方向的速度

components/shooter.ts

```ts
class Shooter extends kokomi.Component {
  ...
  shootBall() {
    ...

    // 给予鼠标方向的速度
    const v = new THREE.Vector3(0, 0, 0);
    v.copy(this.base.interactionManager.raycaster.ray.direction);
    v.multiplyScalar(24);
    ball.body.velocity.set(v.x, v.y, v.z);
  }
}
```

![qOFrf1.gif](https://s2.loli.net/2024/05/22/3N1X8dybsUrR5eB.gif)

### 发出 shoot 事件

为了能让其他的类获取已经发射的弹珠信息，我们需要在发出弹珠的同时触发`shoot`事件，里面带上发射弹珠的数据

components/shooter.ts

```ts
import mitt, { type Emitter } from "mitt";

class Shooter extends kokomi.Component {
  emitter: Emitter<any>;
  constructor(base: kokomi.Base) {
    ...

    this.emitter = mitt();
  }
  shootBall() {
    ...
    this.emitter.emit("shoot", ball);
  }
}
```

### 本阶段地址

https://codesandbox.io/s/2-ftycgk?file=/src/app.ts

## 击碎物体

### 创建 Breaker 类

Breaker 类主要负责粉碎物理世界的刚体

components/breaker.ts

```ts
import * as THREE from "three";
import * as kokomi from "kokomi.js";
import * as CANNON from "cannon-es";

class Breaker extends kokomi.Component {
  constructor(base: kokomi.Base) {
    super(base);
  }
  // 添加可分割物体
  add(obj: any, splitCount = 0) {
    console.log(obj);
  }
  // 碰撞时
  onCollide(e: any) {
    console.log(e);
  }
}

export default Breaker;
```

在 Sketch 中实例化 Breaker 类，定义好所有可粉碎的物体，监听 Shooter 类的`shoot`事件，当有弹珠射出时给她进行碰撞检测，即监听`collide`事件，将被撞到的物体送给 Breaker 类来粉碎

app.ts

```ts
...
import Breaker from "./components/breaker";

class Sketch extends kokomi.Base {
  create() {
    ...
    const breaker = new Breaker(this);

    // 定义好所有可粉碎的物体
    const breakables = [box];
    breakables.forEach((item) => {
      breaker.add(item);
    });

    // 当弹珠发射时监听碰撞，如果触发碰撞则粉碎撞到的物体
    shooter.emitter.on("shoot", (ball: Ball) => {
      ball.body.addEventListener("collide", (e: any) => {
        breaker.onCollide(e);
      });
    });
  }
}
```

### 粉碎物体

本游戏最有意思的部分来了，如何将物体粉碎成小碎块呢？

这里我们可以用一个 three.js 的 example 里面的类——`ConvexObjectBreaker`。

components/breaker.ts

```ts
class Breaker extends kokomi.Component {
  cob: STDLIB.ConvexObjectBreaker;
  constructor(base: kokomi.Base) {
    ...
    const cob = new STDLIB.ConvexObjectBreaker();
    this.cob = cob;
  }
}
```

#### 添加可粉碎的物体

调用`ConvexObjectBreaker`的`prepareBreakableObject`为 mesh 创建分割数据，再给`body`绑上`mesh`的 id，使其能相互对应

components/breaker.ts

```ts
class Breaker extends kokomi.Component {
  ...
  objs: any[];
  // 添加可粉碎物体
  add(obj: any, splitCount = 0) {
    this.cob.prepareBreakableObject(
      obj.mesh,
      obj.body.mass,
      obj.body.velocity,
      obj.body.angularVelocity,
      true
    );
    obj.body.userData = {
      splitCount, // 已经被分割的次数
      meshId: obj.mesh.id, // 使body能对应上其相应的mesh
    };
    this.objs.push(obj);
  }
}
```

#### 碰撞检测

通过碰撞物的 meshId 获取其对应的对象 obj，如果她还没被分割过，就对她进行分割

这里判断她最多只能被分割 1 次，超过就不会再分割（如果对自己 cpu 有自信的可以调高一点:d）

components/breaker.ts

```ts
class Breaker extends kokomi.Component {
  ...
  // 通过id获取obj
  getObjById(id: any) {
    const obj = this.objs.find((item: any) => item.mesh.id === id);
    return obj;
  }
  // 碰撞时
  onCollide(e: any) {
    const obj = this.getObjById(e.body.userData?.meshId);
    if (obj && obj.body.userData.splitCount < 2) {
      this.splitObj(e);
    }
  }
  // 分割物体
  splitObj(e: any) {
    console.log(obj);
  }
}
```

#### 分割物体

本游戏最复杂的一个函数，思路是这样的：

1. 获取 3 个分割所需的数据：网格`mesh`（已有）、碰撞点`poi`（需计算出来）、法线`nor`（也需计算出来）
2. 将分割所需数据传给函数`subdivideByImpact`，获取所有碎片的`mesh`
3. 移除已经破碎的物体对象（渲染世界和物理世界都要移除）
4. 将所有碎片添加至当前的世界（`mesh`已有，`body`的`shape`可以通过快捷函数计算出来，`mass`直接取`userData`里的即可，同步`position`和`quaternion`数据）

最关键的点是碰撞点的计算，`e.contact`是接触方程，她有几个参数官方文档上也没写含义，但通过笔者的理解如下：

1. bi：第一个碰撞体（也就是 body i）
2. bj：第二个碰撞体（也就是 body j）
3. ri：从第一个碰撞体到接触点的世界向量
4. rj：从第二个碰撞体到接触点的世界向量
5. ni：法向量

了解了她们的含义后，我们就能算出碰撞点`poi`和法线`nor`

components/breaker.ts

```ts
class Breaker extends kokomi.Component {
  ...
  // 分割物体
  splitObj(e: any) {
    const obj = this.getObjById(e.body.userData?.meshId); // 碰撞物
    const mesh = obj.mesh; // 网格
    const body = e.body as CANNON.Body;
    const contact = e.contact as CANNON.ContactEquation; // 接触
    const poi = body.pointToLocalFrame(contact.bj.position).vadd(contact.rj); // 碰撞点
    const nor = new THREE.Vector3(
      contact.ni.x,
      contact.ni.y,
      contact.ni.z
    ).negate(); // 法线
    const fragments = this.cob.subdivideByImpact(
      mesh,
      new THREE.Vector3(poi.x, poi.y, poi.z),
      nor,
      1,
      1.5
    ); // 将网格分割成碎片

    // 移除已经破碎的物体
    this.base.scene.remove(mesh);
    setTimeout(() => {
      this.base.physics.world.removeBody(body);
    });

    // 将碎片添加至当前的世界
    fragments.forEach((mesh: THREE.Object3D) => {
      // 将mesh转化为物理世界的shape
      const geometry = (mesh as THREE.Mesh).geometry;
      const shape = kokomi.convertGeometryToShape(geometry);
      const body = new CANNON.Body({
        mass: mesh.userData.mass,
        shape,
        position: new CANNON.Vec3(
          mesh.position.x,
          mesh.position.y,
          mesh.position.z
        ), // 这里别忘了同步碎片的位置，不然碎片会飞出去
        quaternion: new CANNON.Quaternion(
          mesh.quaternion.x,
          mesh.quaternion.y,
          mesh.quaternion.z,
          mesh.quaternion.w
        ), // 旋转方向也要同步
      });
      this.base.scene.add(mesh);
      this.base.physics.add({ mesh, body });

      // 将碎片添加至可破坏物中
      const obj = {
        mesh,
        body,
      };
      this.add(obj, e.body.userData.splitCount + 1);
    });
  }
}
```

![qjaWfH.gif](https://s2.loli.net/2024/05/22/TLhU23nfCjAclN1.gif)

可以看到方块被瞬间击碎成无数个小碎片，简直完美！

#### 击碎事件

击碎时同样需要触发一个事件，就叫`hit`吧

components/breaker.ts

```ts
import mitt, { type Emitter } from "mitt";

class Breaker extends kokomi.Component {
  ...
  emitter: Emitter<any>;
  constructor(base: kokomi.Base) {
    ...
    this.emitter = mitt();
  }
  // 碰撞时
  onCollide(e: any) {
    ...
    if (obj && obj.body.userData.splitCount < 2) {
      ...
      this.emitter.emit("hit");
    }
  }
}
```

#### 添加击碎音效

没有破碎的声音就失去了灵魂，用`kokomi`自带的`AssetManager`加载好音效素材（来源：[爱给网](https://www.aigei.com/s?q=%E7%8E%BB%E7%92%83&type=sound)），击中时播放即可

resources.ts

```ts
import type * as kokomi from "kokomi.js";

import glassBreakAudio from "./assets/audios/glass-break.mp3";

const resourceList: kokomi.ResourceItem[] = [
  {
    name: "glassBreakAudio",
    type: "audio",
    path: glassBreakAudio,
  },
];

export default resourceList;
```

app.ts

```ts
import resourceList from "./resources";

class Sketch extends kokomi.Base {
  create() {
    const assetManager = new kokomi.AssetManager(this, resourceList);
    assetManager.emitter.on("ready", () => {
      const listener = new THREE.AudioListener();
      this.camera.add(listener);

      const glassBreakAudio = new THREE.Audio(listener);
      glassBreakAudio.setBuffer(assetManager.items.glassBreakAudio);

      ...

      // 弹珠击中物体时
      breaker.emitter.on("hit", () => {
        if (!glassBreakAudio.isPlaying) {
          glassBreakAudio.play();
        } else {
          glassBreakAudio.stop();
          glassBreakAudio.play();
        }
      });
    });
  }
}
```

### 本阶段地址

https://codesandbox.io/s/3-qf11ul?file=/src/app.ts

## 游戏模式

物理模拟阶段告一段落，现在让我们开始游戏的玩法吧。

游戏的玩法其实各有千秋：闯关式、无尽模式、限时式、禅模式等等，本文就采用最简单的禅模式。

### 禅模式

其实禅模式也属于无尽模式，只不过她不会考虑任何得失，没有成功和失败的概念，如果是纯粹为了放松，强烈推荐这种模式

实现思路也很简单：你不动她动

components/game.ts

```ts
import * as THREE from "three";
import * as kokomi from "kokomi.js";
import * as CANNON from "cannon-es";
import Box from "./box";
import mitt, { type Emitter } from "mitt";

class Game extends kokomi.Component {
  breakables: any[];
  emitter: Emitter<any>;
  score: number;
  constructor(base: kokomi.Base) {
    super(base);

    this.breakables = [];

    this.emitter = mitt();

    this.score = 0;
  }
  // 创建可破碎物
  createBreakables(x = 0) {
    const box = new Box(this.base);
    box.addExisting();
    const position = new CANNON.Vec3(x, 1, -10);
    box.body.position.copy(position);
    this.breakables.push(box);
    this.emitter.emit("create", box);
  }
  // 移动可破碎物
  moveBreakables(objs: any) {
    objs.forEach((item: any) => {
      item.body.position.z += 0.1;

      // 超过边界则移除
      if (item.body.position.z > 10) {
        this.base.scene.remove(item.mesh);
        this.base.physics.world.removeBody(item.body);
      }
    });
  }
  // 定时创建可破碎物
  createBreakablesByInterval() {
    this.createBreakables();
    setInterval(() => {
      const x = THREE.MathUtils.randFloat(-3, 3);
      this.createBreakables(x);
    }, 3000);
  }
  // 加分
  incScore() {
    this.score += 1;
  }
}

export default Game;
```

地面更长一些

components/floor.ts

```ts
class Floor extends kokomi.Component {
  ...
  constructor(base: kokomi.Base) {
    ...

    const geometry = new THREE.PlaneGeometry(10, 50);
    ...
  }
  ...
}
```

将游戏应用到 Sketch 上

app.ts

```ts
import * as kokomi from "kokomi.js";
import * as THREE from "three";
import Floor from "./components/floor";
import Environment from "./components/environment";
import Shooter from "./components/shooter";
import Breaker from "./components/breaker";
import resourceList from "./resources";
import type Ball from "./components/ball";
import Game from "./components/game";

class Sketch extends kokomi.Base {
  create() {
    const assetManager = new kokomi.AssetManager(this, resourceList);
    assetManager.emitter.on("ready", () => {
      const listener = new THREE.AudioListener();
      this.camera.add(listener);

      const glassBreakAudio = new THREE.Audio(listener);
      glassBreakAudio.setBuffer(assetManager.items.glassBreakAudio);

      this.camera.position.set(0, 1, 6);

      const environment = new Environment(this);
      environment.addExisting();

      const floor = new Floor(this);
      floor.addExisting();

      const shooter = new Shooter(this);
      shooter.addExisting();

      const breaker = new Breaker(this);

      const game = new Game(this);

      game.emitter.on("create", (obj: any) => {
        breaker.add(obj);
      });

      game.createBreakablesByInterval();

      // 当弹珠发射时监听碰撞，如果触发碰撞则粉碎撞到的物体
      shooter.emitter.on("shoot", (ball: Ball) => {
        ball.body.addEventListener("collide", (e: any) => {
          breaker.onCollide(e);
        });
      });

      // 弹珠击中物体时
      breaker.emitter.on("hit", () => {
        game.incScore();
        document.querySelector(".score")!.textContent = `${game.score}`;
        glassBreakAudio.play();
      });

      this.update(() => {
        game.moveBreakables(breaker.objs);
      });
    });
  }
}

const createSketch = () => {
  const sketch = new Sketch();
  sketch.create();
  return sketch;
};

export default createSketch;
```

![qjjNff.gif](https://s2.loli.net/2024/05/22/uUAkOGmIWsVpRdB.gif)

### 本阶段地址

https://codesandbox.io/s/4-xz30nb?file=/src/app.ts

## 优化

这里可以自由发挥，比如可以有以下几点：

1. 添加更多的可破坏图形，比如锥体
2. 改变各种颜色（背景色、材质颜色、光照颜色），优化渲染
3. 给远景添加雾化效果
4. 自己创作地图，将游戏变成闯关式

个人美化后的结果如下

![qvovq0.gif](https://s2.loli.net/2024/05/22/fHqyZisGuBeCwTP.gif)

### 本阶段地址

https://codesandbox.io/s/5-r76xf5?file=/src/app.ts

## 最后

这个游戏成品很粗糙，有很大的提升空间。

尽管如此，创作本身的过程才是最有意义的，只有亲自经历过才会体会其中的美。
