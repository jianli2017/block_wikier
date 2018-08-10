---
title: homebrew-gem使用.
date: 2018-04-23 12:07:12
tags: 工具
categories: 工具
toc: true
---

最近使用cocoapod更新repo会报下面错误：

Updating spec repo `master`
[!] Failed to connect to GitHub to update the CocoaPods/Specs specs repo - Please check if you are offline, or that GitHub is down

经过查找，发现GitHub在2018年2月23日移除了弱加密标准，导致无法正常连接到GitHub。[传送门](https://blog.github.com/2018-02-23-weak-cryptographic-standards-removed/)。

解决办法是：升级Cocoapod、openssl、ruby。首先介绍下更新ruby使用的Homebrew。

<!-- more -->


## Homebrew介绍

Homebrew是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。
援引官方的一句话：又提示缺少套件啦？别担心，Homebrew 随时守候。Homebrew – OS X 不可或缺的套件管理器。

使用ruby安装Homebrew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)”
```

卸载

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/uninstall)"  
```

homebrew使用 

安装包 brew install <packageName>

卸载包 brew uninstall <packageName>

查询可用包 brew search <packageName>

查看已经安装列表 brew list

查看任意包信息brew info <packageName>

## ruby更新

使用brew安装ruby很方便，但缺点也是很明显的，不能实时进行版本的切换，所以还是用 brew + rvm 或brew + rbenv比较好，但是后面的这种方法我没有走通。所以我使用Homebrew更新ruby。

我们不去删除系统自带的ruby，gem，而是自己重新安装一套新的ruby，gem，通过更改PATH环境变量的方式来更新系统，

这样做好处比较安全的，不会破坏原有的苹果系统，又不耽误我使用最新的ruby。

#### 更新Home brew

好了，开始吧

```
brew update
brew install ruby
```

#### 设置环境变量

系统原始版本的/usr/bin/ruby 我们并不删除，只是更改PATH环境变量，且将/usr/local/bin 添加到PATH的前面，这样系统就会首先用

/usr/local/bin下面找到我们用brew安装的ruby ruby 2.5.1p57(2018-03-29 revision 63029) [x86_64-darwin16]版本的了。

到自己目录下的.profile 或者 .bashrc 或者  .bash_profile    

用vim打开 （更改前请备份好这个文件，避免误操作）

在文件的末尾加入:

```
# for brew install
export PATH=/usr/local/bin:$PATH
```

然后重启终端，就可以用到了新的ruby了

检验一下

```
$ ruby --version
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-darwin16]
$ which ruby
/usr/local/bin/ruby               注意：系统的是 /usr/bin/ruby
```
   
#### gem的自我更新 

gem是随着ruby的安装而安装的，所以路径和刚刚安装ruby的路径是相同过的，

此gem是自己安装路径中的gem (/usr/local/bin/gem)，不是系统的gem(/usr/bin/gem)，我们这里也不动系统的gem

```
$ which gem
/usr/local/bin/gem
```

## gem 介绍

Gem是一个管理Ruby库和程序的标准包，它通过[Ruby Gem](http://rubygems.org/)源来查找、安装、升级和卸载软件包，非常的便捷。

Ruby 1.9.2版本默认已安装Ruby Gem，如果你使用其它发行版本，请参考“如何安装Ruby Gem”。

Ruby gem包的安装方式：

所有的gem包，会被安装到 /[Ruby root]/lib/ruby/gems/[ver]/ 目录下，这其中包括了Cache、doc、gems、specifications 4个目录，cache下放置下载的原生gem包，gems下则放置的是解压过的gem包。
当安装过程中遇到问题时，可以进入这些目录，手动删除有问题的gem包，然后重新运行 gem install [gemname] 命令即可。

Ruby Gem命令详解：

//更新Gem自身
//注意：在某些linux发行版中为了系统稳定性此命令禁止执行
$ gem update --system

// 从Gem源安装gem包
$ gem install [gemname]

// 从本机安装gem包
$ gem install -l [gemname].gem

// 安装指定版本的gem包
$ gem install [gemname] --version=[ver]

// 更新所有已安装的gem包
$ gem update

// 更新指定的gem包
// 注意：gem update [gemname]不会升级旧版本的包，此时你可以使用 gem install [gemname] --version=[ver]代替
$ gem update [gemname]

// 删除指定的gem包，注意此命令将删除所有已安装的版本
$ gem uninstall [gemname]

// 删除某指定版本gem
$ gem uninstall [gemname] --version=[ver]

// 查看本机已安装的所有gem包
$ gem list [--local]


## 解决无法连接到GitHub问题

通过下面的命令可以解决链接到Github失败的问题 ：

#### 升级openssl

```
$ which openssl
/usr/local/opt/openssl/bin/openssl

$ openssl version
OpenSSL 1.0.2o  27 Mar 2018

#如果openssl版本低，请使用Homebrew更新

$ brew update

$ brew install openssl

$ brew upgrade openssl

`` If you need to have this software first in your PATH run: echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile

$ echo 'export PATH="/usr/local/opt/openssl/bin:$PATH"' >> ~/.bash_profile
$ source ~/.bash_profile

$ which openssl
/usr/local/opt/openssl/bin/openssl

$ openssl version
OpenSSL 1.0.2n  7 Dec 2017
```

####  升级ruby  

$ ruby --version
ruby 2.5.0p0 (2017-12-25 revision 61468) [x86_64-darwin16]

如果版本比较低，请参考ruby升级

#### 升级cocoapod



$ gem install cocoapods -n /usr/local/bin

$ which pod
/usr/local/bin/pod

$ pod --version
1.5.0


## 参考

1. [mac系统用HomeBrew直接安装ruby](https://blog.csdn.net/wks_lovewei/article/details/73369333)
2. [Cocoapods: Failed to connect to GitHub to update the CocoaPods/Specs specs repo](https://stackoverflow.com/questions/38993527/cocoapods-failed-to-connect-to-github-to-update-the-cocoapods-specs-specs-repo)


