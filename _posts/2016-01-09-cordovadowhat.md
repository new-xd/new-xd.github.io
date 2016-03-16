---
layout:     post
title:      "Cordova都做了什么事"
subtitle:   "了解cordova的主要功能"
date:       2016-01-09 18:22:00+0800
author:     "New"
header-img: ""
tags:
    - cordova
---

- 用html、css、js写app
    - 提供各平台运行前端代码的应用壳
    - 提供将前端代码转化为应用的命令行工具（编译打包）
    - 提供应用信息的在各平台的统一配置（通过工具实现）
    - html css js的运行环境
- 提供js接口调用手机功能
    - 提供完整的插件机制扩展js的能力
    - 提供一些基础功能的插件
    - 提供插件操作工具
    - 提供js和native的通信

## 提供各平台运行前端代码的应用壳

为了将前端代码成功的运行在各平台，cordova提供了各平台的模板应用，
通过把前端代码配置到模板应用中，再对模板应用编译打包，从而生成一个用html、css、js开发的应用

## 提供将前端代码转化为应用的命令行工具（编译打包）

如何在平台配置前端代码是一个较为复杂的过程，当然由cordova工具实现。
配置好各平台的编译环境后，也可以通过cordova编译打包应用

## 提供应用信息的在各平台的统一配置（通过工具实现）

应用的名称、版本等信息每个平台的配置方式都不一样，
不过不用担心，只要在config.xml中配置，cordova工具会自动根据config.xml中的信息配置各平台的工程目录

## html css js的运行环境

为了完美的运行大部分应用，cordova对各平台的webview都进行了针对各平台的设置，以及一些bug fix
已支持前端的大部分特性

## 提供完整的插件机制扩展js的能力

插件是对js的扩展，以js接口的形式提供一部分native功能或者js功能。
插件的机制，包括插件如何加载，插件卸载，插件的js接口如何定义，插件的native代码如何实现
插件通过什么样的接口实现js-native通信

## 提供一些基础功能的插件

官方提供了一些基础插件，方便开发者调用，见[官方文档](http://cordova.apache.org/docs/en/latest/guide/overview/)左侧导航栏下方的PluginAPIs

## 提供插件操作工具

cordova 包含了插件的安装卸载等功能
plugman 提供了较为完整的插件操作

## 提供js和native的通信

各平台的js和native之间的通信方法都不相同，cordova对此进行了封装，统一了各平台js和native直接的通信接口
