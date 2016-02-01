---
layout:     post
title:      "Cordova android启动过程浏览"
subtitle:   "其实那么复杂"
date:       2016-01-31 20:40:00+0800
author:     "New"
header-img: ""
tags:
    - cordova
---

cordova在android平台 其实主要就是启动CordovaActivity，然后构建一个WebView加载前端应用。
先从CordovaActivity开始

## CordovaActivity

主要过程在onCreate中， 这个类是父类，在使用中都是继承它，注释中推荐的子类写法如下

    /**
    *     package org.apache.cordova.examples;
    *
    *     import android.os.Bundle;
    *     import org.apache.cordova.*;
    *
    *     public class Example extends CordovaActivity {
    *       @Override
    *       public void onCreate(Bundle savedInstanceState) {
    *         super.onCreate(savedInstanceState);
    *         super.init();
    *         // Load your application
    *         loadUrl(launchUrl);
    *       }
    *     }
    */

可以看到 先调用CordovaActivity的onCreate 然后 init 最后loadUrl
其实不init也可以，因为loadUrl对是否init有判断，没有init时就会调用init

所以用cordova命令生成的代码是直接调用loadUrl

下面我按照先init在loadUrl的过程讲下主要流程，如下
每缩进一次 代表一层函数调用

    super.onCreate(savedInstanceState); // 子类中调用CordovaActivity的onCreate
        loadConfig(); // 解析config.xml
            // 1. ConfigXmlParser解析
            // 2. 获取preference参数
            // 3. 获取launchUrl
            // 4. 获取插件配置
        // 省略一些对全屏的设置
        super.onCreate(savedInstanceState); // 调用Activity的onCreate
        makeCordovaInterface(); // 创建cordova 接口
            // 生成 CordovaInterfaceImpl
    super.init(); // 初始化cordova环境
        appView = makeWebView(); // 生成CordovaWebView接口的实例
        createViews(); // 主要将webview添加到布局上
            setContentView();
        appView.init(cordovaInterface, pluginEntries, preferences); // 初始化CordovaWebView
    loadUrl(launchUrl); // 加载应用
        // 这个里面有一些对超时、加载出错的处理，但是最终还是调用WebView的loadUrl

简单来说 就是

1. 解析config.xml
2. 创建CordovaInterface
3. 创建CordovaWebView
4. 初始化CordovaWebVIew
5. 加载launchUrl

## config.xml 解析

这里主要就是调用ConfigXmlParser的解析
preferences是属性配置， 在设置全屏或者初始化webview或者初始化插件时都会用到
launchUrl就是默认的启动页面了
插件配置 就是 pluginEntry数组，说明此应用用了哪些插件，包括插件的service名 native类名 是否启动加载

## CordovaInterface

这个接口定义如下

    /**
    * The Activity interface that is implemented by CordovaActivity.
    * It is used to isolate plugin development, and remove dependency on entire Cordova library.
    */
    public interface CordovaInterface {

        /**
        * Launch an activity for which you would like a result when it finished. When this activity exits,
        * your onActivityResult() method will be called.
        *
        * @param command     The command object
        * @param intent      The intent to start
        * @param requestCode   The request code that is passed to callback to identify the activity
        */
        abstract public void startActivityForResult(CordovaPlugin command, Intent intent, int requestCode);

        /**
        * Set the plugin to be called when a sub-activity exits.
        *
        * @param plugin      The plugin on which onActivityResult is to be called
        */
        abstract public void setActivityResultCallback(CordovaPlugin plugin);

        /**
        * Get the Android activity.
        *
        * @return the Activity
        */
        public abstract Activity getActivity();
        

        /**
        * Called when a message is sent to plugin.
        *
        * @param id            The message id
        * @param data          The message data
        * @return              Object or null
        */
        public Object onMessage(String id, Object data);
        
        /**
        * Returns a shared thread pool that can be used for background tasks.
        */
        public ExecutorService getThreadPool();
    }

此接口主要是给插件用的，解耦插件和Activity的联系
主要接口是
startActivityForResult 为插件提供启动activity并获取返回值的方法
getActivity 为插件提供获取webview所运行的Activity
getThreadPool 为插件的具体操作提供线程池 避免阻塞UI线程

而onMessage是给Activity处理各种消息的接口，这里的消息主要是发给插件的，但是如果没有插件处理，则会调用此接口让Activity处理
具体实现是CordovaInterfaceImpl类，实现中onMessage就处理了exit消息，调用finish退出activity

## CordovaWebView

注意 这是一个接口， 主要对Activity封装了webview的各种操作，包括内部的插件机制 js-native通信机制 webview的优化设置。
在实现类CordovaWebViewImpl中，对webview的操作又被封装到了CordovaWebViewEngine接口，
webview操作的相关类图如下

![webview操作相关的类图](/img/cordova-start/CordovaWebView1.png)

插件管理和js-native通信的类图如下

![插件管理和js-native通信的类图](/img/cordova-start/CordovaWebView2.png)

### 创建CordovaWebView

大致过程如下

    makeWebView()
        new CordovaWebViewImpl(makeWebViewEngine())
            CordovaWebViewImpl.createEngine(this, preferences)
                (CordovaWebViewEngine) constructor.newInstance(context, preferences) // 这里的constructor是SystemWebViewEngine的构造函数，这里用了反射
                    new SystemWebView(context)

所以创建过程是

1. new SystemWebView
2. new SystemWebViewEngine
3. new CordovaWebViewImpl

其中除了给对象传递属性赋值外，没有做其他事情
之后对于webview的各种操作， 都可以 通过CordovaWebView -> CordovaWebViewImpl -> SystemWebViewEngine -> SystemWebView进行

### 初始化CordovaWebView

关于Cordova在Android上的通信机制可以看[这里](/2016/01/14/cordovacommunicateanalyze/)

CordovaWebView初始化的大致过程如下

    appView.init(cordovaInterface, pluginEntries, preferences);
        pluginManager = new PluginManager(this, this.cordova, pluginEntries);
        nativeToJsMessageQueue = new NativeToJsMessageQueue();
        nativeToJsMessageQueue.addBridgeMode(new NativeToJsMessageQueue.NoOpBridgeMode()); //添加js轮询模式
        nativeToJsMessageQueue.addBridgeMode(new NativeToJsMessageQueue.LoadUrlBridgeMode(engine, cordova)); //添加loadUrl模式
        engine.init(this, cordova, engineClient, resourceApi, pluginManager, nativeToJsMessageQueue);
            webView.init(this, cordova); // webview 设置SystemWebViewClient SystemWebChromeClient
            initWebViewSettings(); // 对webview的各种设置 主要为了优化
            nativeToJsMessageQueue.addBridgeMode(new NativeToJsMessageQueue.OnlineEventsBridgeMode(...)); // 添加online offline事件通信模式
            bridge = new CordovaBridge(pluginManager, nativeToJsMessageQueue); 
            exposeJsInterface(webView, bridge);
        pluginManager.addService(CoreAndroid.PLUGIN_NAME, "org.apache.cordova.CoreAndroid");
        pluginManager.init();
            this.startupPlugins(); // 对于onload为true的插件加载初始化

初始化过程做了很多操作，按功能可以简单分为

- 插件初始化
- 通信通道设置
- nativeToJsMessageQueue 通信队列初始化
- webview设置

#### 插件初始化

PluginManager管理所有插件，在初始化中创建。
所有和插件相关的功能都会通过这个类处理，比如

- 向插件传递消息插件的 onMessage 接口
- js调用具体的插件 exec 方法
- Activity的生命周期 回调
- webview加载url的相关事件传递

需要注意下webview加载页面或者请求资源时会调用PluginManager相关的方法，判断应用是否有权限
就是判断访问的地址是否在白名单内，如果卸载了白名单插件或者config中没有设置，那么就不能请求

config中

- allow-navigation 是 添加页面的白名单（应用内的页面，在CordovaWebView中加载）
- access 是 资源请求的白名单
- allow-intent 是 添加外部链接的白名单（外部链接会通过浏览器或者其他Activity展现）


PluginManager有点像钩子程序管理者，发生了任何事都要执行相关的所有钩子程序（插件的相关方法）

#### 通信通道设置

通信的通道主要有两个：exposeJsInterface 和 SystemWebChromeClient

在exposeJsInterface方法中，调用

    webView.addJavascriptInterface(exposedJsApi, "_cordovaNative");

向js添加了一个全局变量_cordovaNative， 对应的native接口就是ExposedJsApi接口的实现者SystemExposedJsApi

第二个通道是SystemWebChromeClient中的onJsPrompt方法

这两个通道最终都是将消息交给CordovaBridge做统一处理, 处理之后再返回给js

#### nativeToJsMessageQueue 通信队列初始化

nativeToJsMessageQueue 主要用来存储native要发给js的消息，等待js拿走
插件中exec处理之后的结果即PluginResult最后都会放到nativeToJsMessageQueue

#### webview设置

都在initWebViewSettings()中，主要就是对WebSettings的设置
有兴趣的可以仔细看下，对于自己写webview加载页面应该会有一些帮助


## 加载url

环境初始化好之后，就可以加载url了，这里做了一些超时错误的处理，没有意外页面就可以显示出来了

