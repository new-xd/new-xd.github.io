---
layout:     post
title:      "Cordova Android 通信解析"
subtitle:   "让我们探索Cordova的秘密"
date:       2016-01-14 21:50:00+0800
author:     "New"
header-img: ""
tags:
    - cordova
---

相信这是很多人关注的问题

那让我们来看看Android下的实现，下面的内容都是我自己的理解 难免有不对的地方，见谅

## 接口
js的接口如下

    function androidExec(success, fail, service, action, args)

就是在插件js中调用的exec

native的接口就是 CorodvaPlugin类

## 原理

js和native的通信方式

#### 添加js对象

WebView有方法[addJavascriptInterface](http://developer.android.com/reference/android/webkit/WebView.html#addJavascriptInterface(java.lang.Object, java.lang.String))
通过此方法，可以建立一个js和java共同的对象，js可以直接调用java对象中的方法，从而建立通信通道

#### js的prompt实现

js的[prompt](http://www.w3school.com.cn/jsref/met_win_prompt.asp)方法，用于显示用户可输入的提示框
而这个方法对应的native实现就是[WebChromeClient的onJsPrompt](http://developer.android.com/reference/android/webkit/WebChromeClient.html#onJsPrompt(android.webkit.WebView, java.lang.String, java.lang.String, java.lang.String, android.webkit.JsPromptResult))方法
可以利用prompt，从js传递私有协议，native过滤，从而达到js和native通信

#### loadUrl

WebView的loadUrl方法， 如果url格式为 "javascript:" + js代码， 则是执行js代码。
通过此方法，native可以直接调用js函数，从而通信。
而且这是这三个方式中唯一一个native主动调js的方式
但是cordova虽然实现这个通信方式，却没有使用，代码中的注释如下

    // For LOAD_URL to be viable, it would need to have a work-around for
    // the bug where the soft-keyboard gets dismissed when a message is sent.

    要使用loadRul, 必须解决发消息时软键盘会隐藏的bug。

可能还没有好的bug解决方案，所以这种方式没有被使用

## 设计

对于混合开发，从根本上说，js是上层，应用主体，接口的调用者，native是下层，应用某一功能实现，接口的实现者。
所以都是js调用native。
对于通信方法，前两种也都是js调用native。
所以js和native交互的核心方法，也都是 js调用native。
如下
native的接口

    public interface ExposedJsApi {
        // js 调用 native 执行命令
        public String exec(int bridgeSecret, String service, String action, String callbackId, String arguments) throws JSONException, IllegalAccessException;
        // 改变 native 传递 消息给 js的方式
        public void setNativeToJsBridgeMode(int bridgeSecret, int value) throws IllegalAccessException;
        // 获取 native要传递给js的消息
        public String retrieveJsMessages(int bridgeSecret, boolean fromOnlineEvent) throws IllegalAccessException;
    }

这个接口是第一种方式共同对象要实现的方法
native prompt方式也有类似的三个方法 jsExec jsSetNativeToJsBridgeMode jsRetrieveJsMessages
js侧对应的方法可以参考nativeapiprovider模块

交互流程大致如下

- js侧 调用exec
- js侧 生成 callbackID，存储回调函数
- js侧 通过第一种或者第二种方式 调用exec或者prompt（jsExec）将消息传递到native侧
- native侧 解析消息，调用相应的插件执行（大多是异步执行）
- native侧 执行结束，将结果转化为消息放入消息队列，通知js侧
- js侧 收到通知，调用retrieveJsMessages或者prompt（jsRetrieveJsMessages），将消息队列的消息取走
- js侧 解析出结果和callbackID，找到对应回调
- js侧 调用相应的回调，结束

至于native是如何通知js的，就是通过setNativeToJsBridgeMode或者jsSetNativeToJsBridgeMode设置的
有3种

- 不通知， js轮询
- online事件
- loadUrl，不过没有使用

第一种就不说了，
第二种是通过WebView的[setNetworkAvailable](http://developer.android.com/reference/android/webkit/WebView.html#setNetworkAvailable(boolean))方法
通知webview网络状态改变，
而js就会收到online事件，通过监听online事件，得知native有消息要传递
js侧要监听online事件，需要做些修改

    // For the ONLINE_EVENT to be viable, it would need to intercept all event
    // listeners (both through addEventListener and window.ononline) as well
    // as set the navigator property itself.

这是对这种方式的注释，具体要做的修改就没有再仔细研究了

## 最后

cordova在android上的通信机制就到这了，没有涉及过多的代码，想看具体实现的，可以看下源码
另外 还有就是通信建立的过程，这个包含在cordova启动的过程中，以后再说吧。





