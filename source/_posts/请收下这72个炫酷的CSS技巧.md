title: 请收下这72个炫酷的CSS技巧
author: alphardex
abbrlink: 18258
tags: []
categories: []
date: 2019-12-25 19:34:00
---
## 前言

CSS是一门很特殊的语言，不像一般的编程语言那样需要抽象的思维和严密的逻辑，它真正需要的是**想象力**——将你脑中所想的意象用代码来表现出来。那么意象又是如何产生的呢？最常用的方法就是**探索和观察**。举个例子，笔者平时热爱看番，每看到有意思的场景画面总会下意识地记录下来，这对动画创作会非常有帮助；同样地，一旦笔者发现一个制作精良的网站，也会将网站上那些吸引人的元素仔细审查一遍，灵感也自然就有了。

演示网址1：[codepen](https://codepen.io/alphardex)

演示网址2：[shiroi](https://shiroi.netlify.com)

源码地址：[github](https://github.com/alphardex/shiro)

本文的技巧将不会止步于72个。灵感不息，创作不止。
<!--more-->

## 注意

### 兼容性

本文的所有技巧都**未考虑兼容性**，因为个人认为兼容性是一种束缚，它会妨碍你写出优秀的作品。如果硬是要考虑的话请自行解决，[这个网站](https://caniuse.com)或许能帮到你。

### 框架

本文所用到的技巧皆是SCSS+TypeScript。如果想移植到React或Vue上的话请根据框架本身的特点自行适配。笔者不用这类框架的原因很简单：框架很容易就会过时，原生CSS+JS才是王道。

### 教程

笔者实在是不擅长写这类东西，不过倒是可以把常用的属性和模式列出来，供大家参考参考。

常用属性：[猛戳这里](https://juejin.im/post/6844904033145061389)

常用模式：[猛戳这里](https://juejin.im/post/6844904033405108232)

## 动画

### 利用不同的`delay`实现交错动画

- [Reveal Text](https://codepen.io/alphardex/pen/eYYMYXJ)
- [Staggered Stair Loading](https://codepen.io/alphardex/pen/MWWvRRR)
- [Staggered Square Loading](https://codepen.io/alphardex/pen/LYYZZEz)
- [Staggered Wave Loading](https://codepen.io/alphardex/pen/XWWWBmQ)
- [Gleaming Loading](https://codepen.io/alphardex/pen/QWLYVjV)
- [Particle Burst](https://codepen.io/alphardex/pen/pozqwrO)
- [Gleaming Heading](https://codepen.io/alphardex/pen/rNBrExx)
- [Staggered Shrinking Loading](https://codepen.io/alphardex/pen/eYmORVe)
- [Snow](https://codepen.io/alphardex/pen/dyPorwJ)
- [Staggered Rise In Text](https://codepen.io/alphardex/pen/qBEmGbw)
- [Staggered LandIn Text](https://codepen.io/alphardex/pen/KKwvKGY)

## 文本

### 利用`background-clip:text`配合`color`实现渐变文字效果

- [Shining Text](https://codepen.io/alphardex/pen/VwweapQ)
- [Menu Hover Fill Text](https://codepen.io/alphardex/pen/QWwveZG)

### 利用动态`hsl`颜色实现彩虹文字效果

- [Rainbow Color Text](https://codepen.io/alphardex/pen/ExxQOWV)

### 利用`text-shadow`实现发光文字效果

- [Neon Text](https://codepen.io/alphardex/pen/rNNwmZz)
- [Staggered Glow In Text](https://codepen.io/alphardex/pen/Exxodoq)

### 利用`text-shadow`实现伪3D文字效果

- [Staggered Bouncing 3D Loading](https://codepen.io/alphardex/pen/QWWavvx)

### 利用`web animation`实现冒泡文字效果

- [Bubbling Text](https://codepen.io/alphardex/pen/LYYLdoY)

### 利用动态`max-width`实现文本展开效果

- [Abbr Expansion](https://codepen.io/alphardex/pen/xxKvvMQ)

### 利用绝对定位、3D变换和JS实现翻转文字

- [Rotating Text](https://codepen.io/alphardex/pen/WNNVJeZ)

## 视觉

### 利用`backdrop-filter`实现毛玻璃背景效果

- [Frosted Glass](https://codepen.io/alphardex/pen/pooQMVp)

### 利用背景、绝对定位和`filter`实现毛玻璃景深效果

- [Frosted Glass Depth of Field](https://codepen.io/alphardex/pen/ZEEZpQG)

### 利用`blur`和`contrast`滤镜实现融合效果

- [Snow Scratch](https://codepen.io/alphardex/pen/BaBevXm)

### 利用元素叠加`blur`滤镜实现日光效果

- [Eclipse Loader](https://codepen.io/alphardex/pen/gOOPKJE)
- [Glowing List Hover](https://codepen.io/alphardex/pen/KKKLdaR)
- [Glowing Gradient Border](https://codepen.io/alphardex/pen/GRRaMOV)
- [Glowing Gradient Button](https://codepen.io/alphardex/pen/xxxNpad)
- [Crimson Crescent Loading](https://codepen.io/alphardex/full/eYmGEGp)

### 利用`mix-blend-mode:screen`实现文本遮罩效果

- [Video Mask Text](https://codepen.io/alphardex/pen/wvvLYpV)

### 利用`-webkit-box-reflect`实现倒影效果

- [Card Flip Reflection](https://codepen.io/alphardex/pen/ExaZgxp)

## 页面

### 利用3D变换实现视差效果

- [Parallax](https://codepen.io/alphardex/pen/qBEZELp)

### 利用`position:sticky`实现粘性滚动效果

- [Sticky Sections](https://codepen.io/alphardex/pen/YzPqeMm)

### 利用绝对定位和交错动画实现镜头拉伸背景效果

- [Ken Burns Effect](https://codepen.io/alphardex/pen/wvBovww)

### 利用伪元素、绝对定位和动画实现滑动幻灯片

- [Animated Image Slider](https://codepen.io/alphardex/details/VwYWpJN)

## 组件

### 利用`border-radius`实现曲边导航栏

- [Nav Tab](https://codepen.io/alphardex/pen/abbWOPR) 

### 利用动画和绝对定位实现汉堡菜单

- [Burger Menu](https://codepen.io/alphardex/pen/BaaKvVZ)

### 利用伪元素和动画实现动态划线效果

- [Menu Hover Underline](https://codepen.io/alphardex/pen/MWWEmLK)
- [Menu Hover Magnify](https://codepen.io/alphardex/pen/ExxPRXN)
- [Button Hover Border Stroke With Float Text](https://codepen.io/alphardex/pen/pooYKVa)
- [Header With Slide Bar](https://codepen.io/alphardex/pen/jOEOEzZ)
- [Button Hover Multiple Border Stroke](https://codepen.io/alphardex/full/ZEYXomW)

### 利用伪元素和`overflow:hidden`实现交错分割文本菜单

- [Split Text Menu](https://codepen.io/alphardex/pen/wvvxVPj)
- [Staggered Float Text Menu](https://codepen.io/alphardex/pen/wvBeXjd)
- [Shinchou Menu](https://codepen.io/alphardex/pen/ExavZdV)（慎重勇者风格菜单）

### 利用伪元素和`overflow:hidden`实现填充按钮

- [Confirm Modal](https://codepen.io/alphardex/pen/eYYxzBm)

### 利用伪元素、渐变和`overflow:hidden`实现闪光按钮

- [Button Hover Shining](https://codepen.io/alphardex/pen/eYYzXBZ)

### 利用绝对定位、动画、渐变和`overflow:hidden`实现蛇形边框按钮

- [Snake Border Button](https://codepen.io/alphardex/pen/qBBGxqY)

### 利用伪元素、渐变、背景运动实现动态渐变边框

- [Gradient Border](https://codepen.io/alphardex/pen/vYEYGzp)

### 利用`oveflow:hidden`、`max-height`和`:target`实现手风琴菜单

- [Accordion Menu](https://codepen.io/alphardex/pen/xxxNbar)
- [Accordion Panel](https://codepen.io/alphardex/pen/LYEYaoJ)

### 利用`overflow:hidden`和`scroll`相关属性实现无缝轮播图

- [Carousel](https://codepen.io/alphardex/pen/RwwqqJE)

### 利用兄弟选择器配合伪元素自定义单复选框

- [Todo List](https://codepen.io/alphardex/pen/rNNPQwa)
- [Radio Button](https://codepen.io/alphardex/pen/JjjpNWQ)
- [Checkbox](https://codepen.io/alphardex/pen/yLLjoaL)
- [Toggle](https://codepen.io/alphardex/pen/poopqvE)
- [Elevator Switch](https://codepen.io/alphardex/pen/YzzMMPx)

### 利用各种属性实现各种按钮特效

- [Button Collection](https://codepen.io/alphardex/pen/VwwVLdM)
- [Share Button](https://codepen.io/alphardex/pen/qBBwXZm)
- [Login Button](https://codepen.io/alphardex/pen/VwZNqEK)
- [One-Field Login Form](https://codepen.io/alphardex/pen/xxxRPLE)

### 利用多重`box-shadow`阴影实现发光按钮菜单

- [Glowing Menu Buttons](https://codepen.io/alphardex/pen/ExxKjEN)

### 利用`counter`在伪元素的`content`中显示`var`的值

- [Progress Bar](https://codepen.io/alphardex/pen/MWWwNeR)

### 利用`-webkit-slider-thumb`定制滑块

- [Gradient Range Slider](https://codepen.io/alphardex/pen/QWWXyee)

### 利用伪类校验表单

- [Transparent Material Login Form](https://codepen.io/alphardex/pen/zYYZorR) 

### 利用动画实现卡片展开

- [Card Hover Expand Body](https://codepen.io/alphardex/pen/YzzGjLL)

### 利用`clip-path`实现卡片多方向展开

- [Name Card Hover Expand](https://codepen.io/alphardex/pen/ZEEBRrq)

### 利用没有`perspective`的`transform-style:preserve-3d`实现等距3D效果

- [3D Cube](https://codepen.io/alphardex/pen/yLLrReg)
- [Isometric Icon Hover](https://codepen.io/alphardex/pen/oNNRxGQ)
- [Isometric Images Hover](https://codepen.io/alphardex/pen/QWWRMew)
- [Isometric Icon Nav Bar](https://codepen.io/alphardex/pen/YzPKENd)

### 利用3D变换实现3D下拉菜单

- [3D Dropdown Menu](https://codepen.io/alphardex/pen/rNaNyev)

### 利用动画和JS实现简单的分页栏

- [Pagination](https://codepen.io/alphardex/pen/QWwwwpp)

### 利用伪元素、`overflow:hidden`、动画、JS实现标签页

- [Tabs](https://codepen.io/alphardex/pen/vYEEdGK)

### 利用伪元素、`:checked`、`~`兄弟选择器实现5星评分

- [Star Rating](https://codepen.io/alphardex/pen/povvGNZ)

### 运用伪元素、层叠关系、3D变换、JS实现翻牌时钟

- [Flip Card Clock](https://codepen.io/alphardex/pen/vYYoBNR)

### 利用鼠标事件监听和`web animation`实现图片悬浮菜单

- [Menu Hover Image](https://codepen.io/alphardex/pen/OJPmQGz)

### 利用`conic-gradient`，伪元素和CSS变量实现圆盘度量计

- [Gauge (No SVG)](https://codepen.io/alphardex/pen/BaydVvQ)