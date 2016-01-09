---
layout:     post
title:      "Cordova介绍"
subtitle:   "初步了解cordova，看下效果"
date:       2016-01-08 23:00:00
author:     "New"
header-img: ""
tags:
    - cordova
---

## 是什么？
Apache Cordova is an open-source mobile development framework.
It allows you to use standard web technologies such as HTML5, 
CSS3, and JavaScript for cross-platform development, avoiding
 each mobile platforms' native development language.

它是开源的移动应用开发框架。它允许你使用标准的web技术比如html5 css3 js
进行跨平台开发，从而避免使用各移动平台的native开发语言。

简单来说，就是可以用html css js等web技术开发移动应用，一套代码多平台使用

## 为什么要用
- 跨平台，不用维护多份代码
- 前端开发比native开发的工作量小，维护成本也低
- 现在的移动设备，性能强劲，体验不至于还可以接受

对于一些企业应用，用户体验要求不是很高的情况下，还是可以使用的。

### 为什么我会对这个感兴趣
作为一个android开发工程师，有以下场景

- 应用中需要添加一些小应用，可以使用，方便添加，更新
- 对于已经使用这个框架的应用，需要添加一些定制的功能

## 安装

1. 安装[nodejs](https://nodejs.org/en/)
2. 配置[cnpm](http://npm.taobao.org/)
3. cnpm install -g cordova

其实不用cnpm也可以，就是有时会很慢
这样cordova就安装好了，可以创建工程、安装插件、prepare工程，但是不能编译运行

cordova的跨平台是说*它帮你把html代码放到各平台配置好的native工程中*，编译运行
还是需要你配置好各平台的环境，它才能帮你编译运行
所以想要哪个平台的应用，就[搭这个平台的环境](http://cordova.apache.org/docs/en/latest/guide/platforms/index.html)吧
下面是android平台需要的环境

1. 安装JDK
2. 安装android SDK， 当然还包括流行的android版本、build tools、Android Support Repository
3. 将SDK中的platform-tools 和 tools 添加到环境变量中
4. 创建android 模拟器，一般都会用genymotion

这样的android环境就具备了。也算是安装完成了。对于android开发者来说，这些步骤一般都已经做了。

## 使用
说一下常用的命令，以android为例

创建工程

    cordova create app

这样会在当前目录下创建一个app文件夹，即工程根目录，包含一个cordova默认的工程

进入这个目录

    cd app

此时作为前端开发工程师，就可以开发前端代码了

添加android平台

    cordova platform add android

此时还可以添加一些需要的plugin

    cordova plugin add xxx

xxx可以是插件ID、插件的git链接、本地的相对路径

之后就是运行了

    cordova run android

此时应该就可以在设备上或者genymotion中看到运行的app了
