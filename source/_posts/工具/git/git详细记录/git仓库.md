
---
title: Git仓库(转载)
date: 2018-2-12 12:18:26
categories: Tool
tags: Git
---

本文全部复制[Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000),自己已经理解的内容只是简单复制总结部分，没有理解的就全部粘贴，方便以后进一步学习、理解。

<!--more-->

## 添加远程库

要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git；

关联后，使用命令git push -u origin master第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改；

分布式版本系统的最大好处之一是在本地工作完全不需要考虑远程库的存在，也就是有没有联网都可以正常工作，而SVN在没有联网的时候是拒绝干活的！当有网络的时候，再把本地提交推送一下就完成了同步，真是太方便了！

## 克隆远程仓库

```
git clone git@github.com:michaelliao/gitskills.git
```

## 使用码云

使用GitHub时，国内的用户经常遇到的问题是访问速度太慢，有时候还会出现无法连接的情况（原因你懂的）。

如果我们希望体验Git飞一般的速度，可以使用国内的Git托管服务——[码云][1]（[gitee.com][2]）。

和GitHub相比，码云也提供免费的Git仓库。此外，还集成了代码质量检测、项目演示等功能。对于团队协作开发，码云还提供了项目管理、代码托管、文档管理的服务，5人以下小团队免费。

码云的免费版本也提供私有库功能，只是有5人的成员上限。

使用码云和使用GitHub类似，我们在码云上注册账号并登录后，需要先上传自己的SSH公钥。选择右上角用户头像 -> 菜单“修改资料”，然后选择“SSH公钥”，填写一个便于识别的标题，然后把用户主目录下的.ssh/id_rsa.pub文件的内容粘贴进去：

![gitee-add-ssh-key](http://of685p9vy.bkt.clouddn.com/git/repositorygitee-add-ssh-key.jpg)

点击“确定”即可完成并看到刚才添加的Key：

![gitee-key](http://of685p9vy.bkt.clouddn.com/git/repositorygitee-key.jpg)

如果我们已经有了一个本地的git仓库（例如，一个名为learngit的本地库），如何把它关联到码云的远程库上呢？

首先，我们在码云上创建一个新的项目，选择右上角用户头像 -> 菜单“控制面板”，然后点击“创建项目”：

![gitee-new-repo](http://of685p9vy.bkt.clouddn.com/git/repositorygitee-new-repo.jpg)

项目名称最好与本地库保持一致：

然后，我们在本地库上使用命令git remote add把它和码云的远程库关联：

```
git remote add origin git@gitee.com:liaoxuefeng/learngit.git

```

> 小提示 ： origin 远程仓库也就是url对应的仓库 在本地的别名。

如果在使用命令git remote add时报错：

```
git remote add origin git@gitee.com:liaoxuefeng/learngit.git
fatal: remote origin already exists.

```

```
git remote -v
origin    git@github.com:michaelliao/learngit.git (fetch)
origin    git@github.com:michaelliao/learngit.git (push)

```

我们可以删除已有的GitHub远程库：

```
git remote rm origin

```

```
git remote add origin git@gitee.com:liaoxuefeng/learngit.git

```

```
git remote -v
origin    git@gitee.com:liaoxuefeng/learngit.git (fetch)
origin    git@gitee.com:liaoxuefeng/learngit.git (push)

```

有的小伙伴又要问了，一个本地库能不能既关联GitHub，又关联码云呢？

答案是肯定的，因为git本身是分布式版本控制系统，可以同步到另外一个远程库，当然也可以同步到另外两个远程库。

使用多个远程库时，我们要注意，<font color=red>git给远程库起的默认名称是origin</font>，如果有多个远程库，我们需要用不同的名称来标识不同的远程库。

仍然以learngit本地库为例，我们先删除已关联的名为origin的远程库：

```
git remote rm origin

```

```
git remote add github git@github.com:michaelliao/learngit.git

```

接着，再关联码云的远程库：

```
git remote add gitee git@gitee.com:liaoxuefeng/learngit.git

```

现在，我们用git remote -v查看远程库信息，可以看到两个远程库：

```
git remote -v
gitee    git@gitee.com:liaoxuefeng/learngit.git (fetch)
gitee    git@gitee.com:liaoxuefeng/learngit.git (push)
github    git@github.com:michaelliao/learngit.git (fetch)
github    git@github.com:michaelliao/learngit.git (push)

```

```
git push github master

```

```
git push gitee master

```

![multi-remote](http://of685p9vy.bkt.clouddn.com/git/repositorymulti-remote.jpg)

码云也同样提供了Pull request功能，可以让其他小伙伴参与到开源项目中来。你可以通过Fork我的仓库：[https://gitee.com/liaoxuefeng/learngit][3]，创建一个your-gitee-id.txt的文本文件， 写点自己学习Git的心得，然后推送一个pull request给我，这个仓库会在码云和GitHub做双向同步。


> 小贴士 ：git fetch 和git pull的区别
> 
> * fetch 命令只是将远端的数据拉到本地仓库，并不自动合并到当前工作分支
> * pull 若某个分支用于跟踪某个远端仓库的分支，可以使用 git pull 命令自动抓取数据下来，然后将远端分支自动合并到本地仓库中当前分支


## 参考

* [Git-基础-远程仓库的使用](https://git-scm.com/book/zh/v1/Git-基础-远程仓库的使用)