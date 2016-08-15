---
layout:     post
title:      "React Native尝试"
subtitle:   " \"Hello React Native\""
date:       2016-08-04 0:09:00+0800
author:     "New"
header-img: ""
tags:
    - react native
---
基于React Native docs 0.30

# install
1. 安装node (安装最新版)
2. 安装Python2 
3. npm install -g react-native-cli
4. 安装 Android编译环境

# 创建项目

    react-native init ProjectName
    cd ProjectName
    react-native run-android

添加了一些新安装的npm包之后， 需要重新 npm install，然后再 react-native run-android

编译前端资源代码，生成native编译需要的资源，命令如下

    react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res/

之后就可以用native的方式编译应用，以上是android，ios以后再补，类似

命令行编译原生的命令（android）

    cd android&&gradlew assembleRelease

输出日志：

    adb logcat *:S ReactNative:V ReactNativeJS:V

# 已有项目环境创建

已有的项目不会将node\_modules进行代码管理，所以一般checkout下来都是没有node\_modules的

1. npm install 重新安装node_modules
2. react-native run-android 

# 部分问题

## reload js无效
有些时候 reload js不好用，总是要重启packager，请确认你的node python2 react react-native 都是最新版

## no propType for native prop 之类的错误
更新react-native的版本之后，记得react-native upgrade升级react-native自带的应用模板代码，不然也可能出现莫名其妙的错误

## native开发时总是报网络错误
进行Android开发的时候，在只修改native代码的情况下，建议把模板Application中 return BuildConfig.DEBUG 改为 return false，
避免总是尝试连接packager服务器

## ios跳转native界面非常卡
进行ios开发的时候native module的导出的方法不是在主线程中操作的，如果要做跳转界面，需要切换到主线程，不然特别慢，
另外就是默认的rootController不支持跳转界面的，需要修改为可以跳转界面的controller

## 文字背景设置
ios上 Text 默认会有白色背景，而android上 Text默认是透明背景  
color的格式如下

- '#f0f' (#rgb)
- '#f0fc' (#rgba)
- '#ff00ff' (#rrggbb)
- '#ff00ff00' (#rrggbbaa)
- 'rgb(255, 255, 255)'
- 'rgba(255, 255, 255, 1.0)'
- 'hsl(360, 100%, 100%)'
- 'hsla(360, 100%, 100%, 1.0)'
- 'transparent'
- 'red'
其中alpha值 是在后面，这点与android上的写法略有不同

## ios的状态栏设置
ios默认是占满全屏的，这样会让状态栏和界面重叠在一起，如果不是要这种效果需要重新处理一下


# 基础知识

## 应用基础

index.android.js index.ios.js 分别是 android和ios的程序入口


    import React, { Component } from 'react';
    import { AppRegistry, Text } from 'react-native';

    class HelloWorldApp extends Component {
        render() {
            return (
            <Text>Hello world!</Text>
            );
        }
    }

    AppRegistry.registerComponent('HelloWorldApp', () => HelloWorldApp);

AppRegistry用来注册应用的第一个Component，所有的界面元素都可以封装为Component。  
Component可以组合，形成树形的Component组织结构，从而构建复杂的界面

import from class extends () => 等都是ES6的语法，react-native自带ES6支持.

    <Text>Hello world!</Text>

为JSX语法，混合js和html，<>内大写的元素是Component，小写是html标签

## props

Component创建的时候可以带参数，称为props

    import React, { Component } from 'react';
    import { AppRegistry, Image } from 'react-native';

    class Bananas extends Component {
        render() {
            let pic = {
                uri: 'https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg'
            };
            return (
                <Image source={pic} style={{width: 193, height: 110}}/>
            );
        }
    }

    AppRegistry.registerComponent('Bananas', () => Bananas);

比如Image的source为props，JXS中的第一个{}表示转义，表示内部是js代码，source={pic}表示将pic赋值给source，这样source就有一个属性uri  
在Image Component内部可以通过 this.props.source.uri 来获取uri的值

## state
控制Component的状态的数据有两种，一种是上面的props， 还有一种就是state。  
props是父Component设置的，在Component的生命周期中是（相对）固定的（可以被父Component修改）。  
state是可以改变的，代表Component可以变化的状态。  

一般在构造函数中初始化state，通过setState修改状态


    import React, { Component } from 'react';
    import { AppRegistry, Text, View } from 'react-native';

    class Blink extends Component {
        constructor(props) {
            super(props);
            this.state = {showText: true};

            // Toggle the state every second
            setInterval(() => {
                this.setState({ showText: !this.state.showText });
            }, 1000);
        }

        render() {
            let display = this.state.showText ? this.props.text : ' ';
            return (
                <Text>{display}</Text>
            );
        }
    }

    class BlinkApp extends Component {
        render() {
            return (
                <View>
                    <Blink text='I love to blink' />
                    <Blink text='Yes blinking is so great' />
                    <Blink text='Why did they ever take this out of HTML' />
                    <Blink text='Look at me look at me look at me' />
                </View>
            );
        }
    }

    AppRegistry.registerComponent('BlinkApp', () => BlinkApp);

注意 state不能直接修改，而是通过setState传入改变的属性，来修改的。

## style

    import React, { Component } from 'react';
    import { AppRegistry, StyleSheet, Text, View } from 'react-native';

    class LotsOfStyles extends Component {
    render() {
        return (
        <View>
            <Text style={styles.red}>just red</Text>
            <Text style={styles.bigblue}>just bigblue</Text>
            <Text style={[styles.bigblue, styles.red]}>bigblue, then red</Text>
            <Text style={[styles.red, styles.bigblue]}>red, then bigblue</Text>
        </View>
        );
    }
    }

    const styles = StyleSheet.create({
    bigblue: {
        color: 'blue',
        fontWeight: 'bold',
        fontSize: 30,
    },
    red: {
        color: 'red',
    },
    });

    AppRegistry.registerComponent('LotsOfStyles', () => LotsOfStyles);

所有的核心Component有一个style props，用来表示Component的style

布局相关的设置 参见[这里](http://facebook.github.io/react-native/docs/layout-props.html)

## 处理用户对Component的操作

    import React, { Component } from 'react';
    import { AppRegistry, Text, TextInput, View } from 'react-native';

    class PizzaTranslator extends Component {
      constructor(props) {
        super(props);
        this.state = {text: ''};
      }

      render() {
        return (
          <View style={{padding: 10}}>
            <TextInput
              style={{height: 40}}
              placeholder="Type here to translate!"
              onChangeText={(text) => this.setState({text})}
            />
            <Text style={{padding: 10, fontSize: 42}}>
              {this.state.text.split(' ').map((word) => word && '🍕').join(' ')}
            </Text>
          </View>
        );
      }
    }

    AppRegistry.registerComponent('PizzaTranslator', () => PizzaTranslator);

上面这个例子处理了Component的文字变化,注意这里的props onChangeText是一个回调方法，不再是一个“属性值”

## ScrollView ListView
ScrollView可以滚动的Component

    import React, { Component } from 'react';
    import{ AppRegistry, ScrollView, Image, Text, View } from 'react-native'

    class IScrolledDownAndWhatHappenedNextShockedMe extends Component {
      render() {
          return(
            <ScrollView>
              <Text style={{fontSize:96}}>Scroll me plz</Text>
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Text style={{fontSize:96}}>If you like</Text>
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Text style={{fontSize:96}}>Scrolling down</Text>
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Text style={{fontSize:96}}>What's the best</Text>
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Text style={{fontSize:96}}>Framework around?</Text>
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Image source={require('./img/favicon.png')} />
              <Text style={{fontSize:80}}>React Native</Text>
            </ScrollView>
        );
      }
    }


    AppRegistry.registerComponent(
      'IScrolledDownAndWhatHappenedNextShockedMe',
      () => IScrolledDownAndWhatHappenedNextShockedMe);

ListView比ScrollView好的地方是，它不会渲染所有的子Component，限制的地方是只能是结构化的数据

    import React, { Component } from 'react';
    import { AppRegistry, ListView, Text, View } from 'react-native';

    class ListViewBasics extends Component {
      // Initialize the hardcoded data
      constructor(props) {
        super(props);
        const ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
        this.state = {
          dataSource: ds.cloneWithRows([
            'John', 'Joel', 'James', 'Jimmy', 'Jackson', 'Jillian', 'Julie', 'Devin'
          ])
        };
      }
      render() {
        return (
          <View style={{paddingTop: 22}}>
            <ListView
              dataSource={this.state.dataSource}
              renderRow={(rowData) => <Text>{rowData}</Text>}
            />
          </View>
        );
      }
    }

    // App registration and rendering
    AppRegistry.registerComponent('AwesomeProject', () => ListViewBasics);

ListView的 dataSource prop 是 ListView.DataSource类型， rowHasChanged必须有，表示有没有发生变化  
renderRow prop表示如何将数据转化为View

## 网络请求

[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)

    fetch('https://mywebsite.com/endpoint/', {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        firstParam: 'yourValue',
        secondParam: 'yourOtherValue',
      })
    })

fetch返回一个Promise，用于处理返回

    getMoviesFromApiAsync() {
	   return fetch('http://facebook.github.io/react-native/movies.json')
	     .then((response) => response.json())
	     .then((responseJson) => {
	       return responseJson.movies;
	     })
	     .catch((error) => {
	       console.error(error);
	     });
	 }

## Navigator

    render() {
      return (
        <Navigator
          initialRoute={{ title: 'My Initial Scene', index: 0 }}
          renderScene={(route, navigator) => {
            <MyScene title={route.title} />
          }}
        />
      );
    }

Navigator用于页面跳转，initialRoute这个prop是初始的route， 这个route其实是一个js对象  
而 renderScene prop是路由配置函数，根据route 返回相应的界面。

navigator 有push pop等方法跳转页面， [api](http://facebook.github.io/react-native/docs/navigator.html)

# 路由

我使用下面这个router库
[react-native-router-flux](https://github.com/aksonov/react-native-router-flux) 

## [Why I need to use it?](https://github.com/aksonov/react-native-redux-router/blob/master/README.md#why-i-need-to-use-it)
- Use Redux actions to replace/push/pop screens with easy syntax like Actions.login for navigation to login screen
- Forget about passing navigator object to all React elements, use actions from anywhere in your UI code.
- Configure all of your screens ("routes") once (define animations, nav bars, etc.), at one place and then just use short actions commands. For example if you use some special animation for Login screen, you don't need to code it anywhere where an user should be redirected to login screen.
- Use route "schemas" to define common property for some screens. For example some screens are "modal" (i.e. have animation from bottom and have Cancel/Close nav button), so you could define group for them to avoid any code repeatition.
- Use popup with Redux actions (see Error popup within Example project)
- Hide nav bar for some screens easily

## 需要说明的地方
Router作为根component  
Scene代表路由配置  
路由配置并不是树状结构的，虽然Scene的配置看起来像是树状结构，但实际只有每个叶子Scene才真正对应一个界面  
特点是只有叶子节点有代表界面的component属性，虽然部分非叶子节点也有component属性，但是这些一般都是tab页，侧边栏等容器类的component。  
专门写这样一句是因为最开始我以为二级界面应该是一级界面的子Scene，后来发现二级界面和一级界面的Scene是兄弟关系。  
统一的错误处理 可以通过Modal来实现  
登录可以通过Switch来实现，但是页面历史需要注意是否可用（参见下面问题中的关于Switch的描述）
默认的导航栏和tabbar都是覆盖在当前Scene上的，所以需要设置当前Scene的marginTop或marginBottom来解决，
一般做法是在Router的getSceneStyle中设置，参考Example  

## 基础
Scene中key表示route， component表示对应的组件，通过添加Scene的属性来设置一些页面跳转或者效果
initial 表示默认显示
tabs 表示tab容器
icon 是这个tab的icon
Actions.key() 就可以跳转到key对应的界面

## 问题
- 无法清晰的描述界面的栈信息
- Switch 没有区分栈，导致不能pop的界面，如果进行pop，会对真实的栈产生pop效果
- 从store中获取的当前scene可能与真实的情况不符，比如tab页，有时候显示的是tab页中具体的页面，有时是tab页

写了一个小的[demo](https://github.com/new-xd/ReactNativeDemo)

# redux
redux是管理应用数据状态的js库，实现flux设计思路。  

主要思路是 应用的任何变化（也不绝对），都应该由Action发起，经过reducer处理，导致state发生改变，从而引起view改变

## redux的state和Component的state是什么关系？
没关系，这是两个不同的概念  
redux的state表示的是应用整体的状态和数据，比如用户是否是登录状态，用户的个人信息，用户的待办列表  
Component的state表示的是Component的状态和数据，比如登录页中，用户输入的用户名和密码，密码是否明文显示

由于应用都是由Component组合而成的，那redux的state是不是应用中各Component的state组合而成的呢？  
不是，一方面并不是所有的Component的状态都可以在redux的state中找到，比如密码是否明文显示  
另一方面redux的state的组织结构和Component的组织结构是不一样的，虽然可以实现的很像。

真的一点关系都没有吗？
如果正确的使用redux，他们真的没关系

## redux的使用思路
有2个关键的原则：

1. 单一数据源（应用的数据和状态）
2. Component的数据由父Component通过props设置

简单来说，就是任何变化先发起Action，引起redux的state改变，导致Component的props改变，导致view改变。
举个例子：首页加载数据  

- 首页componentDidMount 发起数据请求
- 数据返回，发送Action，type为首页数据请求成功
- reducer处理此Action，更新redux中首页数据
- 首页的props更新为返回的数据
- 首页执行render更新界面

在这个过程中首页这个Component的state根本没有出场。

是不是使用了redux的state，Component的state就不需要了？  
也不是，一些Component的UI变化，可以放到Component自身的state中管理。  
比如说登录页，密码是否明文显示，是不需要存在redux的state中的
