---
title: IOS12 兼容
date: 2018-08-22 12:07:12
tags: IOS12 兼容
categories: IOS12
toc: true
---

主要遇到的问题：

1. Xcode10移除了libstdc++库，由libc++这个库取而代之，苹果的解释是libstdc++已经标记为废弃有5年了，建议大家使用经过了llvm优化过并且全面支持C++11的libc++库。
2. CocoaPods 1.3.1 版本不能将pod中的资源文件拷贝到APP中

<!--more-->

## libstdc++问题 

* 现象是无法找到libstdc++，编译报错如下：

```
library not found for -lstdc++.6.0.9
```

### 临时解决方案

* 临时解决办法----从Xcode9中复制libstdc++库到Xcode10中，命令如下：

```
cp /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/usr/lib/libstdc++.* /Applications/Xcode-beta.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/usr/lib/

cp /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/usr/lib/libstdc++.* /Applications/Xcode-beta.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/usr/lib/
```

* 最终解决办法：

需要联系第三方公司，将GMThirdParty/BestPay 、GMF_EaseMobSDK2.2.9更新。


### 模拟器无法启动的解决方案

报错如下：    

```
dyld: Library not loaded: /usr/lib/libstdc++.6.dylib
  Referenced from: /Users/lijian/Library/Developer/CoreSimulator/Devices/6CB2CF98-149C-43A1-8A93-516FE4243C8C/data/Containers/Bundle/Application/2C25BADD-A36F-4B11-9885-34E8AEE826AC/GomeStaff.app/GomeStaff
  Reason: no suitable image found.  Did find:
	/usr/lib/libstdc++.6.dylib: mach-o, but not built for iOS simulator
```

原因分析： 

上面的错误的含义是动态链接器无法加载到libstdc++.6.dylib，但是真机上是可以加载出来，所以，推测模拟器的运行环境去掉了这个库，那么，我们如果将模拟器的运行环境中添加上这个库，是不是就可以了？ 答案是肯定的。

下面的问题是：我么如何找到模拟器运行环境加载库的路径呢？ 我的思路是使用xcode9运行模拟器，然后打印系统库的加载路径，这个路径就是我们要找的路径。

查找系统库的方法如下,用xcode9运行下面的代码：

```
#include <mach-o/dyld.h>
#include <mach/mach.h>

for(int i = 0; i < _dyld_image_count(); i++)
{
    const char* aa = _dyld_get_image_name(i);
    NSString *str = [NSString stringWithUTF8String:aa];
    LOG_LJ_(@"~~~~~~~~~~~~~~~~~~~~~~~~%@",str);
}

```

输出如下：

```
~/Users/lijian/Downloads/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/CoreSimulator/Profiles/Runtimes/iOS.simruntime/Contents/Resources/RuntimeRoot/usr/lib/libiconv.2.dylib
```

上面的路径就是模拟器的系统库路径。所以，将xcode9对应目录下的三个libstdc++复制到xcode10的对应目录即可。

## CocoaPods无法复制资源到app中

现象：登录界面没有图标，或者启动时 NSBundle的initWithUrl方法崩溃。

解决办法：

1. 升级CocoaPods到1.4.0，命令是：`sudo gem install -n /usr/local/bin cocoapods -v 1.4.0` 。
2. 移除`Build Phases` 中的 `[cp]Copy Pods Resources` 。
3. 重新pod install。


## info 重复问题

报错如下：

```
Multiple commands produce '/Users/lijian/Library/Developer/Xcode/DerivedData/GomeStaff-eccvobvclqelmrgkhibvvrxbftdz/Build/Products/Debug-iphonesimulator/GomeStaff.app/Info.plist':
1) Target 'GomeStaff' (project 'GomeStaff') has copy command from '/Users/lijian/Desktop/GomeGit/GomeStaff/GomeStaff/Supporting Files/Info.plist' to '/Users/lijian/Library/Developer/Xcode/DerivedData/GomeStaff-eccvobvclqelmrgkhibvvrxbftdz/Build/Products/Debug-iphonesimulator/GomeStaff.app/Info.plist'
2) Target 'GomeStaff' (project 'GomeStaff') has process command with output '/Users/lijian/Library/Developer/Xcode/DerivedData/GomeStaff-eccvobvclqelmrgkhibvvrxbftdz/Build/Products/Debug-iphonesimulator/GomeStaff.app/Info.plist'
```

原因分析：Xcode自动会将Info.plist复制到GomeStaff.app中，但是在Build Phases-> Copy Bundle Resources中也包含复制Info.plist的功能，这样就两次复制，第二次复制失败。

解决方法：

将Build Phases-> Copy Bundle Resources  中的info.plist 去掉。



## 参考 

* [Xcode10和iOS12踩坑](https://juejin.im/post/5b1634f0f265da6e61788998)
* [libstdc++适配Xcode10与iOS12](http://www.cocoachina.com/ios/20180611/23749.html)

