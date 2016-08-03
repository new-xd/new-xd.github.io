---
layout:     post
title:      "Cordova介绍"
subtitle:   "初步了解cordova，看下效果"
date:       2016-01-08 23:00:00+0800
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

## 应用场景

- 想要开发android ios应用，但是不会java和Objective-c，学习成本高，学习前端技术可以较快的写出应用，而且android和ios都有了
- 已有的native应用中，某个模块用前端技术快速实现

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

其实不用cnpm用npm也可以，就是有时会很慢
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

###　如何创建工程

创建工程

    cordova create app

这样会在当前目录下创建一个app文件夹，即工程根目录，包含一个cordova默认的工程

进入这个目录

    cd app

此时作为前端开发工程师，就可以开发前端代码了

添加android平台

    cordova platform add android

之后就是运行了

    cordova run android

此时应该就可以在设备上或者genymotion（当然你要先打开一个denymotion模拟器）中看到运行的app了  
此时显示的应用是cordova的sample，你可以在此基础上进行开发  

注意，不需要把项目导入Eclipse/AndroidStudio/Xcode

### 如何在js中调用手机硬件功能
比如 摄像、震动  
答案是插件，cordova提供了一些基础插件，以调用手机功能，需要哪个，安装哪个  
安装命令  

    cordova plugin add xxx

xxx可以是插件id、是插件代码地址（网络或者本地都可以）  
卸载命令  

    cordova plugin rm xxx

xxx是插件的id，可以通过下面的命令查看。  
列出本地已安装的插件  

    cordova plugin ls

这个命令会列出你项目中已安装的插件信息，包括插件id和插件版本

插件的作用不局限与调用手机硬件功能，它本质上只是定义了一套js调用native代码的机制和规范，  
当你熟悉怎么开发插件之后，你也可以把一些js不好处理的逻辑放到native去做。  

安装之后怎么用呢？这个需要查看各插件的[api文档](http://cordova.apache.org/docs/en/latest/#plugin-apis)

### cordova工程、各手机平台、各插件关系

首先说明下  
cordova工程是一个跨平台工程  
一个工程中可以创建android、ios平台的app
要创建哪个平台的app，可以通过

    cordova platform add android
    cordova platform add ios

创建，当然也可以删除

    cordova platform rm android
    cordova platform rm ios

也可以查看本工程已创建了哪些平台

    cordova platform ls

一个工程可以包括多个平台，在编译的时候也可以同时编译多个平台  
（当然要有多个平台的编译环境，cordova不编译，它只是调用各平台编译工具编译）

插件是用js接口提供native功能，是哪个平台的native功能呢？  
这需要看插件的实现，插件可以是跨平台，也可以是只实现了某一个平台。  
所以真正使用插件时，需要确定它支持哪些平台，是否满足你的需要。

当安装插件时，cordova会在各平台安装此插件，  
如果插件只实现了android端，那在ios端虽然可能显示安装了插件，但是实际没有具体实现
当创建平台时，cordova会记录已安装的插件，并把这些插件安装到新创建的平台  

简单来说就是 注意下插件不一定所有平台都实现了，其他的安装卸载，cordova已经做的很好了

### 编译运行

编译  

    cordova build      // 都编译
    cordova build android  // 编译android app
    cordova build ios // 编译ios app

build命令实际上分两步

    cordova prepare
    cordova compile

compile是真正的编译命令，那prepare是做什么的呢？  

1. 根据cordova config.xml修改各平台的配置
2. 根据安装的插件修改各平台的代码
3. 将html js css代码正确的放到各平台的工程中

简单来说就是 将各平台编译的代码准备好  

运行，真机运行

    cordova run android
    cordova run ios

注意，cordova把genymotion当做真机看待

模拟运行

    cordova emulator android
    cordova emulator ios

ios要能够在模拟器运行，需要在安装[官网](http://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html)的说明配置下环境




