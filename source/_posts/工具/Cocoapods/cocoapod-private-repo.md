---
title: 创建私有仓库
toc: true
date: 2018-03-01 14:50:54
tags:
categories:
photos: 
- http://of685p9vy.bkt.clouddn.com/cocoa-pod/createRepo/privateRepo.png
---

本文说明制作Cocoapod私有库的过程。本文涉及到两个仓库 '官方仓库'、'私有仓库'。

1. [官方仓库](https://github.com/CocoaPods/Specs.git)的作用代表CocoaPods的官方podspec存放地址。 具体可以参考:[CocoaPods官方源](https://github.com/CocoaPods/Specs)、[Specs](http://guides.cocoapods.org/making/specs-and-specs-repo.html); 
2. [私有仓库](https://github.com/jianli2017/LJRepo.git)的是私有podspec存放地址。在git中仓库名称是LJRepo，clone到本地的别名是MyPrivateRepo。后文中私有仓库--LJRepo指的git地址，MyPrivateRepo指的是clone到本地的名称。

<!--more-->


## 创建私有仓库


仓库（Spec Repo） 是所有的Pods的一个索引，是所有公开\私有Pods的podspec文件仓库，其实就是一个部署在服务器的Git仓库，当你使用CocoaPods 后它会被Clone到本地的 ~/.cocoapods/repos 目录下。

首先在git上创建一个私有远端仓库[LJRepo](https://github.com/jianli2017/LJRepo.git)，步骤如下：

1. 在GitHub上创建私有仓库[LJRepo](https://github.com/jianli2017/LJRepo.git)，空的就可以。

2. 将私有仓库[LJRepo](https://github.com/jianli2017/LJRepo.git)添加到cocoapod中，远端私有仓库[LJRepo](https://github.com/jianli2017/LJRepo.git) 在本地中的别名是MyPrivateRepo，这样以后操作MyPrivateRepo就相当于操作[LJRepo](https://github.com/jianli2017/LJRepo.git)， 命令如下：

```
pod repo add 'MyPrivateRepo' 'git@github.com:jianli2017/LJRepo.git' 
```

> 注意：这个Git 仓库地址要换成你自己的创建的 Specs git 地址！！！ 成功后会在~/.cocoapods/repos目录下就能看到MyPrivateRepo了，至此，第一步就完成了创建私有仓库。

创建完成后，查看~/.cocoapods/repos 目录的变化：

```
# cd 到~/.cocoapods/repos 目录
cd ~/.cocoapods/repos

# 查看目录结构
tree -L 3
```

大概的文件目录如下:

```
.
├── MyPrivateRepo
│   └── LJMenu
│       └── 1.0.1
└── master
    ├── CocoaPods-version.yml
    ├── README.md
    └── Specs
        ├── 0
        ├── 1
        ├── 2
```

其中master就是官方的Sepc Repo,跟master同目录级别的MyPrivateRepo目录就是我自己创建的私有Sepc Repo。私有仓库中LJMenu是以前创建的，如果以前没有创建，MyPrivateRepo下面是空的。

也可以使用'pod repo list ' 命令查看仓库信息，结果如下：

```
master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/lijian/.cocoapods/repos/master

MyPrivateRepo
- Type: git (master)
- URL:  git@github.com:jianli2017/LJRepo.git
- Path: /Users/lijian/.cocoapods/repos/MyPrivateRepo

2 repos
```

小结

上面讲解了私有仓库的创建方法。创建完成后，从两个方面描述私有仓库，进一步认识私有创库：文件目录、pod命令


## 创建[LJMenu库](https://github.com/jianli2017/LJMenu.git)

1.创建LJMenu库： 首先，在Git上创建一个[LJMenu仓库](https://github.com/jianli2017/LJMenu.git),当然你也是可以在公司内网创建的。 创建方法使用Cocoapods提供的一个[Using Pod Lib Create](http://guides.cocoapods.org/making/using-pod-lib-create) 工具。

在Terminal中执行cd命令，进入要创建项目的目录，执行以下命令：

```
#pod lib create [项目名]
pod lib create LJMenu

```

'pod lib create'命令会在当前目录创建LJMenu项目，接着在Terminal控制台会输出：

```
Cloning `https://github.com/CocoaPods/pod-template.git` into `LJMenu`.
Configuring LJMenu template.
Ignoring ffi-1.9.14 because its extensions are not built.  Try: gem pristine ffi --version 1.9.14

------------------------------

To get you started we need to ask a few questions, this should only take a minute.

2018-03-02 13:55:04.386 defaults[30912:784564] 
The domain/default pair of (org.cocoapods.pod-template, HasRunbefore) does not exist
If this is your first time we recommend running through with the guide: 
 - http://guides.cocoapods.org/making/using-pod-lib-create.html
 ( hold cmd and double click links to open in a browser. )

 Press return to continue.

```

选择回车按钮，接着会出现一系列的问题：

```
What platform do you want to use?? [ iOS / macOS ]
 > ios

What language do you want to use?? [ Swift / ObjC ]
 > objc

Would you like to include a demo application with your library? [ Yes / No ]
 > yes

Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > specta

Would you like to do view based testing? [ Yes / No ]
 > yes

What is your class prefix?
 > LJ
```
回答完问题后，会创建出LJMenu项目。结构如下：

```
.
├── Example
│   ├── LJMenu
│   ├── LJMenu.xcodeproj
│   ├── LJMenu.xcworkspace
│   ├── Podfile
│   ├── Podfile.lock
│   ├── Pods
│   └── Tests
├── LICENSE
├── LJMenu     **这个是创建的LJMenu项目**
│   ├── Assets
│   └── Classes
├── LJMenu.podspec  
├── README.md
└── _Pods.xcodeproj -> Example/Pods/Pods.xcodeproj

```

2、添加实现代码

```
LJMenu
├── Assets
└── Classes
    ├── LJMenu
    │   ├── IFMMenu.h
    │   ├── IFMMenu.m
    │   ├── IFMMenuContainerView.h
    │   ├── IFMMenuContainerView.m
    │   ├── IFMMenuItem.h
    │   ├── IFMMenuItem.m
    │   ├── IFMMenuView.h
    │   └── IFMMenuView.m
    └── ReplaceMe.m

3 directories, 9 files

```

在本教程中我在上面的Classes文件目录添加了 IFMMenu*.[h、m]八个文件。
3.开发模式下测试pod库的代码 打开Example工程目录Podfile文件：

```
pod 'MyLib', :path => '../' # 指定路径
#pod 'MyLib', :podspec => '../MyLib.podspec'  # 指定podspec文件

```

然后在Example工程目录下执行 pod install命令安装依赖，打开项目工程，可以看到库文件都被加载到Pods子项目中了 不过它们并没有在Pods目录下，而是跟测试项目一样存在于Development Pods/MyLib中，这是因为我们是在本地测试，而没有把podspec文件添加到Spec Repo中的缘故。测试库文件没有问题,接着我们需要执行第4步。

4.提交[LJMenu库](https://github.com/jianli2017/LJMenu.git)到git上。 在Terminal中执行以下命令：

```
git add .
git commit -m '1.0.2'
git remote add origin git@github.com:jianli2017/LJMenu.git
git push origin master     #提交到远端仓库
git tag -m "v1.0.2" "v1.0.2" #打上标签，这个很重要
git push --tags     #推送tag到远端仓库

```

到这里，成功提交到远程仓库---[LJMenu库](https://github.com/jianli2017/LJMenu.git)，以后就可以使用git上的LJMenu库了。

## 创建并提交LJMenu库的podspec文件到私有仓库MyPrivateRepo

1.配置LJMenu库的podspec 文件

```
#
# Be sure to run `pod lib lint LJMenu.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'LJMenu'
  s.version          = '1.0.2'
  s.summary          = 'A short description of LJMenu.'

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!

  s.description      = <<-DESC
TODO: Add long description of the pod here.
                       DESC

  s.homepage         = 'https://github.com/jianli2017/LJMenu'
  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'jianli2017' => 'lijian-ds1@gomeplus.com' }
  s.source           = { :git => 'https://github.com/jianli2017/LJMenu.git', :tag => 'v1.0.2'}
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.ios.deployment_target = '8.0'

  s.source_files = 'LJMenu/Classes/**/*'
  
  # s.resource_bundles = {
  #   'LJMenu' => ['LJMenu/Assets/*.png']
  # }

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end
```

podspec更多配置请参考:[官方文档](http://guides.cocoapods.org/syntax/podspec.html)

2.验证LJMenu.podspec

编辑完LJMenu.podspec文件后，需要验证一下这个LJMenu.podspec文件是否可用 ,在Terminal中执行cd进入MyLib项目根目录然后，执行以下命令：

```
pod spec lint  --allow-warnings 

```

当你看到 Terminal 中输出：

```
 -> LJMenu (1.0.2)
    - WARN  | summary: The summary is not meaningful.
    - WARN  | url: There was a problem validating the URL https://github.com/jianli2017/LJMenu.

Analyzed 1 podspec.

LJMenu.podspec passed validation.

```

表示这个LJMenu.podspec 验证通过，是一个符合CocoaPods规则的配置文件。

3.本地测试LJMenu.podspec文件 

打开Example工程目录Podfile文件修改下pod 的引用

```
  #pod 'LJMenu', :path => '../' # 指定路径
  pod 'LJMenu', :podspec => '../LJMenu.podspec'  # 指定podspec文件

```

然后在Example工程目录下执行pod install命令安装依赖，打开项目工程，现在可以看到库文件都被加载到Pods子项目中了

4.向Spec Repo提交podspec 

测试库文件没有问题我们就把MyLib.podspec提交到远程Spec Repo仓库中，就是本文开头说的[官方仓库][13] 在Terminal中执行 cd进入MyLib项目根目录然后，执行以下命令：

```
# pod repo push [Repo名] [podspec 文件名字]
$ pod repo push MyPrivateRepo ./LJMenu.podspec --allow-warnings

```

如果提交成功，在Terminal会输出：

```
Validating spec
 -> LJMenu (1.0.2)
    - WARN  | summary: The summary is not meaningful.
    - WARN  | url: There was a problem validating the URL https://github.com/jianli2017/LJMenu.

Updating the `MyPrivateRepo' repo

Already up-to-date.

Adding the spec to the `MyPrivateRepo' repo

 - [Update] LJMenu (1.0.2)

Pushing the `MyPrivateRepo' repo

```

表示提交成功了！这个组件库就添加到我们的私有Spec Repo中了，可以进入到~/.cocoapods/repos/MySpecs目录下查看

```
 cd ~/.cocoapods/repos/
 tree -L 3 MyPrivateRepo/
```

结果如下：

```
MyPrivateRepo/
└── LJMenu
    ├── 1.0.1
    │   └── LJMenu.podspec
    └── 1.0.2
        └── LJMenu.podspec

3 directories, 2 files

```

再去看我们的Spec Repo远端仓库 也就是[官方仓库][14]，也有了一次提交，这个podspec也已经被Push上去了。

至此，我们的这个组件库就已经制作添加完成了，使用pod search命令就可以查到我们自己的库了. 在Terminal中执行 pod search MyLib

```
-> LJMenu (1.0.2)
   A short description of LJMenu.
   pod 'LJMenu', '~> 1.0.2'
   - Homepage: https://github.com/jianli2017/LJMenu
   - Source:   https://github.com/jianli2017/LJMenu.git
   - Versions: 1.0.2, 1.0.1 [MyPrivateRepo repo]

```

## 使用制作好的Pod

在完成这一系列步骤之后，我们就可以在正式项目中使用这个私有的Pod了只需要在项目的Podfile里增加以下一行代码即可, 在正式项目的Podfile 里添加私有Spec Repo

```
#私有Spec Repo

source 'git@github.com:jianli2017/LJRepo.git' 
source 'git@github.com:CocoaPods/Specs.git'

pod 'LJMenu', '~> 1.0.2'
```

然后执行pod install，安装依赖，然后打开项目可以看到，我们自己的库文件已经出现在Pods子项目中的Pods子目录下了，而不再是Development Pods。


## 将LJMenu发布到官方仓库中

### 注册CocoaPods

首先使用pod trunk me查看自己是否注册过：如果没有下面类似的内容输出,则表示没有注册过

```
- Name:     jianli2017
- Email:    jianli2017@163.com
- Since:    February 28th, 04:01
- Pods:     None
- Sessions:
- February 28th, 04:01 - July 6th, 04:04. IP: 101.254.248.194
```

使用pod trunk register命令注册。

```
pod trunk register jianli2017@163.com 'jianli2017' --verbose
```

注册完成后，使用下面的命令，将LJMenu库的spec推送到官方仓库中。

```
pod trunk push  --allow-warnings
```
推送完成后，可以使用pod search 查看。

## tip

#### pod命令使用方法
pod的命令如果不知道怎么用，可以使用pod --help命令查看使用方法：

```
Usage:

$ pod COMMAND

CocoaPods, the Cocoa library package manager.

Commands:

+ cache         Manipulate the CocoaPods cache
+ deintegrate   Deintegrate CocoaPods from your project
+ env           Display pod environment
+ init          Generate a Podfile for the current directory
+ install       Install project dependencies according to versions from a
Podfile.lock
+ ipc           Inter-process communication
+ lib           Develop pods
+ list          List pods
+ outdated      Show outdated project dependencies
+ plugins       Show available CocoaPods plugins
+ repo          Manage spec-repositories
+ search        Search for pods
+ setup         Setup the CocoaPods environment
+ spec          Manage pod specs
+ trunk         Interact with the CocoaPods API (e.g. publishing new specs)
+ try           Try a Pod!
+ update        Update outdated project dependencies and create new Podfile.lock

Options:

--silent        Show nothing
--version       Show the version of the tool
--verbose       Show more debugging information
--no-ansi       Show output without ANSI codes
--help          Show help banner of specified command
```

通过上面可以看到pod 的所有命令。常用的有pod init、pod install、 pod update、pod lib、pod repo等等，如果对pod repo不了解，可以使用pod repo --help进一步查看使用方法。

```
Usage:

$ pod repo [COMMAND]

Manage spec-repositories

Commands:

+ add       Add a spec repo
+ lint      Validates all specs in a repo
> list      List repos
+ push      Push new specifications to a spec-repo
+ remove    Remove a spec repo
+ update    Update a spec repo

Options:

--silent    Show nothing
--verbose   Show more debugging information
--no-ansi   Show output without ANSI codes
--help      Show help banner of specified command
```

可以看出，pod repo add 、pod repo list 等命令，如果对pod repo list命令不知道如何使用，可以使用pod repo list --help命令进一步查看使用方法

```
Usage:

$ pod repo list

List the repos from the local spec-repos directory at `~/.cocoapods/repos/.`

Options:

--count-only   Show the total number of repos
--silent       Show nothing
--verbose      Show more debugging information
--no-ansi      Show output without ANSI codes
--help         Show help banner of specified command
```

上面的方法对任何的pod命令都使用，通过上面的方法我们可以学习会pod命令的使用方法。


## 参考资料

 [利用CocoaPods创建私有库](https://www.jianshu.com/p/107cc74847ab)

