---
title: appledoc生成文档实践
date: 2019-05-05 10:18:26
categories: 工具
tags: 工具 
toc: true
---


<!--more-->

## 安装

第一步，下载appledoc，安装：

```
git clone git://github.com/tomaz/appledoc.git
cd ./appledoc
sudo sh install-appledoc.sh
```

第二步，验证appledoc是否安装成功：

```
appledoc --help
```

第三步， touch命令创建脚本，复制下面代码到脚本，然后添加执行权限，执行脚本：

```
appledoc \
--output ./apiDoc \
-i *.m \
--project-name "GSecretKey" \
--project-company "com.Gome" \
--no-create-docset \
--keep-undocumented-objects \
--keep-undocumented-members \
--no-warn-undocumented-object \
--no-warn-undocumented-member  \
./
```

这样就可以将`./`目录下所有头文件中的注释自动生成出对应的文档到`./apiDoc`目录下。实际使用中，可以根据需要修改源文件路径、生成文件的路径、project-name、project-company。

上面的脚本是可以运行的，但是少了注释，难以理解。不添加注释的原因是：上面的命令太长，使用多行显示，使用 `\`实现多行功能，如果在 `\` 后直接添加注释，脚本会运行报错，没有地方添加注释。所以，可执行的脚本没有注释，下面将有注释的脚本单独列出来，方便理解:

```
#!/bin/bash
appledoc \
#文档输出目录
--output ./apiDoc \                                       
#忽略.m文件，因.m中均为私有api和属性，开源的接口文档中理应忽略掉
-i *.m \                                                         
#工程的名字
--project-name "GSecretKey" \
#公司的名字
--project-company "com.Gome" \
#不生成docset，直接输出html
--no-create-docset \
#没有注释的文件也输出html  -->目的是看到所有的文件
--keep-undocumented-objects \
#没有注释的属性和方法也输出到html  -->目的是看到所有的属性和方法
--keep-undocumented-members \
#没有注释的文件不提示警告
--no-warn-undocumented-object \
#没有注释的属性和方法不提示警告
--no-warn-undocumented-member \
#需要输出的文件路径  -->这里推荐最好直接为当前工程路径平级输出，便于维护和使用
./
```

## 问题

> 实际使用上问题挺多，尤其Xcode9之后，普遍的方法会出现此错误：ERROR | !> xcrun: error: unable to find utility "docsetutil", not a developer tool or in PATH

解决办法：终端->编写脚本->运行脚本->更新脚本从而规避docsetutil找不到等错误，经过验证，将命令放在脚本中，确实解决了这个问题。


## <div id= refer>参考</div>

1. [iOS 开发_编写接口文档（appledoc实用篇）](https://www.jianshu.com/p/f5cb3c728a5b)



