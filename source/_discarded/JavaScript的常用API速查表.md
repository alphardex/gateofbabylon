title: JavaScript的常用API速查表
author: alphardex
tags:
  - JavaScript
  - Cheatsheet
categories:
  - 前端
abbrlink: 59160
date: 2020-06-06 17:13:00
---
本文罗列了JavaScript的常用API。

<!--more-->

# Array

## array

### push

数组末尾添加元素

### forEach

对数组中每个元素执行一次回调函数

### map

对数组中每个元素执行一次回调函数并返回新数组

### filter

过滤掉回调函数返回false的值，将剩余值返回

### reduce

对数组的每个元素执行一个归并函数，返回一个值

### includes

判断数组是否包含一个特定元素

### slice

返回一个数组切片

### find

找到数组中第一个满足回调函数的值

### findIndex

找到数组中第一个满足回调函数的值的下标

### join

将数组中的所有元素拼接成一个字符串，支持分隔符

### sort

原地排序数组

### reverse

原地反转数组

### every

当所有元素通过回调函数测试时返回true

### some

当任一元素通过回调函数测试时返回true

## Array

### from

从一个可迭代的对象创建出一个数组实例

### length

数组长度

# Global Objects

## Global

### parseInt

将字符串解析为整数

# JSON

## JSON

### parse

将字符串解析为对象

### stringify

将对象转化为字符串

# Math

## Math

### random

0-1之间的随机数

### abs

绝对值

### max

最大值

### min

最小值

# Number

## number

### toFixed

保留n位小数

# Object

## Object

### assign

将所有可枚举属性的值从一个或多个源对象复制到目标对象，并返回目标对象

### entries

获取对象所有property的键值对数组

### keys

获取对象所有property的键数组

### values

获取对象所有property的值数组

# Promise

## Promise

### all

接受多个Promise实例，当这些实例都被resolve时才返回一个Promise

## promise

### then

返回一个Promise

### catch

返回一个Promise，并只处理reject的情况

# String

## string

### split

将字符串以特定的分隔符分割为多个子字符串

### toUpperCase

转大写

### toLowerCase

转小写

### trim

去除左右两端的空格

### match

返回通过正则匹配的字符串数组

### replace

返回字符串，替换掉与正则相匹配的部分

# RegExp

## regExp

### test

判断字符串是否通过正则匹配