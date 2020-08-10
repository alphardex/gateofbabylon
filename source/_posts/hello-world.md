---
title: JavaScript语言专属特性
tags:
  - JavaScript
categories:
  - 前端
author: alphardex
abbrlink: 32589
date: 2020-06-06 16:57:00
---
本文归纳了笔者最常用的 JavaScript 语言专属特性。

<!--more-->

# 函数

## 箭头函数

函数的简易定义方式，配合高阶函数`map`、`filter`、`reduce`实现函数式编程

### map - 映射

``` javascript
[1, 2, 3, 4, 5].map(e => e ** 2);
// [1, 4, 9, 16, 25]
```

### filter - 过滤

``` javascript
[1, 2, 3, 4, 5].filter(e => e > 3);
// [4, 5]
```

### reduce - 归并

``` javascript
[1, 2, 3, 4, 5].reduce((acc, cur) => acc + cur);
// 15
```

# 解构赋值

## 解构类型

### 数组解构

``` javascript
let [x, y] = [1, 2];
console.log(x, y);
// 1 2
```

### 对象解构

``` javascript
let {name, age} = {name: 'alphardex', age: 23};
console.log(name, age);
// alphardex 23
```

## 应用场景

### 提取JSON数据

``` javascript
let { title, content } = jsonData;
```

### 返回函数多个值

``` javascript
let [x, y, z] = func();
```

### 交换变量值

``` javascript
let a = 1, b = 2;
[a, b] = [b, a];
```

# 数组

## 扩展运算符

转换数组为用逗号分隔的参数序列

### 合并\拷贝数组或对象

``` javascript
let arr1 = [1, 2], arr2 = [3, 4];
let arr = [...arr1, ...arr2];
console.log(arr);
// [1, 2, 3, 4]
let obj1 = {name: 'alphardex'}, obj2 = {age: 24};
let obj3 = {...obj1, ...obj2};
console.log(obj3);
// { name: 'alphardex', age: 24 }
let arr = [1, 2, 3];
let arrCopy = [...arr1];
console.log(arrCopy);
// [1, 2, 3]
```

### 获取剩余元素

``` javascript
let [x, ...rest] = [1, 2, 3];
console.log(rest);
// [2, 3]
```

# for of

for of 可用于迭代可迭代对象

可迭代对象包括数组、类数组对象、字符串、Map、Set、生成器等，且迭代时可以中断

``` javascript
let arr = [1, 2, 3];

for (let value of arr) {
  console.log(value);
  if (value === 2) {
    break;
  }
}
// 1 2
```

# async

async 函数返回隐式的 Promise，是异步代码的理想的编写方式

```javascript
let sleep = time => new Promise(resolve => setTimeout(resolve, time));

async function main() {
  console.log(`开始：${new Date()}`);
  await sleep(1000);
  console.log(`结束：${new Date()}`);
}

main();
```

假设定义了 3 个异步操作任务

```javascript
async function task1() {
  console.log("task 1 start");
  await sleep(1000);
  console.log("task 1 end");
}

async function task2() {
  console.log("task 2 start");
  await sleep(1000);
  console.log("task 2 end");
}

async function task3() {
  console.log("task 3 start");
  await sleep(1000);
  console.log("task 3 end");
}
```

## 顺序执行

```javascript
async function main() {
  const tasks = [task1, task2, task3];
  for (let i = 0; i < tasks.length; i++) {
    await tasks[i]();
  }
}

main();
// task 1 start
// task 1 end
// task 2 start
// task 2 end
// task 3 start
// task 3 end
```

## 并发执行

```javascript
function main() {
  const tasks = [task1, task2, task3];
  tasks.forEach(task => task());
}

main();
// task 1 start
// task 2 start
// task 3 start
// task 1 end
// task 2 end
// task 3 end
```

## 任务完成后执行某操作

```javascript
async function main() {
  await Promise.all([task1(), task2(), task3()]);
  console.log("all complete!");
}

main();
// task 1 start
// task 2 start
// task 3 start
// task 1 end
// task 2 end
// task 3 end
// all complete!
```