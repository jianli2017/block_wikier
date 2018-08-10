---
title: 合并真机模拟器静态库(转载)
date: 2018-06-06 12:07:12
tags: ios
categories: ios
toc: true
---

这篇文章解释合并真机模拟器静态库的脚本

<!-- more -->

## 合成脚本

合并的脚本如下：

```
if [ "${ACTION}" = "build" ]
then
#要build的target名
target_Name=${PROJECT_NAME}
    echo "target_Name=${target_Name}"

#build之后的文件夹路径
build_DIR=${SRCROOT}/build
  echo "build_DIR=${build_DIR}"
#真机build生成的头文件的文件夹路径
DEVICE_DIR_INCLUDE=${build_DIR}/Release-iphoneos/include/${PROJECT_NAME}
    echo "DEVICE_DIR_INCLUDE=${DEVICE_DIR_INCLUDE}"
#真机build生成的.a文件路径
DEVICE_DIR_A=${build_DIR}/Release-iphoneos/lib${PROJECT_NAME}.a
    echo "DEVICE_DIR_A=${DEVICE_DIR_A}"
#模拟器build生成的.a文件路径
SIMULATOR_DIR_A=${build_DIR}/Release-iphonesimulator/lib${PROJECT_NAME}.a
    echo "SIMULATOR_DIR_A=${SIMULATOR_DIR_A}"

#目标文件夹路径
INSTALL_DIR=${SRCROOT}/Products/${PROJECT_NAME}
    echo "INSTALL_DIR=${INSTALL_DIR}"
#目标头文件文件夹路径
INSTALL_DIR_Headers=${SRCROOT}/Products/${PROJECT_NAME}/Headers
    echo "INSTALL_DIR_Headers=${INSTALL_DIR_Headers}"
#目标.a路径
INSTALL_DIR_A=${SRCROOT}/Products/${PROJECT_NAME}/lib${PROJECT_NAME}.a
    echo "INSTALL_DIR_A=${INSTALL_DIR_A}"

#判断build文件夹是否存在，存在则删除
if [ -d "${build_DIR}" ]
then
rm -rf "${build_DIR}"
fi
#判断目标文件夹是否存在，存在则删除该文件夹
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
#创建目标文件夹
mkdir -p "${INSTALL_DIR}"

#build之前clean一下
xcodebuild -target ${target_Name} clean

#模拟器build
xcodebuild -target ${target_Name} -configuration Release -sdk iphonesimulator
#真机build
xcodebuild -target ${target_Name} -configuration Release -sdk iphoneos
#复制头文件到目标文件夹
cp -R "${DEVICE_DIR_INCLUDE}" "${INSTALL_DIR_Headers}"
#合成模拟器和真机.a包
lipo -create "${DEVICE_DIR_A}" "${SIMULATOR_DIR_A}" -output "${INSTALL_DIR_A}"

#打开目标文件夹
open "${INSTALL_DIR}"
fi

```

## 铺路

本代码中用到的核心命令：

### xcodebuild
苹果给的一个命令。主要用来编译Xcode的工程。
可以在终端中输入xcodebuild -h来查看命令的详情，介绍一下本脚本中用到的几个参数

* clean:clean一下工程
* -configuration Release
使用Release方式编译，还可以使用Debug
* -sdk iphoneos
真机编译，还可以使用-sdk iphonesimulator模拟器编译

### cp "源文件路径" "目标文件路径"
复制"源文件路径"的文件到 "目标文件路径"

### lipo 

lipo -create "模拟器.a文件路径" "真机.a文件路径" -output "目标.a文件路径"

将模拟器和真机的.a包合成。

### 用到的一些shell脚本基础命令

#### echo "你要写的东西"
打印的log,将"你要写的东西"打印出来，相当于OC中的NSLog
Xcode的话，运行脚本后，可以在这里找到log

#### 赋值命令。 变量名=变量值
比如将"CrazyStone"赋值给MyName变量

MyName=CrazyStone
#### ${变量名}

取出变量名的内容。
比如：取出变量MyName中的内容
${MyName}
#### 判断语句
if [ 条件语句 ]then
...
fi
条件语句为真就执行then后面的语句，不成立就结束判断语句
#### 本脚本中用到的判断语句：
[ -d "文件夹路径" ] ：判断是否为文件夹

## 脚本结构解释

看完上面，我想你再看一下代码应该就能理解脚本，然后可以做一些简单的改动了。下面再介绍一下脚本的结构。

### 执行条件-- 编译

```
if [ "${ACTION}" = "build" ]
then
#我们的大部分脚本代码
fi
```

执行脚本的时候做个判断，在Xcode里面build这个工程的时候就执行then后面的脚本


### 工程名称定义

```
#要build的target名
target_Name=${PROJECT_NAME} 
    echo "target_Name=${target_Name}"
```
变量target_Name是我们要编译的target的名字，在这里指的是工程的名字${PROJECT_NAME}，也就是MySDK。

顺便说一下，ACTION和PROJECT_NAME都是Xcode里面定义的，这是在Xcode里面写脚本的一个好处。

###  build 路径


```
#build之后的文件夹路径
build_DIR=${SRCROOT}/build
    echo "build_DIR=${build_DIR}"

#真机build生成的头文件的文件夹路径
DEVICE_DIR_INCLUDE=${build_DIR}/Release-iphoneos/include/${PROJECT_NAME}
    echo "DEVICE_DIR_INCLUDE=${DEVICE_DIR_INCLUDE}"

#真机build生成的.a文件路径
DEVICE_DIR_A=${build_DIR}/Release-iphoneos/lib${PROJECT_NAME}.a
    echo "DEVICE_DIR_A=${DEVICE_DIR_A}"

#模拟器build生成的.a文件路径
SIMULATOR_DIR_A=${build_DIR}/Release-iphonesimulator/lib${PROJECT_NAME}.a
    echo "SIMULATOR_DIR_A=${SIMULATOR_DIR_A}"
```

这里是定义的build之后各个文件的路径。我们执行了xcodebuild命令之后，会在工程目录生成一个build文件夹，里面有build之后生成的文件。打开Finder看看就知道各个文件的路径了。


### build目录的位置

```
#目标文件夹路径
INSTALL_DIR=${SRCROOT}/Products/${PROJECT_NAME}
    echo "INSTALL_DIR=${INSTALL_DIR}"

#目标头文件文件夹路径
INSTALL_DIR_Headers=${SRCROOT}/Products/${PROJECT_NAME}/Headers
    echo "INSTALL_DIR_Headers=${INSTALL_DIR_Headers}"

#目标.a路径
INSTALL_DIR_A=${SRCROOT}/Products/${PROJECT_NAME}/lib${PROJECT_NAME}.a
    echo "INSTALL_DIR_A=${INSTALL_DIR_A}"
```

这里就是定义目标变量的路径了。你想把文件放在哪里？在这里定义咯。${SRCROOT}表示工程的根目录。用了这么久的Xcode，这个有用过吧(全局头文件配置过吧？)？

### 文件状态判断

```
#判断build文件夹是否存在，存在则删除
if [ -d "${build_DIR}" ]
then
rm -rf "${build_DIR}"
fi

#判断目标文件夹是否存在，存在则删除该文件夹
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
#创建目标文件夹
mkdir -p "${INSTALL_DIR}"
```


这里就是文件的操作了。如果有这两个文件夹，就删除掉。为什么？为了保证我们工程的纯净啊。

### 编译


```
#build之前clean一下
xcodebuild -target ${target_Name} clean

#模拟器build
xcodebuild -target ${target_Name} -configuration Release -sdk iphonesimulator

#真机build
xcodebuild -target ${target_Name} -configuration Release -sdk iphoneos
```

这里就跟平常操作一样了。先clean一下工程，然后模拟器编译一次，真机编译一次。

### 合并

```
#复制头文件到目标文件夹
cp -R "${DEVICE_DIR_INCLUDE}" "${INSTALL_DIR_Headers}"

#合成模拟器和真机.a包
lipo -create "${DEVICE_DIR_A}" "${SIMULATOR_DIR_A}" -output "${INSTALL_DIR_A}"
```

关键代码。拷贝头文件到我们的目标位置去。合成.a包。大功告成。


### 打开目标文件夹

```
#打开目标文件夹
open "${INSTALL_DIR}"
```

最后，打开文件夹。检查一下文件是否真正生成了。

## shell脚本基础知识

如果你想了解更多关于shell脚本的知识，可以看看这篇文章：Linux shell脚本基础学习详细介绍


## xcworkspace 工程对应的脚本

```
#模拟器build
xcodebuild -workspace ${target_Name}.xcworkspace -scheme ${target_Name} -configuration ${build_model} -sdk iphonesimulator

#真机build
xcodebuild -workspace ${target_Name}.xcworkspace -scheme ${target_Name} -configuration ${build_model} -sdk iphoneos
```
对于xcworkspace工程，需要将编译的脚本替换 ，其中--workspace、-scheme是必须的 ，scheme 可以通过xcodebuild -list 查看。

## iOS设备架构

模拟器：
iPhone4s-iPnone5：i386
iPhone5s-iPhone7 Plus：x86_64

真机:
iPhone3gs-iPhone4s：     armv7
iPhone5-iPhone5c：        armv7s
iPhone5s-iPhone7 Plus： arm64

## 参考

[【iOS开发】静态库.a文件合成脚本解释](https://www.jianshu.com/p/9cf90b9537fd)