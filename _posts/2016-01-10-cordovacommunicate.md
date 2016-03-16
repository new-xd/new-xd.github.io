---
layout:     post
title:      "Cordova通信接口"
subtitle:   "Cordova js native通信接口简介"
date:       2016-01-10 14:06:00+0800
author:     "New"
header-img: ""
tags:
    - cordova
---

由于cordova的目标是用前端代码开发，
所以js-native之间的通信在设计上都是由js调用native，
native处理完后，返回处理结构。

## js如何调用native
在cordova-plugin-device的device.js中，获取设备信息的方法如下

    exec = require('cordova/exec'),
    Device.prototype.getInfo = function(successCallback, errorCallback) {
        exec(successCallback, errorCallback, "Device", "getDeviceInfo", []);
    };
    
核心就是exec方法

- successCallback 是 请求成功的回调
- errorCallback 是 请求失败的回调
- "Device" 是 native的服务名，这个服务名会对应于一个native的插件实现
    - 就是插件配置信息中的feature的名字
- "getDeviceInfo" 是 native服务提供的方法
- [] 中应该参入此方法的参数

所有的js调用native方法都是通过此接口

## native如何响应js的调用
从js的接口可以看出，native应该实现一个“服务”Device，这个“服务”应该有个“方法”getDeviceInfo,接受参数为空。

但是因为平台不同，每个平台的具体实现也都不想同，所以“服务”和“方法”的具体实现形式都可能不同。
这里我只说Android

在Android平台，“服务”对应一个CordovaPlugin类，js调用最终会调用到此类的execute方法

    public boolean execute(String action, JSONArray args, CallbackContext callbackContext);

而js中的“方法”就是参数中的action，js中对应的参数为args，callbackContext负责成功失败回调
所以cordova-plugin-device的native实现Device.java大概如下

    public class Device extends CordovaPlugin {

        // 省略

        /**
        * Executes the request and returns PluginResult.
        *
        * @param action            The action to execute.
        * @param args              JSONArry of arguments for the plugin.
        * @param callbackContext   The callback id used when calling back into JavaScript.
        * @return                  True if the action was valid, false if not.
        */
        public boolean execute(String action, JSONArray args, CallbackContext callbackContext) throws JSONException {
            if ("getDeviceInfo".equals(action)) {
                JSONObject r = new JSONObject();
                r.put("uuid", Device.uuid);
                r.put("version", this.getOSVersion());
                r.put("platform", this.getPlatform());
                r.put("model", this.getModel());
                r.put("manufacturer", this.getManufacturer());
                r.put("isVirtual", this.isVirtual());
                r.put("serial", this.getSerialNumber());
                callbackContext.success(r);
            }
            else {
                return false;
            }
            return true;
        }
        
        // 省略
    }

可以看到调用的“方法”是通过

    "getDeviceInfo".equals(action)

判断确定的，而“服务”又是如何对应到这个Device类的呢？

回头再看cordova-plugin-device的插件配置文件有这么一段

    <platform name="android">
        <config-file target="res/xml/config.xml" parent="/*">
            <feature name="Device" >
                <param name="android-package" value="org.apache.cordova.device.Device"/>
            </feature>
        </config-file>

        <source-file src="src/android/Device.java" target-dir="src/org/apache/cordova/device" />
    </platform>

在上篇插件配置的博客中，我只说了config-file是对xml文件新增加节点，source-file是copy文件
但是没有对增加的什么，拷贝的什么进行分析，实际上这些操作都是为了保证js的“服务”对应到android的Device类

首先 在config.xml加入feature
name 就是 js调用的“服务”名
android-package 就是指 此“服务”在android平台对应的类
然后 将插件中的Device.java 拷贝到对应的包下

这样在android平台，应用启动时，就可以通过config.xml，得知有一个“服务”为Device,具体实现为org.apache.cordova.device.Device
当js调用时 通过class反映射出Device类的实例，调用execute方法，将“方法” 参数 回调传入。
这样native就可以响应js的调用了

## native如何将结果返回给js
通过上面我们知道是通过CallbackContext将native的处理结果返回给js
其接口如下

    public class CallbackContext {
        public void sendPluginResult(PluginResult pluginResult);
        public void success(JSONObject message);
        public void success(String message);
        public void success(JSONArray message);
        public void success(byte[] message);
        public void success(int message);
        public void success();
        public void error(JSONObject message);
        public void error(String message);
        public void error(int message);
    }

sendPluginResult就是向js发送结果
而success和error都是对sendPluginResult的简单封装
