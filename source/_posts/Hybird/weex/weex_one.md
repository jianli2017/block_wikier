---
title: weex系列抄之一---环境搭建
date: 2018-05-04 12:07:12
tags: weex
categories: weex
toc: true
---

## React Native 和 Weex

自从Weex出生的那一天起，就无法摆脱和React Native相互比较的命运。React Native宣称“Learn once, write anywhere”，而Weex宣称“Write Once, Run Everywhere”。Weex从出生那天起，就被给予了一统三端的厚望。React Native可以支持iOS、Android，而Weex可以支持iOS、Android、HTML5。

在Native端，两者的最大的区别可能就是在对JSBundle是否分包。React Native官方只允许将React Native基础JS库和业务JS一起打成一个JS bundle，没有提供分包的功能，所以如果想节约流量就必须制作分包打包工具。而Weex默认打的JS bundle只包含业务JS代码，体积小很多，基础JS库包含在Weex SDK中，这一点Weex与Facebook的React Native和微软的Cordova相比，Weex更加轻量，体积小巧。

在JS端，Weex又被人称为Vue Native，所以 React Native 和 Weex 的区别就在 React 和 Vue 两者上了。

笔者没有写过React Native，所以也没法客观的去比较两者。不过知乎上有一个关于Weex 和 React Native很好的对比文章[《weex&React Native对比》](https://zhuanlan.zhihu.com/p/21677103)，推荐大家阅读。

前两天[@Allen 许帅](http://www.weibo.com/122678100)也在[Glow 技术团队博客](http://tech.glowing.com/cn/)上面发布了一篇[《React Native 在 Glow 的实践》](http://tech.glowing.com/cn/react-native-at-glow/)这篇文章里面也谈了很多关于React Native实践相关的点，也强烈推荐大家去阅读。

## 入门手册

关于小白想入门Weex，当然最基础的还是要通读文档，文档是官方最好的学习资料。官方的基础文档有两份：

[教程文档](http://weex-project.io/cn/guide/)  
[手册文档](http://weex-project.io/cn/references/)


在文档手册里面包含了Weex所有目前有的组件，模块，每个组件和模块的用法和属性。遇到问题可以先过来翻翻。很有可能有些组件和模块没有那些属性。

## Weex系列工具

看完官方文档以后，就可以开始上手构建工程项目了。

Weex也和前端项目一样，拥有它自己的脚手架全家桶。

1. weex-toolkit 
2. weexpack 
3. playground 
4. code snippets 
5. weex-devtool。

#### weex-toolkit

weex-toolkit是用来初始化项目，编译，运行，debug所有工具。

通过下面的命令安装 

```
npm install -g weex-toolkit
```

安装完成后就可以使用weex 命令了 

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weex-toolkit.png)

我们主要用的就是debug 和 compile命令。debug 是启动chrome调试器 、compile命令是将写好的js文件编译为JS bundle。

#### weexpack

weexpack是用来打包JSBundle的，实际也是对Webpack的封装。

#### playground

playground是一个上架的App，这个可以用来通过扫码实时在手机上显示出实际的页面。

 ![](http://of685p9vy.bkt.clouddn.com/hybird/weex/playground.png)

#### code snippets

code snippets这个是一个在线的playground。

 ![](http://of685p9vy.bkt.clouddn.com/hybird/weex/dotwe.png)

#### weex-devtool

weex-devtool 可以使用chrome调试JS 代码。  这个工具我认为主要方便前端开发使用。

使用步骤，首先启动调试服务，终端输入

```
weex debug
```

结果如下:

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weexdebug.png)

同时启动chrome。然后在代码中将host换为图中的host。

```
NSString *strHost =  [[NSUserDefaults standardUserDefaults] objectForKey:GWX_WEEX_DEBUG_HOST];
    if(strHost.length>5)
    {
        NSString *strURL = [NSString stringWithFormat:@"ws://%@/debugProxy/native",strHost];
        [WXDevTool setDebug:YES];
        [WXLog setLogLevel:WXLogLevelLog];
        [WXDevTool launchDevToolDebugWithUrl:strURL];
    }
    else
    {
        [WXDebugTool setDebug:NO];
        [WXLog setLogLevel:WXLogLevelError];
    }
```

启动app，看到app链接到了 chrome调试服务器 

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weexdebugapp.png)

下图是debuger 截图 ：

![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weexdebugdebug.png)

下图是inspect截图：
![](http://of685p9vy.bkt.clouddn.com/hybird/weex/weexdebugpage.png)

具体使用方法看这两篇文章即可：

1. [《Weex 入坑指南：Debug 调试是一门手艺活》](https://zhuanlan.zhihu.com/p/25331465)  

2. [《Weex调试神器——Weex Devtools使用手册》](https://github.com/weexteam/article/issues/50)



## [Weex Market插件](https://github.com/halfrost/Halfrost-Field/blob/master/contents/iOS/Weex/Weex_pseudo-best_practices_for_iOS_developers.md#2-weex-market%E6%8F%92%E4%BB%B6)

在日常开发中，我们可以全部自己开发完所有的Weex界面，当然还可以用一些已有的优秀的轮子。Weex的所有优秀的轮子都在Weex Market里面。

 [Weex Market](https://camo.githubusercontent.com/bdaba9a49b7d7b32486fb7d68a1c5c965c07b19c/687474703a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f313139343031322d353631643838663562333739353763352e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430) 

在这个Market里面有很多已经写好的轮子，直接拿来用，可以节约很多时间。

比如这里很火的weex-chart。weex-chart图表插件是通过g2-mobile依赖[gcanvas插件][23]实现的

如果你想使用[Weex Market][24]的Plugin插件，你可以使用weex plugin 命令：

```
$ weex plugin add plugin_name
```

你只需要输入插件的名称就可以从远程添加插件到你本地的项目，比如添加 weex-chart，我们可以输入命令：

```
$ weex plugin add weex-chart
```

我们可以使用plugin remove移除插件，比如移除安装好的 weex-cahrt：

```
$ weex plugin remove weex-chart

```

这个插件库里面我用过weex-router，还不错，用它来做weex的路由管理。推荐使用。


## 参考

1. [官方教程](http://weex-project.io/cn/guide/)
2. [官方手册](http://weex-project.io/cn/references/)
3. [iOS 开发者的 Weex 伪最佳实践指北](https://github.com/halfrost/Halfrost-Field/blob/master/contents/iOS/Weex/Weex_pseudo-best_practices_for_iOS_developers.md#ios-开发者的-weex-伪最佳实践指北)
