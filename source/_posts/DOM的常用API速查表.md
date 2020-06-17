title: DOM的常用API速查表
author: alphardex
tags:
  - JavaScript
  - DOM
categories:
  - 前端
date: 2020-06-06 17:22:00
---
大多数情况下，笔者坚信能用CSS解决的，就不会用JS来解决

但是，CSS在某些情况下还是有一定的局限性，比如跟踪鼠标事件，监视页面滚动等

这时，我们就要借助强大的DOM API了，在不用框架的基础上，就能用原生JS华丽地解决一些常见但CSS做不到的需求

本文罗列了笔者平时最常用的DOM API

<!--more-->

# Document

页面文档

## document

### querySelector

获取CSS选择器匹配到的单个元素，后面加`All`表示匹配所有元素

### createElement

创建一个元素

### designMode

设为`on`即可自由编辑页面

# Nodes

节点

## Node

### textContent

获取元素的文本内容

### append

为元素添加子元素

### parentElement

获取元素的父元素

# Element

元素

## element

### className

类名

### classList

类名列表，常用方法：`add`、`remove`、`contains`、`toggle`

### animate

为元素添加动画，具体参见 [Web Animations API](https://devdocs.io/dom/web_animations_api/using_the_web_animations_api)

### setAttribute

设置元素的属性键值对

### getAttribute

获取元素某一属性值

### removeAttribute

去除元素某一属性值

### getBoundingClientRect

返回元素的大小

### scrollBy

使元素沿xy轴滚动特定距离

### clientWidth | clientHeight

获取元素内部宽高（`content + padding`）

# Elements

包括`HTMLElement`等的所有`HTML`标签元素

## HTMLElement

### style

元素样式

`getPropertyValue`方法用于获取自定义变量的值

`setProperty`方法用于设置自定义变量

### dataset

获取元素的自定义属性键值对

### offsetWidth | offsetHeight

获取元素宽高（`content + padding + border`）

### tagName

获取元素标签名

## HTMLInputElement

### value

表单元素的值

### checked

单复选框的勾选状态

### checkValidity

验证表单元素的合法性

### setCustomValidity

将元素定义为`invalid`并附加信息，空字符串表示元素`valid`

### files

`input type="file"`返回的`FileList`

# Event

事件

## EventTarget

### addEventListener

监听某个事件，请参考DOM Event速查表

## Event

### target

事件的发出者

### currentTarget

事件的监听者

### preventDefault

防止浏览器的默认行为

### stopPropagation

防止冒泡

## MouseEvent

### clientX | clientY

鼠标事件时鼠标的XY位置

## KeyBoardEvent

### code

键盘事件时键盘按下的键的代号

## ProgressEvent

ajax请求事件

### loaded

已加载资源大小

### total

资源总大小

## MessageEvent

`Worker`通信事件

### data

接收到的信息

# Location

代表着当前的URL对象

## location

- `href`：URL字符串，若更改则会跳转到别的页面
- `search`：查询字符串

# URL

## URL

### createObjectURL

为`Blob`对象创建URL，通常用于图片预览功能

## URLSearchParams

将查询字符串解析为可迭代的对象

# File

文件

## File

### name

文件名

### size

文件大小

## Blob

### slice

将文件切成二进制块

## FileReader

文件读取器

### readAsArrayBuffer

读取文件数据

### onload

读取完毕时触发

### result

读取的文件内容

# Window

全局

## window

### setTimeout

定时器，清空用`clearTimeout`

### setInterval

重复定时器，清空用`clearInterval`

### requestAnimationFrame

按帧对网页进行重绘

### localStorage

可以在本地缓存数据

- setItem: 存
- getItem: 取
- removeItem: 删
- clear: 清

### innerWidth | innerHeight

视窗宽高

### fetch

从服务器获取数据

# Navigator

## navigator

### userAgent

当前浏览器的`userAgent`，常用于判断浏览器型号

# XMLHttpRequest

ajax请求

## XMLHttpRequest

### open

新建请求，设置`method`和`url`

### setRequestHeader

设置请求头键值对

### send

将请求发送给服务器

### onload

请求成功时触发

### response

响应的`body`数据

### responseType

响应类型：`blob`、`json`等

### onprogress

监视xhr的进度

### upload

`onprogress`：监视上传xhr的进度

### abort

取消请求

## FormData

`send`发送的`body`数据

### append

添加表单数据键值对

# Fetch

从服务器获取数据

## Response

### ok

响应成功

## Body

### json

获取`json`形式的响应数据

# Web Workers

可以将耗时的任务放在后台运行，使得主线程（UI）不会出现阻塞

## Worker

### importScripts

引入脚本

### postMessage

用于`Worker`和主页面之间传递信息

### onmessage

监听传递的信息

### close

关闭`Worker`

# Intersection Observer

交叉观测器，用于观察元素在当前视口上是否“可见”，常用于图片懒加载，滚动时监听动画等

## IntersectionObserver

观察器

### observe

观察某元素

## IntersectionObserverEntry

被观察的元素

### isIntersecting

被观察的元素与当前视口是否交叉

# DOM Event

常见的DOM事件

## Mouse

### click

鼠标点击时触发

### mouseenter

鼠标移入时触发

### mouseleave

鼠标移出时触发

## Form

### change

`input`，`select`，`textarea`等元素的`value`改变时触发

## Load

### load

当一个资源以及其依赖的资源加载完毕时出发，常用：`window`