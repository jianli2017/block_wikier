---
title: NSInvocation的基本使
date: 2018-12-11 15:35:49
tags: 小码哥视频
categories: 小码哥视频（上）
toc: true
---

小码哥视频学习记录

<!--more-->

## 第一天 

### ssh 链接iphone手机

```
ssh root@ 10.144.32.27
alpine
```

ifunbox

## 第二天 

ssh 版本：SSH-2

查看SSH版本：

```
cd /etc/ssh
ls
cat ssh_config  or cat sshd_config
```

SSH 通信过程：

1. 建立安全链接
2. 客户端认证(基于密码类型的、基于秘钥的认证（免密码，优先选择 ）)
3. 数据传输

服务器公钥：

```
/etc/ssh/ssh_host_rs_key.pub
```

重新生成秘钥

```
ssh-keygen -R 10.144.36.206
```

自动拷贝公钥到服务器

```
ssh-keygen 
ssh-copy-id
```

手动拷贝公钥到服务器

```
scp ~/.ssh/id_rsa.pub root@10.144.36.206:~/.ssh
cat id_rsa.pub >> authorized_keys
```



22 端口

通过usb进行ssh链接

usbmuxd   端口映射  
python ~/Documents/ios/usbmuxd/tcprelay.py -t 22:10010
ssh root@localhost -p 10010 

sh bash 子进程中执行
source 当前进程中执行

cycript 是C++、JavaScript、java 混合物， 调试某个APP  

cycript 命令进入cycript环境  
cycript -p 进程名称  
adv-cmds  执行ps命令  

UIApp= [UIApplication sharedApplication]

定义变量 :var app = UIApp.keyWindow

```
#地址 可以访问对象
```

```
* 对象  访问对象的成员变量
```

```
递归打印视图： view.recursiveDescription().toString()
```

```
筛选某种类型的视图： choose(UITableViewCell)
```


## 第四天 

拆分胖二进制文件

```
lipo test -thin -armv7 -o test_armv7
lipo -crate test_armv7 test_arm64 -o test
```

file命令查看文件格式

XNU源码中有loader.h文件，定义了文件的格式

otool命令

dyld 只能加载MH_EXECUTE、MH_DYLIB|MH_BUNFLE类型的Mach-O文件






