title: 将 Shadertoy 上的特效移植为桌面壁纸
author: alphardex
abbrlink: 11232
date: 2022-06-19 11:23:40
tags:
---
## 前言

大家好，这里是 CSS 兼 WebGL 魔法使——alphardex。本文就让我们来用[kokomi.js](https://github.com/alphardex/kokomi.js)的[shadertoy组件](https://github.com/alphardex/kokomi.js#shader-toy-tag)将Shadertoy上的炫酷特效移植到我们的桌面上~

![aurora.gif](https://s2.loli.net/2022/06/19/gR7NwvcCIs6Ezfn.gif)

<!--more-->

## 缘起

[Shadertoy](https://www.shadertoy.com)是笔者经常逛的一个网站，上面有许多CG级别的炫酷特效，有一天，笔者突然萌生了一个想法：能否将这些特效移植到桌面上，作为动态壁纸来欣赏。

恰好前阵子笔者正在进行[kokomi.js](https://juejin.cn/post/7078578317053394975)（一个Web 3D的辅助框架）的开发，里面有一个[ScreenQuad](https://github.com/alphardex/kokomi.js#screenquad)的组件，能渲染着色器代码的结果，再借助[自定义组件](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components/Using_custom_elements)这一特性，笔者封装了一个[shadertoy的组件](https://github.com/alphardex/kokomi.js#shader-toy-tag)，只要将片元着色器的代码复制进去，就能将着色器本身的代码给渲染出来，并且跟Shadertoy网站本身带来的效果几乎别无二致。

## 准备特效

本地新建一个html文件，引入kokomi.js，注册好shadertoy组件，清除页面的空白部分

```html
<style>
    body {
        margin: 0
    }
</style>
<script src="https://unpkg.com/kokomi.js@1.9.21/build/kokomi.umd.js"></script>
<script>
    kokomi.ShaderToyElement.register();
</script>
```

接下来我们可以使用`<shader-toy>`这个标签了（放在`<style>`标签下面）

```html
<shader-toy></shader-toy>
```

你会发现屏幕上出现了一个颜色变换的场景，其实这就是[shadertoy官方的模板代码](https://www.shadertoy.com/new)

接下来我们可以开始往里面填充着色器特效了，以这个[极光特效](https://www.shadertoy.com/view/XtGGRt)为例，将右侧的着色器代码CV至`<shader-toy>`标签内的`<script type='frag'>`标签内即可

当然这里给个友善的小提醒：CV完后记得标记出处，表示对原作者的尊重

```html
<shader-toy>
    <!-- Credit: https://www.shadertoy.com/view/XtGGRt -->
    <script type="frag">
      (将着色器代码CV于此)
    </script>
</shader-toy>
```

这样我们就能看见特效成功地被渲染出来了

https://code.juejin.cn/pen/7110784867540926478

PS：有的特效本身还带有材质，如果有的话就要添加img标签，将name设定为对应的材质名（如iChannel0），加上hidden隐藏起来即可

```html
<img src="./iChannel0.png" name="iChannel0" hidden />
```

如果想要更多的特效，可以移步[这个仓库](https://github.com/alphardex/shadertoy-wallpaper)

## 移植桌面壁纸

笔者将使用[Wallpaper Engine](https://store.steampowered.com/app/431960/Wallpaper_Engine/)来完成接下来的操作，目前windows系统只有这个才能达到动态桌面的效果，暂时找不到别的替代品（如果有可以直接在下面回复），mac或linux系统请自行寻找替代的解决方案（只要支持能用html作为桌面背景的软件就行）

打开WE，点击左下角的“壁纸编辑器”

![1.png](https://s2.loli.net/2022/06/19/XCjrnmgIGJKyhkF.png)

这时会跳出一个“欢迎”的弹窗，点击“创建壁纸”

![2.png](https://s2.loli.net/2022/06/19/E3pf5vYcH6rxwWk.png)

选择你之前准备好的特效的`html`文件，再点击“确认”

![3.png](https://s2.loli.net/2022/06/19/cA3kxQGeXLSY6Ed.png)

点击左上角的“文件”，再点击“应用壁纸”，完成！

![4.png](https://s2.loli.net/2022/06/19/bqdse6DWkSnzfxy.png)

最后的效果如下，还能鼠标交互哦

![](https://s3.bmp.ovh/imgs/2022/06/19/96a48e9fda2f7811.gif)

## 原理

想了解shadertoy组件原理的可以直接移步[组件的源码](https://github.com/alphardex/kokomi.js/blob/main/src/web/shadertoy.ts)，本质上是做了这些事情：

1. 申明一个静态的注册方法，里面负责注册组件本身到页面上
2. 选择组件内部的`<script type='frag'>`元素，获取其内容（着色器代码），用它来创建ScreenQuad组件
3. [ScreenQuad组件](https://github.com/alphardex/kokomi.js/blob/main/src/shapes/screenQuad.ts)本质上是一个铺满屏幕的平面元素，里面内置了几乎所有shadertoy的uniform变量，并且能根据时间等要素进行渲染更新

## 最后

桌面是不是有种焕然一新的感觉？

Web 3D，永远超乎你的想象。