---
title: set命令
date: 2018-07-20 12:07:12
tags: set
categories: linux命令
toc: true
---

export命令设置或显示环境变量。

<!--more-->

## Linux export 命令

功能说明：设置或显示环境变量。（比如我们要用一个命令，但这个命令的执行文件不在当前目录，这样我们每次用的时候必须指定执行文件的目录，麻烦，在代码中先执行export，这个相当于告诉程序，执行某某东西时，需要的文件或什么东东在这些目录里）

语　　法：export [-fnp][变量名称]=[变量设置值]

补充说明：在shell中执行程序时，shell会提供一组环境变量。 export可新增，修改或删除环境变量，供后续执行的程序使用。export的效力仅及于该此登陆操作。

参　　数：

-f 　代表[变量名称]中为函数名称。

　-n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
　
　-p 　列出所有的shell赋予程序的环境变量。
　
　一个变量创建时，它不会自动地为在它之后创建的shell进程所知。而命令export可以向后面的shell传递变量的值。当一个shell脚本调用并执行时，它不会自动得到原为脚本（调用者）里定义的变量的访问权，除非这些变量已经被显式地设置为可用。export命令可以用于传递一个或多个变量的值到任何后继脚本。     ----《UNIX教程》
　
　
　
　在 linux 里设置环境变量的方法 （ export PATH ）
　
　一般来说，配置交叉编译工具链的时候需要指定编译工具的路径，此时就需要设置环境变量。例如我的mips-linux-gcc编译器在“/opt/au1200_rm /build_tools/bin”目录下，build_tools就是我的编译工具，则有如下三种方法来设置环境变量：
　
　1、直接用export命令：
　#export PATH=$PATH:/opt/au1200_rm/build_tools/bin
　查看是否已经设好，可用命令export查看：
　
　
　
　[root@localhost bin]#export
　declare -x BASH_ENV="/root/.bashrc"
　declare -x G_BROKEN_FILENAMES="1"
　declare -x HISTSIZE="1000"
　declare -x HOME="/root"
　declare -x HOSTNAME="localhost.localdomain"
　declare -x INPUTRC="/etc/inputrc"
　declare -x LANG="zh_CN.GB18030"
　declare -x LANGUAGE="zh_CN.GB18030:zh_CN.GB2312:zh_CN"
　declare -x LESSOPEN="|/usr/bin/lesspipe.sh %s"
　declare -x LOGNAME="root"
　declare -x LS_COLORS="no=00:fi=00:di=01;34:ln=01;36:pi=40;33:so=01;35:bd=40;33;01:cd=40;33;01:or=01;05;37;41:mi=01;05;37;41:ex=01;32:*.cmd=01;32:*.exe=01;32:*.com=01;32:*.btm=01;32:*.bat=01;32:*.sh=01;32:*.csh=01;32:*.tar=01;31:*.tgz=01;31:*.arj=01;31:*.taz=01;31:*.lzh=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.gz=01;31:*.bz2=01;31:*.bz=01;31:*.tz=01;31:*.rpm=01;31:*.cpio=01;31:*.jpg=01;35:*.gif=01;35:*.bmp=01;35:*.xbm=01;35:*.xpm=01;35:*.png=01;35:*.tif=01;35:"
　declare -x MAIL="/var/spool/mail/root"
　declare -x OLDPWD="/opt/au1200_rm/build_tools"
　declare -x PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/X11R6/bin:/root/bin:/opt/au1200_rm/build_tools/bin"
　declare -x PWD="/opt/au1200_rm/build_tools/bin"
　declare -x SHELL="/bin/bash"
　declare -x SHLVL="1"
　declare -x SSH_ASKPASS="/usr/libexec/openssh/gnome-ssh-askpass"
　declare -x SSH_AUTH_SOCK="/tmp/ssh-XX3LKWhz/agent.4242"
　declare -x SSH_CLIENT="10.3.37.152 2236 22"
　declare -x SSH_CONNECTION="10.3.37.152 2236 10.3.37.186 22"
　declare -x SSH_TTY="/dev/pts/2"
　declare -x TERM="linux"
　declare -x USER="root"
　declare -x USERNAME="root"
　
　可以看到灰色部分有设置的路径，说明环境变量已经设好，PATH里面已经有了我要加的编译器的路径。
　
　2、修改profile文件：
　#vi /etc/profile
　在里面加入:
　export PATH="$PATH:/opt/au1200_rm/build_tools/bin"
　
　3. 修改.bashrc文件：
　# vi /root/.bashrc
　在里面加入：
　export PATH="$PATH:/opt/au1200_rm/build_tools/bin"
　
　后两种方法一般需要重新注销系统才能生效，最后可以通过echo命令测试一下：
　# echo $PATH
　看看输出里面是不是已经有了 /my_new_path这个路径了。
　
　另有：4. 修改/etc/re.local文件：
　# vi /etc/re.local
　在里面加入：
　export PATH="$PATH:/opt/au1200_rm/build_tools/bin"
　
　
　-----------------------------------------------------------------------------------------------------------------------
　
　“/bin”、“/sbin”、“ /usr/bin”、“/usr/sbin”、“/usr/local/bin”等路径已经在系统环境变量中了，如果可执行文件在这几个标准位置，在终端命令行输入该软件可执行文件的文件名和参数(如果需要参数)，回车即可。
　
　　　如果不在标准位置，文件名前面需要加上完整的路径。不过每次都这样跑就太麻烦了，一个“一劳永逸”的办法是把这个路径加入环境变量。命令 export $PATH="路径”(或“PATH=$PATH:路径”) ($PATH为环境变量名，如DVSDK；调用时用$DVSDK)可以把这个路径加入环境变量，但是退出这个命令行就失效了。要想永久生效，需要把这行添加到环境变量文件里。有两个文件可选：“/etc/profile”和用户主目录下的“.bash_profile”，“/etc/profile”对系统里所有用户都有效，用户主目录下的“.bash_profile”只对这个用户有效。
　　　
　　　　　export $PATH="$PATH:路径1:路径2:...:路径n” （或“PATH=$PATH:路径1:路径2:...:路径n"　），意思是可执行文件的路径包括原先设定的路径，也包括从“路径1”到“路径n”的所有路径。当用户输入一个一串字符并按回车后，shell会依次在这些路径里找对应的可执行文件并交给系统核心执行。那个“$PATH”表示原先设定的路径仍然有效，注意不要漏掉。某些软件可能还有“PATH”以外类型的环境变量需要添加，但方法与此相同，并且也需要注意“$”。
　　　　　
　　　　　　　注意，与DOS/Window不同，UNIX类系统环境变量中路径名用冒号分隔，不是分号。另外，软件越装越多，环境变量越添越多，为了避免造成混乱，建议所有语句都添加在文件结尾，按软件的安装顺序添加。
　　　　　　　
　　　　　　　　　格式如下()：
　　　　　　　　　
　　　　　　　　　　　# 软件名-版本号
　　　　　　　　　　　
　　　　　　　　　　　　　PATH=$PATH:路径1:路径 2:...:路径n
　　　　　　　　　　　　　
　　　　　　　　　　　　　　　其他环境变量=$其他环境变量:...
　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　在“profile”和“.bash_profile”中，“#”是注释符号，写在这里除了视觉分隔外没有任何效果。
　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　设置完毕，注销并重新登录，设置就生效了。如果不注销，直接在shell里执行这些语句，也能生效，但是作用范围只限于执行了这些语句的shell。
　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　相关的环境变量生效后，就不必老跑到软件的可执行文件目录里去操作了。
　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　-----------------------------------------------------------------------------------------------------------------------
　　　　　　　　　　　　　　　　　　　　　
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　 
　　　　　　　　　　　　　　　　　　　　　　 
　　　　　　　　　　　　　　　　　　　　　　 执行一个脚本时，会先开启一个子shell环境（不知道执行其它程序是不是这样），然后将父shell中的所有系统环境变量复制过来，这个脚本中的语句就在子shell中执行。（也就是说父shell的环境变量在子shell中可以调用，但反过来就不行，如果在子shell中定义了环境变量，则只对该shell或者它的子shell有效，当该子shell结束时，也可以理解为脚本执行完时，变量消失。）为了证明这一点，请看脚本内容：
　　　　　　　　　　　　　　　　　　　　　　 　　test=’value’
　　　　　　　　　　　　　　　　　　　　　　 　　　　export test
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　这样的脚本执行完后，test实际上是不存在的。接着看下面的：
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　test=’value’
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　export test
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　　　bash
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　　　　　这里在脚本最后一行再开一个子shell，该shell应该是脚本文件所在shell的子shell，这个脚本执行完后，是可以看到test这个变量的，因为现在是处于它的子shell中，当用exit退出子shell后，test变量消失。
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　　　　　　　如果用source对脚本进行执行时，如果不加export，就不会在子shell中看到这个变量，因为它还不是一个系统环境变量呀，如脚本内容是：
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　　　　　　　　　test=’value’
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　　　　　　　　　　　用source执行后，在shell下是能看到这个变量，但再执行bash开一个子shell时，test是不会被复制到子shell中的，因为执行脚本文件其实也是在一个子shell中运行，所以我再建另一个脚本文件执行时，是不会输入任何东西的，内容如：echo $test。所以这点特别注意了，明明在提示符下可以用echo $test输出变量值，为什么把它放进脚本文件就不行了呢？
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　　　　　　　　　　　　　所以得出的结论是：1、执行脚本时是在一个子shell环境运行的，脚本执行完后该子shell自动退出；2、一个shell中的系统环境变量才会被复制到子shell中（用export定义的变量）；3、一个shell中的系统环境变量只对该shell或者它的子shell有效，该shell结束时变量消失（并不能返回到父shell中）。3、不用export定义的变量只对该shell有效，对子shell也是无效的。
　　　　　　　　　　　　　　　　　　　　　　 　　　　　　　　　　　　　　　　　　　　　　　　后来根据版主的提示，整理了一下贴子：为什么一个脚本直接执行和用source执行不一行呢？这也是我自己碰到的一个问题。manual原文是这样的：Read and execute commands from filename in the current shell environment and return the exit status of the last command executed from filename.明白了为什么不一样了吧？直接执行一个脚本文件是在一个子shell中运行的，而source则是在当前shell环境中运行的。
