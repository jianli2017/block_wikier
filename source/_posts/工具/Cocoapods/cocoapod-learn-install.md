---
title: cocoapod学习 安装和使用（1）
toc: true
date: 2018-02-27 10:29:06
tags:
categories:
---

[CocoaPods](https://guides.cocoapods.org/) Get on with building your app, not duplicating code ,是管理iOS工程依赖的第三方库的工具，通过CocoaPods，我们可以很方便地管理每个第三方库的版本，且不需要做太多的配置。本文在[看一遍就会的CocoaPods的安装和使用教程](https://www.jianshu.com/p/1711e131987d)的基础上稍微修改而来。
<!-- more -->

## 什么是CocoaPods？

[CocoaPods](https://guides.cocoapods.org/)是管理iOS工程依赖的第三方库的工具，通过CocoaPods，我们可以很方便地管理每个第三方库的版本，且不需要我们做太多的配置。直观、集中和自动化地管理项目的第三方库。

我们都有这样的经历，当添加第三方库的时候，需要导入一堆相关依赖库。当需要更新某个第三方库的时候，又需要手动移除该库，导入新的库，然后再配置。这些是很麻烦且没有意义的工作。

使用CocoaPods管理第三方库后，只需要相当少的配置，就可将第三方库集成到工程中，其它的一切都交由CocoaPods来管理即可，我们使用起来就更省心了。

## 安装CocoaPods

1.首先更新gem到最新版本，在终端中输入：sudo gem update --system更新gem

> Gem是一个管理Ruby库和程序的标准包，它通过Ruby Gem（如 http://rubygems.org/ ）源来查找、安装、升级和卸载软件包，非常的便捷。

2.删除自带的ruby镜像，终端输入：`gem sources --remove https://rubygems.org/`。 

3.添加淘宝的镜像，终端输入：`gem sources -a https://gems.ruby-china.org/`(原来的淘宝镜像 https://ruby.taobao.org/已经不能用了)。 

4.可以用`gem sources -l`来检查使用替换镜像位置成功，结果应该只有 `https://gems.ruby-china.org/` 才对。

5.安装CocoaPods，终端输入：sudo gem install cocoapods。 

> 如果提示Operation not permitted，执行sudo gem install -n /usr/local/bin cocoapods命令安装，原因请参考[系统集成保护](https://www.jianshu.com/p/23c01067cf7e)。

6.然后配置下CocoaPods，终端输入：pod setup。

到这里CocoaPods就安装好了。

## 查找第三方库

比如查找MJExtension，终端输入：`pod search MJExtension`，第一次搜索他需要建索引，等待一会儿就可以了。

输出结果如下：

```
-> MJExtension (3.0.13)
   A fast and convenient conversion between JSON and model
   pod 'MJExtension', '~> 3.0.13'
   - Homepage: https://github.com/CoderMJLee/MJExtension
   - Source:   https://github.com/CoderMJLee/MJExtension.git
   - Versions: 3.0.13, 3.0.12, 3.0.11, 3.0.10, 3.0.9, 3.0.8, 3.0.7, 3.0.6,
   3.0.5, 3.0.4, 3.0.3, 3.0.2, 3.0.0, 2.5.16, 2.5.15, 2.5.14, 2.5.13, 2.5.12,
   2.5.10, 2.5.9, 2.5.8, 2.5.7, 2.5.6, 2.5.5, 2.5.3, 2.5.2, 2.5.1, 2.5.0, 2.4.4,
   2.4.2, 2.4.1, 2.4.0, 2.3.8, 2.3.7, 2.3.6, 2.3.5, 2.3.4, 2.3.3, 2.3.2, 2.3.1,
   2.3.0, 2.2.0, 2.1.1, 2.1.0, 2.0.4, 2.0.3, 2.0.2, 2.0.1, 2.0.0, 1.2.1, 1.2.0,
   1.1.0, 1.0.1, 1.0.0, 0.3.2, 0.3.1, 0.3.0, 0.2.0, 0.1.3, 0.1.2, 0.1.1, 0.1.0,
   0.0.3, 0.0.2, 0.0.1 [master repo]

-> MJExtension_HPTest (0.0.1)
   花圃测试项目
   pod 'MJExtension_HPTest', '~> 0.0.1'
   - Homepage: https://github.com/LetMeCrazy/testPods
   - Source:   https://github.com/LetMeCrazy/testPods.git
   - Versions: 0.0.1 [master repo]

```
## 引入第三方库到项目中

我先在桌面上新建一个cocoapodUse项目，然后演示把MJExtension导进去。

2.然后生成并编辑一个`Podfile`文件，命令为:

`vim Podfile`

要导入的第三方都要写在Podfile中。进去后需要先按

`I` 键进入编辑状态，写完后按`esc`，然后按`shift+zz`(或者先按`shift+:`,再按`wq`)就可以保存退出了。

Podfile的格式如下，其中'cocoapodUse'为你的target的名字。

```
platform :ios,'8.0'

target 'cocoapodUse' do

pod 'MJExtension', '~> 3.0.13'

end
```

3.安装，命令为：

`pod install`

安装成功之后，第三方库就被包含到项目中了。

之前我们一直是双击Test.xcodeproj打开项目，以后我们就要双击Test.xcworkspace打开了，可以看到MJExtension已经被引入了。

## 项目引入MJExtension

你会发现当引入MJExtension的头文件时，可以 `#import <MJExtension.h>`或者`#import <MJExtension/MJExtension.h>`，但是却不能在输入`#import "MJExtension.h"`的时候出现提示。虽然强制输入也可以编译通过，但是感觉很不爽。 解决这个问题的办法是在工程的Build Settings搜索Search，然后在User header search paths中添加$(SRCROOT)并选择recursive。

现在就可以提示#import "MJExtension.h"啦。


## 增加新的第三方

如果使用过程中我还想添加其他的第三方怎么办，只要在Podfile里面接着添加，然后终端再执行pod install就可以了。


## 更新CocoaPods中的第三方库

第三方库们都有人在维护升级，我们需要隔断时间就要更新下我们工程中第三方库的版本。只需要终端输入命令pod update就可以了。

如果遇到pod install或者pod update慢的问题，原因在于当执行以上两个命令的时候会升级CocoaPods的spec仓库，加一个参数可以省略这一步，然后速度就会提升不少。加参数的命令如下： pod install --verbose --no-repo-update 

## 删除CocoaPods中的某些第三方库。

当我们需要去掉某个第三方库时，只需要在Podfile删除该引入该库的语句，然后执行pod update或者pod install就可以了。

## 升级CocoaPods

升级CocoaPods版本的命令和安装CocoaPods的命令一样，都是sudo gem install cocoapods。 如果老版本升级cocoapods的时候提示Operation not permitted - /usr/bin/xcodeproj，改用命令sudo gem install -n /usr/local/bin cocoapods --pre就可以了。

## 卸载CocoaPods

卸载CocoaPods的命令是sudo gem uninstall cocoapods

执行完命令后，最下面打印Successfully uninstalled cocoapods字样就代表已经成功卸载了。

## CocoaPods Mac App的安装和使用

CocoaPods桌面应用版下载地址：[https://cocoapods.org/app][1] 打开应用会提示你是否安装命令行工具，选择install就也可以在命令行使用Pod了。省去了上面的步骤们，方便快捷的使用CocoaPods。

现在假如要给一个Test项目加入第三方库 1.选择File-New Podfile from Xcode Project，去选择项目的Project文件。

2.填写自动生成的Podfile，并且安装。

然后就可以去打开工程了，是不是比命令行简单多了。 

> 注意：Cocoapods.app 删掉并执行命令可能会报错：Unable to locate the CocoaPods.app application bundle. Please ensure the application is available and launch it at least once
> 这时候只要执行sudo gem install -n /usr/local/bin cocoapods命令就可以了。

## CocoaPods官方使用指南

链接：[CocoaPods 官方使用指南](https://guides.cocoapods.org/)有什么不了解的或者遇到错误可以去这里查看一下。

## XCode的CocoaPods插件

[CocoaPods-xcode-plugin]是一个XCode的插件，可以很方便的在Xcode通过pods安装各种第三方库。前提是终端已经安装好CocoaPods。

## 参考资料
* [看一遍就会的CocoaPods的安装和使用教程](https://www.jianshu.com/p/1711e131987d)
* [升级 OS X 10.11 cocoapods 使用不正常的问题](https://www.jianshu.com/p/23c01067cf7e)

