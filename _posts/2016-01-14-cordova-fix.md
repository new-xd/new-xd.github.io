---
layout:     post
title:      "Cordova 坑"
subtitle:   "自己遇到的一些坑"
date:       2016-01-14 21:50:00+0800
author:     "New"
header-img: ""
tags:
    - cordova
---

用于积累自己使用过程中，遇到的一些坑及自己的解决
慢慢完善

#### npm一直在转圈
赶紧用[cnpm](http://npm.taobao.org/)

#### 每次工程、平台、插件都是重新下载，好慢
可以创建一个干净的工程，添加好平台，然后备份好，
之后每次新开项目拷贝备份，
然后在config.xml修改包名 应用名 版本 再cordova prepare就ok了

插件也可以备份，一般的插件更新不频繁，把plugins下的插件备份起来
安装的时候，用本地的相对路径安装

#### 经常编译时获取不到maven库的metadata.xml而失败
这个可能是我的个人情况，时有发生
在执行cordova build 或者 cordova run时，总是说无法解决依赖
因为获取https://repo1.maven.org/maven2/.../maven-metadata.xml失败
而我通过浏览器访问确实连接不上，但是我把https改为http就可以。。

没办法，我就修改platforms\android\cordova\lib\plugin-build.gradle
把

    repositories {
        mavenCentral()
    }

改为

    repositories {
        maven {
            url "http://repo1.maven.org/maven2/"
        }
    }

然后就好了，这个文件是Android工程中build.gradle的模板文件，改这个才有效

#### ionic上的 platform-xxx 标签
这个标签是通过hooks脚本添加上去，而脚本里找的都是默认位置的index.html，
所以如果移动了index.html的位置，这个标签是没有的。
虽然到现在也没发现少了这个标签会有什么后果，不过先记着。

#### CoreAndroid.java 异常
在这个插件的onDestroy中，注销了一个广播监听
虽然理论上没有问题，但是不知道什么情况下，
onDestroy会执行两次，导致重复注销，从而异常，crash。
规避手段就是 注销之后 置null，再加上非空判断再注销

#### setNetworkAvaible 异常
这个情况多出现在native异步执行js调用，但是在native执行完之前页面已经关闭
此时再调用回调，会触发native通知js，从而调用setNetworkAvaible导致异常 crash
规避手段在调用回调之前判断页面是否已经关闭（不再需要执行结果了）