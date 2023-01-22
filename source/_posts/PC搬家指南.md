title: PC搬家指南
author: alphardex
abbrlink: 54698
tags: []
categories: []
date: 2020-09-13 09:57:00
---
前提是把所有文件备份到了[百度网盘](https://pan.baidu.com/download/)上

<!--more-->

# 装机

## 软件

Installers文件夹里的安装包按需安装

chrome-extensions在chrome浏览器里全部解压

将bookmarks.html导入chrome书签

vscode-extensions在vscode里全部解压

## 配置

将Installers里的config.zip解压到C盘的用户文件下

## 开发

首先装好所有全局包

```sh
npm install -g cloc hexo-cli http-server npkill rimraf typescript
```

```sh
pip install looter
```

然后将gitee上的python-gadgets里的batch_clone_repo.py搞到本地运行，将所有repo克隆到本地

## 资源

将度盘上的资源拷到本地即可