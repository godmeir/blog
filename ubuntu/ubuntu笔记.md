##ubuntu使用笔记
###如何获取root账户的权限

---
###如何建立共享文件夹
---
###多个版本的jdk安装与切换
/etc/profile的设置方法对所有登录的用户都有效。
~/.bashrc只对当前用户有效。

上面两个都是配置文件，开机后，系统会先读取/etc/profile，再读~/.bashrc。
不同的用户~/.bashrc文件可以有不同的设置，而/etc/profile则是共用一个，只有root才能修改。
~/.bashrc对/etc/profile有追加覆盖的效果。

### bashrc与profile的区别（安装JDK时学习收集）
<pre>
要搞清bashrc与profile的区别，首先要弄明白什么是交互式shell和非交互式shell，什么是login shell 和non-login shell。
交互式模式就是shell等待你的输入，并且执行你提交的命令。这种模式被称作交互式是因为shell与用户进行交互。这种模式也是大多数用户非常熟悉的：登录、执行一些命令、签退。当你签退后，shell也终止了。 shell也可以运行在另外一种模式：非交互式模式。在这种模式下，shell不与你进行交互，而是读取存放在文件中的命令,并且执行它们。当它读到文件的结尾，shell也就终止了。
bashrc与profile都用于保存用户的环境信息，bashrc用于交互式non-loginshell，而profile用于交互式login shell。系统中存在许多bashrc和profile文件，下面逐一介绍：
/etc/pro此文件为系统的每个用户设置环境信息,当第一个用户登录时,该文件被执行.
并从/etc/profile.d目录的配置文件中搜集shell的设置.
/etc/bashrc:为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取。有些linux版本中的/etc目录下已经没有了bashrc文件。
~/. pro每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该
文件仅仅执行一次!默认情况下,它设置一些环境变量,然后执行用户的.bashrc文件.
~/.bashrc:该文件包含专用于某个用户的bash shell的bash信息,当该用户登录时以及每次打开新的shell时,该文件被读取.
另外,/etc/profile中设定的变量(全局)的可以作用于任何用户,而~/.bashrc等中设定的变量(局部)只能继承/etc/profile中的变量,他们是"父子"关系.

.bashrc：非登陆用户使用，如ssh，xterm等，使用exit退出的
.bash_profile：供登陆用户使用（longin），使用 logout退出。
* /etc/bashrc 存有整个系统的别名和功能； 
* /etc/profile 存有整个系统的环境参数和启动程式； 
* $HOME/.bashrc 存有用户的的别名和功能； 
* $HOME/.bash_profile 存有用户的环境参数和启动程式； 
* $HOME/.bash_logout 存有退出系统时的结束方式； 
* $HOME/.inputrc 存有主要绑定数值和其他位元数值； 
# /etc/profile
# System wide environment and startup programs 
# --整个系统环境和启动程式 
# 
# Functions and aliases go in /etc/bashrc 
# --/etc/bashhrc中的功能和别名 
# 
# This file sets the following features: 
# --这个文档设定下列功能： 
# 
# o path --路径 
# o prompts --提示符 
# o a few environment variables --几个环境变数 
# o colour ls --ls 的颜色 
# o less behaviour --设定less的功能 
# o keyboard settings --键盘设置 
# 
# Users can override these settings and/or add others in their 
# $HOME/.bash_profile 
# 用户可在$HOME/.bash_profile中取消这些设定和（或）增加其他设定
_________________________________________________________________
此处为 /etc/bashrc： 
_________________________________________________________________
# /etc/bashrc
# System wide functions and aliases 
# 整个系统的功能和别名 
# 
# Environment stuff goes in /etc/profile 
# /etc/profile中的环境参数
_________________________________________________________________
此处为 .bashrc： 
_________________________________________________________________
# $HOME/.bashrc 
# Source global definitions
# this is needed to notify the user that they are in non-login shell 
# 需要以下设定，以便通知处於不登录（non-login）外围程序（shell）中的用户
_________________________________________________________________ 

此处为.bash_profile： 
_________________________________________________________________ 

# $HOME/.bash_profile 

# User specific environment and startup programs 
# 用户特定的环境参数和启动程式 
# 
# This file contains user-defined settings that override 
# those in /etc/profile 
# 这个文档中存有用户自订的设置，可取代/etc/profile 中的数值 
# 
# Get aliases and functions 
# 设定别名和功能 
# 
1. Login Shell
A login shell means that you got the shell after authenticating to the system, usually by giving your user name and password.

Files read:
/etc/profile
~/.bash_profile, ~/.bash_login or ~/.profile: first existing readable file is read
~/.bash_logout upon logout.


2. Non-Login Shell
A non-login shell means that you did not have to authenticate to the system. For instance, when you open a terminal using an icon, or a menu item, that is a non-login shell.

Files read:
~/.bashrc
This file is usually referred to in ~/.bash_profile:
if [ -f ~/.bashrc ]; then . ~/.bashrc; fi
3。总结
如果你在用那个shell之前有login,那个就是login shell,比如tty, 如果没有,就是non-login!比如一般的xterm
</pre>