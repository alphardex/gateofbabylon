abbrlink: 19835
date: 2020-10-26 11:18:51
---
## 前言

大家好，这里是CSS魔法使——alphardex。

“密室逃脱”这个词想必大家并不陌生，在以前的flash时代，这是一类很经典的益智游戏之一。玩家常常会被困在一间密室中，而过关的目的就是想法设法逃出这件密室。以下是笔者玩的最早的一个密室逃脱游戏——深红房间，它也可以说是密室逃脱类游戏的先祖。

![](https://i.loli.net/2020/10/26/hxYi8IbmvDRwKHa.jpg)

接下来，笔者要用纯CSS实现一款类似的密室逃脱类游戏。

是的，你没听错，纯CSS，也就意味着完全没有JS的参与。有人就纳闷了：WTF?CSS，一个网页布局的语言，居然还能写游戏？可惜的是，CSS还真能写游戏。接下来随笔者一起进入这个不思议的国度吧。

## 攻略

每次笔者玩密室逃脱游戏卡关时，总会去搜搜攻略，看完后就能把游戏玩通。因此当我们做密室逃脱类游戏时，首先要考虑的事情就是攻略。以下是笔者为本文密室逃脱游戏所制定的攻略

```
1. 左转，转动地球仪
2. 右转，发现一根锤子，点击捡起，记住墙上的数字
3. 左转，点击柜子，用锤子砸开它，获得一个圆盘
4. 点击墙上的壁画，壁画移开，看到一圆盘印，嵌入圆盘，获得一个usb
5. 右转2次，将usb插入电脑，电脑开启，输入墙上的密码，获得钥匙
6. 右转，用钥匙打开大门，游戏结束
```

## 开关

制定完攻略后，就要开始确定该游戏的核心所在——开关。说到开关，大家觉得HTML里的哪个元素最适合用来做开关？答案是单复选框。

说起单复选框，就不得不提这2个CP——`label`和兄弟选择符。`label`负责将该元素与其对应的复选框用`for`来关联起来，而兄弟选择符则负责与`:checked`伪类配合好，当某元素被勾选时，其相邻的元素就会受到它的影响。

首先，让我们来看一看一个简单的开关例子

```html
<input type="radio" id="globe" class="globe-trigger" />
<input type="radio" id="hammer" class="hammer-trigger" />
<label for="globe" class="globe">
  <img src="https://i.loli.net/2020/10/25/YBnOQ2jVtSTmFkE.png" alt class="w-8" />
</label>
<label for="hammer" class="hammer">
  <img src="https://i.loli.net/2020/10/25/KhVp4EaMoYrjlIC.png" alt class="w-6" />
</label>
```

```scss
.hammer {
  display: none;
}

.globe-trigger:checked {
  & ~ {
    .globe {
      pointer-events: none;
    }

    .hammer {
      display: inline-block;
    }
  }
}

.hammer-trigger:checked {
  & ~ {
    .hammer {
      transform: scale(0);
      opacity: 0;
    }
  }
}
```

![](https://i.loli.net/2020/10/26/5eB4axnul7SovtK.gif)

可以看到我们用`label`元素包裹了对应的图片，并关联好了对应的开关。当用户点击地球仪`globe`时，`globe-trigger`开关就会被触发，这就是`label`的关联性

触发开关后，开关旁边对应的元素状态就发生了变化：`globe`变得无法被点击；`hammer`元素出现，这就是兄弟选择符的作用

同理，点击锤子`hammer`时，与其关联的`hammer-trigger`开关被触发，与此同时旁边的`hammer`就会消失，代表被用户“捡起”这一动作

理解开关的原理后，我们就可以把开关给隐藏起来啦

```scss
input[type="checkbox"],
input[type="radio"] {
  display: none;
}
```

## 场景切换

假设我们游戏地图分为4块，且可以用导航箭头来切换。

游戏的地图其实是一张长图，如下图所示

![](https://i.loli.net/2020/10/26/lhypFBrKeZaSxHu.png)

```html
<div class="camera">
  <!-- 导航 -->
  <input type="radio" id="nav-1" name="nav" class="nav-trigger-1" />
  <input type="radio" id="nav-2" name="nav" class="nav-trigger-2" />
  <input type="radio" id="nav-3" name="nav" class="nav-trigger-3" />
  <input type="radio" id="nav-4" name="nav" class="nav-trigger-4" />
  <!-- 长图 -->
  <form class="stage">
    <!-- 开关 -->
    <input type="checkbox" id="globe" class="globe-trigger" />...
    <!-- 场景 -->
    <div class="scene scene-1">
      <label for="...">...</label>
      <nav class="navs">
        <label for="nav-4" class="nav-left"></label>
        <label for="nav-2" class="nav-right"></label>
      </nav>
    </div>
  </form>
</div>
```

首先，设定游戏的固定视角，将多余的部分裁掉

```scss
.camera {
  --stage-width: 18rem;
  --scene-id: 0;

  position: relative;
  width: var(--stage-width);
  height: var(--stage-width);
  overflow: hidden;
}
```

然后，设定导航，根据所选的导航来确定长图的平移距离

```scss
@for $i from 1 through 4 {
  .nav-trigger-#{$i}:checked {
    & ~ .stage {
      --scene-id: #{$i - 1};
    }
  }
}

.stage {
  transform: translateY(calc(var(--stage-width) * var(--scene-id) * -1));
}

.scene {
  position: relative;
  width: var(--stage-width);
  height: var(--stage-width);
}
```

比如在场景1，用户向右走，导航2被触发，长图将上平移一个单位，如下图所示

![](https://i.loli.net/2020/10/26/xtp5gihWOHvKj1F.png)

这样就完成了场景切换这一效果

## 完成项目

此刻，我们已经具备完成密室逃脱游戏所必须的知识了。根据上面的攻略，一步步定制好所有开关，摆放好所有物件，且能确保场景能自由切换，这样一个纯CSS密室逃脱游戏就成功诞生啦

在线游玩地址：[Escape Room Game](https://codepen.io/alphardex/full/GRqWRyB)

![](https://i.loli.net/2020/10/26/FmuoR9zbxdp4s7X.gif)