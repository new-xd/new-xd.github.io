---
layout:     post
title:      "Cordova工程结构"
subtitle:   "让我们看看cordova代码是怎么组织的"
date:       2016-01-09 08:21:00+0800
author:     "New"
header-img: ""
tags:
    - cordova
---

如果英文不是问题，请移步[官方文档](http://cordova.apache.org/docs/en/latest/)
官方文档，肯定比我的权威可信，而且更新及时

## 默认的目录结构
创建一个cordova工程之后，默认的目录结构如下

    ├─.
    │  config.xml
    │  
    ├─hooks
    │      README.md
    │      
    ├─platforms
    ├─plugins
    └─www
        │  index.html
        │  
        ├─css
        │      index.css
        │      
        ├─img
        │      logo.png
        │      
        └─js
                index.js

config.xml是cordova的配置文件，下面重点介绍下
hooks是存放cordova命令钩子脚本的目录，主要为了一些项目特殊需要，方便对cordova命令做功能扩展
platforms是存放各平台的代码，此目录代码由cordova生成，一般不需要手动修改
plugins是存放安装的插件的源代码，安装插件时会先把插件代码下载到此目录，然后开始安装到各平台
www存放前端代码，即应用的主要代码，主要的工作目录

## config.xml

下面是config.xml中的内容

    <?xml version='1.0' encoding='utf-8'?>
    <widget id="io.cordova.hellocordova" version="0.0.1" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">
        <name>HelloCordova</name>
        <description>
            A sample Apache Cordova application that responds to the deviceready event.
        </description>
        <author email="dev@cordova.apache.org" href="http://cordova.io">
            Apache Cordova Team
        </author>
        <content src="index.html" />
        <plugin name="cordova-plugin-whitelist" version="1" />
        <access origin="*" />
        <allow-intent href="http://*/*" />
        <allow-intent href="https://*/*" />
        <allow-intent href="tel:*" />
        <allow-intent href="sms:*" />
        <allow-intent href="mailto:*" />
        <allow-intent href="geo:*" />
        <platform name="android">
            <allow-intent href="market:*" />
        </platform>
        <platform name="ios">
            <allow-intent href="itms:*" />
            <allow-intent href="itms-apps:*" />
        </platform>
    </widget>

其中widget的属性id是应用ID，对于android就像应用程序的在manifest中声明的包名，version就是应用版本。
content标签的src是指应用启动首先加载的页面，是一个相对路径，相对于工程的www文件夹。
plugin标签说明应用需要的插件，name为插件ID。

> \<access\> elements define the set of external domains the app is allowed to communicate with. 
The default value shown above allows it to access any server.

access标签配置应用可以通信的外部域名。默认值\*表示所有域名
allow-intent 是配置 js调用cordova提供的loadUrl接口 可以打开的外部界面（这个界面包括native界面和网页）
platform标签配置各平台特有的配置

#### 注意

这个config.xml是对应用的统一配置，  
对于某个平台，会在此config.xml基础上添加该平台需要的插件信息、参数信息、针对性配置等，生成新的config.xml  
各平台的config.xml都在各平台的代码中，不需要手动修改，每次编译打包的时候，会重新生成。  
手动修改可能会造成一些不能理解的问题（了解细节才能发现问题）  

## 添加一个Android平台
执行

    cordova platform add android

之后，platforms下会多一个android文件夹，里面是android平台的代码，
plugins下会多一个cordova-plugin-whitelist插件的源代码，如下

    ├─.
    │  config.xml
    │  
    ├─hooks
    │      README.md
    │      
    ├─platforms
    │  │  platforms.json
    │  │  
    │  └─android
    │      │  .gitignore
    │      │  AndroidManifest.xml
    │      │  build.gradle
    │      │  project.properties
    │      │  settings.gradle
    │      │  
    │      ├─assets
    │      │  └─www
    │      │      │  cordova.js
    │      │      │  cordova_plugins.js
    │      │      │  index.html
    │      │      │  
    │      │      ├─cordova-js-src
    │      │      │  │  exec.js
    │      │      │  │  platform.js
    │      │      │  │  
    │      │      │  ├─android
    │      │      │  │      nativeapiprovider.js
    │      │      │  │      promptbasednativeapi.js
    │      │      │  │      
    │      │      │  └─plugin
    │      │      │      └─android
    │      │      │              app.js
    │      │      │              
    │      │      ├─css
    │      │      │      index.css
    │      │      │      
    │      │      ├─img
    │      │      │      logo.png
    │      │      │      
    │      │      ├─js
    │      │      │      index.js
    │      │      │      
    │      │      └─plugins
    │      │          └─cordova-plugin-whitelist
    │      │                  whitelist.js
    │      │                  
    │      ├─cordova
    │      │  │  ...这个文件夹下是cordova在android平台的命令行工具，省略
    │      │                  
    │      ├─CordovaLib
    │      │  │  AndroidManifest.xml
    │      │  │  build.gradle
    │      │  │  cordova.gradle
    │      │  │  project.properties
    │      │  │  
    │      │  └─src
    │      │      └─org
    │      │          └─apache
    │      │              └─cordova
    │      │                  │  AuthenticationToken.java
    │      │                  │  CallbackContext.java
    │      │                  │  Config.java
    │      │                  │  ConfigXmlParser.java
    │      │                  │  CordovaActivity.java
    │      │                  │  CordovaArgs.java
    │      │                  │  CordovaBridge.java
    │      │                  │  CordovaClientCertRequest.java
    │      │                  │  CordovaDialogsHelper.java
    │      │                  │  CordovaHttpAuthHandler.java
    │      │                  │  CordovaInterface.java
    │      │                  │  CordovaInterfaceImpl.java
    │      │                  │  CordovaPlugin.java
    │      │                  │  CordovaPreferences.java
    │      │                  │  CordovaResourceApi.java
    │      │                  │  CordovaWebView.java
    │      │                  │  CordovaWebViewEngine.java
    │      │                  │  CordovaWebViewImpl.java
    │      │                  │  CoreAndroid.java
    │      │                  │  ExposedJsApi.java
    │      │                  │  ICordovaClientCertRequest.java
    │      │                  │  ICordovaCookieManager.java
    │      │                  │  ICordovaHttpAuthHandler.java
    │      │                  │  LOG.java
    │      │                  │  NativeToJsMessageQueue.java
    │      │                  │  PluginEntry.java
    │      │                  │  PluginManager.java
    │      │                  │  PluginResult.java
    │      │                  │  Whitelist.java
    │      │                  │  
    │      │                  └─engine
    │      │                          SystemCookieManager.java
    │      │                          SystemExposedJsApi.java
    │      │                          SystemWebChromeClient.java
    │      │                          SystemWebView.java
    │      │                          SystemWebViewClient.java
    │      │                          SystemWebViewEngine.java
    │      │                          
    │      ├─libs
    │      ├─platform_www
    │      │  │  cordova.js
    │      │  │  
    │      │  └─cordova-js-src
    │      │      │  exec.js
    │      │      │  platform.js
    │      │      │  
    │      │      ├─android
    │      │      │      nativeapiprovider.js
    │      │      │      promptbasednativeapi.js
    │      │      │      
    │      │      └─plugin
    │      │          └─android
    │      │                  app.js
    │      │                  
    │      ├─res
    │      │  ├─drawable-* android工程的icon和图片资源，省略
    │      │  │      
    │      │  ├─values
    │      │  │      strings.xml
    │      │  │      
    │      │  └─xml
    │      │          config.xml
    │      │          
    │      └─src
    │          ├─io
    │          │  └─cordova
    │          │      └─hellocordova
    │          │              MainActivity.java
    │          │              
    │          └─org
    │              └─apache
    │                  └─cordova
    │                      └─whitelist
    │                              WhitelistPlugin.java
    │                              
    ├─plugins
    │  │  android.json
    │  │  fetch.json
    │  │  
    │  └─cordova-plugin-whitelist
    │      │  CONTRIBUTING.md
    │      │  LICENSE
    │      │  NOTICE
    │      │  package.json
    │      │  plugin.xml
    │      │  README.md
    │      │  RELEASENOTES.md
    │      │  whitelist.js
    │      │  
    │      ├─doc
    │      │  ├ 白名单插件的文档说明，省略
    │      │          
    │      └─src
    │          └─android
    │                  WhitelistPlugin.java
    │                  
    └─www
        │  前端代码没有变化就不列了


其实就是下了一个android工程的模板，然后又下了一个白名单插件，安装到android工程中  
在这个android工程中，  
可以看到assets下有前端代码，其中包括根目录下的代码，cordova的js框架，白名单插件对应的js代码，  
还有cordova、CordovaLib、platform_www3个文件夹  
cordova下都是针对android平台较底层的命令行工具  
CordovaLib是应用依赖的cordova框架的android实现lib  
platform_www是android平台的cordova js框架实现  
而src目录下，主要就是运行前端代码的MainActivity和白名单插件对应的native实现  
另外就是 会在res/xml下生成一个android平台的config.xml, 方便android平台解析配置  

## 添加个插件
以cordova-plugin-device插件为例
执行

    cordova plugin add cordova-plugin-device

之后，目录结构新增变化如下

    │  
    ├─platforms
    │  │  
    │  └─android
    │      │  
    │      ├─assets
    │      │  └─www
    │      │      │      
    │      │      └─plugins
    │      │          ├─cordova-plugin-device
    │      │          │  └─www
    │      │          │          device.js
    │      │          
    │      └─src
    │          ├─io
    │          │  └─cordova
    │          │      └─hellocordova
    │          │              MainActivity.java
    │          │              
    │          └─org
    │              └─apache
    │                  └─cordova
    │                      ├─device
    │                      │      Device.java
    │                      │      
    │                      └─whitelist
    │                              WhitelistPlugin.java
    │                              
    ├─plugins
    │  │  android.json
    │  │  fetch.json
    │  │  
    │  ├─cordova-plugin-device
    │  │  │  CONTRIBUTING.md
    │  │  │  LICENSE
    │  │  │  NOTICE
    │  │  │  package.json
    │  │  │  plugin.xml
    │  │  │  README.md
    │  │  │  RELEASENOTES.md
    │  │  │  
    │  │  ├─doc
    │  │  │  ├─插件文档 省略
    │  │  │          
    │  │  ├─src
    │  │  │  ├─android
    │  │  │  │      Device.java
    │  │  │  │      
    │  │  │  ├─blackberry10
    │  │  │  │      index.js
    │  │  │  │      
    │  │  │  ├─browser
    │  │  │  │      DeviceProxy.js
    │  │  │  │      
    │  │  │  ├─firefoxos
    │  │  │  │      DeviceProxy.js
    │  │  │  │      
    │  │  │  ├─ios
    │  │  │  │      CDVDevice.h
    │  │  │  │      CDVDevice.m
    │  │  │  │      
    │  │  │  ├─tizen
    │  │  │  │      DeviceProxy.js
    │  │  │  │      
    │  │  │  ├─ubuntu
    │  │  │  │      device.cpp
    │  │  │  │      device.h
    │  │  │  │      device.js
    │  │  │  │      
    │  │  │  ├─windows
    │  │  │  │      DeviceProxy.js
    │  │  │  │      
    │  │  │  └─wp
    │  │  │          Device.cs
    │  │  │          
    │  │  ├─tests
    │  │  │      plugin.xml
    │  │  │      tests.js
    │  │  │      
    │  │  └─www
    │  │          device.js
    
首先 plugins目录下，增加了cordova-plugin-device的代码  
android工程中，assets下 增加了device.js 插件的js代码  
src下增加了 插件对应的android实现。  
config.xml中的也添加了device插件的信息  

从plugins目录下，可以看下插件的代码的目录结构  
插件主要包括3部分  

- plugin.xml 插件的配置文件
- src文件夹，主要包括此插件在各平台的实现
- www/device.js 插件对应的js接口代码

从上面的目录结构也可以看出当安装某个插件后，  
插件的各平台实现代码就转移到各平台工程中的源代码目录中  
插件的js代码就转移到各平台存放前端代码的地方  
而这些转移操作一方面由cordova脚本控制，一方面由插件的配置文件plugin.xml控制  
另外plugin.xml中的配置信息也会整合到各平台的config.xml中。  

安装插件后，前端开发者就可以在代码中调用插件提供的js接口了，具体调用方式可以查看插件的说明  

## 针对某平台开发
针对某平台开发意思是说放弃cordova的跨平台特性，用html、css、js等技术开发某特定平台的应用。  
这种情况下，上面说到的cordova工程目录就有些冗余了。  
但是，官方对于针对某平台开发的工程创建，是这样建议的  

1. 先创建跨平台的工程（上面说的目录结构）  
2. 添加要针对的平台（比如说android）  
3. 此时 platforms/android 下的工程，就可以作为 针对android平台的工程目录  

相关的操作命令也在platforms/android/cordova/下  
另外 还有一种方式 就是利用某平台的工具直接生成相应平台的工程  
比如[Android平台的](https://github.com/apache/cordova-android)  
具体命令请参考[各平台命令行工具说明](http://cordova.apache.org/docs/en/latest/guide/platforms/index.html)  

所以 针对某平台的工程目录结构和跨平台中某平台的工程目录结构是相似的  

插件的安装工具也与跨平台的有所区别，是[plugman](https://www.npmjs.com/package/plugman)  
安装工具  

    cnpm install -g plugman

安装插件  

    plugman install --platform <platform> --project <directory> --plugin <plugin>

platform指定平台，project指定此平台的工程目录，plugin指定插件  

## 最后
至此，对cordova的目录结构应该有个大致了解。  
但是上面说的不是绝对的，比如插件的目录结构可能由于插件的功能（仅需要js或者native代码）不同，而有所缺失。  
又或者需要一些钩子脚本，而多出脚本目录。  