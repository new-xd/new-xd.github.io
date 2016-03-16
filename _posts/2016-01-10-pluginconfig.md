---
layout:     post
title:      "Cordova Plugin 配置"
subtitle:   "让我们看看插件的配置规范"
date:       2016-01-10 10:30:00+0800
author:     "New"
header-img: ""
tags:
    - cordova
---

关于plugin的配置，没有说明的请参考[官方文档](http://cordova.apache.org/docs/en/latest/plugin_ref/spec.html)

## 插件结构
在前面已经提过，插件主要由

- plugin.xml
- src 各平台实现
- www/\*.js js实现

组成， 另外可能还有

- scripts/\*.js 插件需要的钩子脚本

## plugin.xml
以cordova-plugin-device的plugin.xml为例，省略部分平台如下

    <?xml version="1.0" encoding="UTF-8"?>
    <plugin xmlns="http://apache.org/cordova/ns/plugins/1.0"
        xmlns:rim="http://www.blackberry.com/ns/widgets"
        xmlns:android="http://schemas.android.com/apk/res/android"
        id="cordova-plugin-device"
        version="1.1.0">
        <name>Device</name>
        <description>Cordova Device Plugin</description>
        <license>Apache 2.0</license>
        <keywords>cordova,device</keywords>
        <repo>https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git</repo>
        <issue>https://issues.apache.org/jira/browse/CB/component/12320648</issue>

        <js-module src="www/device.js" name="device">
            <clobbers target="device" />
        </js-module>

        <!-- android -->
        <platform name="android">
            <config-file target="res/xml/config.xml" parent="/*">
                <feature name="Device" >
                    <param name="android-package" value="org.apache.cordova.device.Device"/>
                </feature>
            </config-file>

            <source-file src="src/android/Device.java" target-dir="src/org/apache/cordova/device" />
        </platform>

    </plugin>

根节点的id是插件ID，就是命令行安装插件时使用的插件ID
version是插件版本
后面的一些标签是对插件的描述。
重点来了

#### js-module
js-module 是 对插件js代码的配置
首先，插件的js代码都是模块化的，一个文件就是一个js模块
可以说一个js-module配置了一个js模块
src 是指此模块的源代码，
name 是指此模块的名字
target 是指此模块对外的接口
实际上 src是此js模块的实现，在插件源代码中是 没有封装成模块的
当插件安装之后，插件在各平台的js文件会在源文件的基础上加上模块定义
比如 device.js 原来是这样

    /*
    license 省略
    */

    var argscheck = require('cordova/argscheck'),
    ...
    省略js代码
    ...
    module.exports = new Device();

安装到android平台后，就加上了模块定义

    cordova.define("cordova-plugin-device.device", function(require, exports, module) { /*
    license 省略
    */

    var argscheck = require('cordova/argscheck'),
    ...
    省略js代码
    ...
    module.exports = new Device();

    });

cordova.define 是cordova提供的模块定义方法。
cordova-plugin-device.device 就是模块名由插件ID+模块name（js-module中的name）组成
而在js要调用此模块的方法 就是 通过模块对外的接口device（js-module中的target）调用
这就是clobbers标签的作用

>    Three tags are allowed within \<js-module\>:

>    \<clobbers target="some.value"/\> indicates that the module.exports is inserted into the window object as window.some.value. You can have as many \<clobbers\> as you like. Any object not available on window is created.

>    \<merges target="some.value"/\> indicates that the module should be merged with any existing value at window.some.value. If any key already exists, the module's version overrides the original. You can have as many \<merges\> as you like. Any object not available on window is created.

>    \<runs/\> means that your code should be specified with cordova.require, but not installed on the window object. This is useful when initializing the module, attaching event handlers or otherwise. You can only have up to one \<runs/\> tag. Note that including a \<runs/\> with \<clobbers/\> or \<merges/\> is redundant, since they also cordova.require your module.

>    An empty \<js-module\> still loads and can be accessed in other modules via cordova.require.

js-module中可以有3种标签

- clobbers target=some.value 说明这个模块会被以 window.some.value的形式 插入到window对象，可以有多个clobbers标签，window上不存的对象，会被创建。
- merges target=some.value 说明这个模块对外的接口 会被 合并到 window.some.value上，如果存在的接口会被覆盖，可有有多个标签，不存在的对象会被创建
- runs 意味着你的js需要作为模块执行一次，但是不需要在window对象上暴露对外接口。这个用于初始化模块、绑定监听器比较有用。只能有一个runs标签。注意和clobbers或者merges公用是多余的，因为这两个标签都会执行一次模块代码
- 空的js-module仍然会被加载，而且可以在其他模块中用cordova.require调用


#### 各平台的插件实现
以device在android平台的配置为例

##### config-file
config-file表示插件对android平台xml文件的添加标签节点
target是要修改的xml文件，为相对于平台工程根目录的相对路径
parent表示在哪个节点下添加标签，/\*表示根节点下
config-file内的标签就是添加到目标文件的内容

举个例子，在android平台下，在AndroidManifest中添加activity定义

		<config-file parent="/manifest/application" target="AndroidManifest.xml">
			<activity android:name="com.your.package.YourActivity"/>
		</config-file>

目前还没有发现如何修改已有标签，只能新添

对于插件，需要对各平台的config.xml添加插件信息，如下

        <config-file target="res/xml/config.xml" parent="/*">
            <feature name="Device" >
                <param name="android-package" value="org.apache.cordova.device.Device"/>
            </feature>
        </config-file>

这样就会在Android平台代码中，对res/xml/config.xml文件的根节点下插入

        <feature name="Device" >
            <param name="android-package" value="org.apache.cordova.device.Device"/>
        </feature>

这个就是插件的配置信息

feature代表此插件提供的一个功能模块，name是此功能模块的命名，
android-package是此功能模块对应的android实现。

cordova在初始化过程中，通过解析feature就可以知道该平台都有哪些功能模块，分别对应的实现是谁。

##### source-file
source-file表示插件android平台的实现如何放入android工程，比较简单
src是源代码路径，相对于插件源代码根目录
target为安装目录，相对于平台工程根目录

实际上source-file不仅用于安装源代码，你可以把它看成拷贝配置
把需要的文件（代码、图片、jar、其他资源）从插件目录 拷贝到 平台工程目录中

##### framework
另外android平台还有一个可能用到的标签 framework

    <!-- Depend on latest version of GCM from play services -->
    <framework src="com.google.android.gms:play-services-gcm:+" />
    <!-- Depend on v21 of appcompat-v7 support library -->
    <framework src="com.android.support:appcompat-v7:21+" />
    <!-- Depend on library project included in plugin -->
    <framework src="relative/path/FeedbackLib" custom="true" />
    <!-- 设置引用的lib工程之间的引用关系，我自己未测试 -->
    <framework src="extras/android/support/v7/appcompat" custom="false" parent="FeedbackLib" />

主要用于说明插件源代码依赖的库，src说明依赖的库，如果是本地路径，就是相对于插件根目录。
详情可以看[文档](http://cordova.apache.org/docs/en/latest/plugin_ref/spec.html)中 framework for Android部分

各平台的配置 肯定因为平台不同而不同，所以会有一些平台特有的标签，而我这里也是只针对Android讲了一下，
其他平台的 还请看下[官方文档](http://cordova.apache.org/docs/en/latest/plugin_ref/spec.html)

#### 插件配置文件中还会用到的一些标签

首先是engines和engine标签，表示此插件支持的环境

    <engines>
        <engine name="cordova" version=">=1.7.0" />
        <engine name="cordova-android" version=">=1.8.0" />
        <engine name="cordova-ios" version=">=1.7.1" />
    </engines>

然后是 asset标签，表示需要安装到平台www下的静态文件

    <!-- a single file, to be copied in the root directory -->
    <asset src="www/foo.js" target="foo.js" />
    <!-- a directory, also to be copied in the root directory -->
    <asset src="www/foo" target="foo" />

之后是 dependency标签，表示此插件依赖的插件

    <dependency id="com.plugin.id" url="https://github.com/myuser/someplugin" commit="428931ada3891801" subdir="some/path/here" />

id为依赖的插件ID，url为插件的git地址，commit表示插件的git版本，subdir表示依赖的插件在此git下的相对路径

还有 hook标签，表示此插件需要的一些的钩子脚本

    <hook type="after_plugin_install" src="scripts/afterPluginInstall.js" />

对于钩子脚本，以后再开新篇介绍，其实也简单，上面这个标签的意思是说 在插件安装之后，要执行脚本scripts/afterPluginInstall.js
