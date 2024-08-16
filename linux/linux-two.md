---
title: Linux 4
date: 2020-07-08T14:36:55+08:00
description: ""
draft: true
tags: []
categories:
  - linux
---

### 概述

熟悉windows系统的同学都知道，win系统本身是存在分区盘符的概念的，如：C盘一般是存放我们的系统文件。其实任何操作系统都是有文件系统的，这样才可以存放我们的文件，linux也不例外。但是Linux和windows组织的方式是不太一样的，在linux中第一个分区称之为根分区，采用符号“/”来表示，下面一张图可以看到基本的分区结构，这里的分区结构展示的MBR类型的分区,GPT另作讨论：

![disk-part](http://cdn.oyfacc.cn/linux-diskpart.jpg)

在mbr类型的分区中，主分区+扩展分区=4个分区，而且扩展分区最多有一个，不直接存储数据，而是划分为逻辑分区后再存储数据，这里的分区主要是针对单个磁盘说的。MBR类型的分区最大支持的磁盘大小为2T，至于为什么，后面会详细讲述。其实生产环境中用到扩展分区真的很少，大多数都是一个硬盘上就存在一个分区。逻辑分区的号码一般是从5开始的。

##### lsblk命令： 查看全部磁盘和磁盘的分区

```shell
[root@ops ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  500G  0 disk 
└─vdb1 253:17   0  500G  0 part 
vda    253:0    0  100G  0 disk 
└─vda1 253:1    0  100G  0 part /
上篇文章有解析
```

##### Linux用户

Linux的账户类型分为两类：1，管理账户。2，普通用户

管理账户用于管理操作系统，普通用户一般权限有限，只能做一些仅有的操作，像查看日志这些基本的功能

其中`root`账户比较特别，是超级管理员，拥有全部的权限，危险性很高，轻易不要暴露

系统账号的ID，在centos6系统中，普通账户的ID从500开始，500之前是管理账户，centos7普通用户从1000开始，之前是管理账户，而root一般是0号账户

```shell
[root@ops ~]# echo $UID
0
用于查看当前登陆账户的ID
```

##### 终端

终端是指我们当前操作的页面，这个页面可以是命令行，也可以是桌面。终端分为两种：1，物理终端，2，虚拟终端

物理终端：一般是说/dev/consol，也就是通常所说的连接机器的显示器和键盘

虚拟终端：一般指远程连接过来的终端,/de/pts/0

模拟终端：指和我们交互的命令行窗口，一般通过`ctrl+alt+F(1~6)`切换模拟终端 /dev/tty

```linux
[root@ops ~]# tty
/dev/pts/0
查看所有的终端
```

##### 运行级别

在linux中存在运行级别的概念

0级别：关机

1级别：单用户模式

3级别：命令行模式

5级别：桌面模式

6级别：重启

```shell
[root@ops ~]# init (0 1 3 5 6)
根据上述的解释分别的操作
```

##### shell

linux中和人命令行交互的程序叫shell，也就是壳，他是命令解释器+高级语言，也就是说，shell是一种变成语言，也是一种交互方式。shell介于系统调用外部，和库函数同级别

shell既然是一种交互的方式，那就存在很多种实现，现在基本上最流行的是bash，在mac上比较流行zsh

```shell
[root@ops ~]# echo $SHELL
/bin/bash
查看当前使用的shell类型
[root@ops ~]# cat /etc/shells 
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
查看当前系统支持的shell类型
```

其实有一种特别的shell`/sbin/nologin`可以看到是不可登陆shell，这种shell主要用于应用程序的运行，不需要交互的登陆终端，更安全。

```shell
[root@ops ~]# getent passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
。。。。。
获取全部的用户信息
[root@ops ~]# getent passwd root
root:x:0:0:root:/root:/bin/bash
获取单个用户的详细信息
```

##### who:显示用户

```shell
[root@ops ~]# who
root     pts/0        2020-03-27 23:01 (10.10.20.40)
用户    虚拟终端号    登陆时间          IP地址
[root@ops ~]# who -b
         system boot  2019-08-13 14:28
         显示系统启动时间
```

##### whom: 显示用户

```shell
[root@ops ~]# whoami
root
```

##### iconv:win系统文件转换linux格式，不常用

````shell
iconv -f gb2312 file1 -o file2
````

##### 其他的杂项

想要修改登录后的提示信息可以修改`/etc/motd`文件




#### screen

在日常运维工作中，我们会遇到长线执行的任务，但是这个任务需要一直占用一个终端，如copy一个大文件到其他机器，如下载一个大文件到本地。。。

`screen`命令非常的有用，主要用于时间较长的任务剥离终端继续运行。

在linux系统中，任何进程的创建都是存在一个父进程来负责管理子进程的，一般情况下，你在当前shell运行的进程，父进程就是这个shell，当这个shell退出，子进程也会退出，这样任务就会停止。

选项总结：

- -S 新建一个终端 -S 666
- -ls 列出全部终端
- -X 加入一个存在的终端 -X 666 加入666
- 快捷键 Crtl +a+d 剥离终端
- -r 回到上一次剥离的终端

#### echo

回显命令，用于输出变量值或者重定向输出到指定位置，和其他语言的print相似

选项总结：

- -n 不自动换行
- -e 启用字符解释

示例：

```shell
[root@ops ~]# echo -n hhh
hhh[root@ops ~]# echo hhh   
hhh
```

关于单引号和双引号，其实是强引用和弱引用的区别，在强引用的状态下，全部字符都被翻译成字符串，弱引用变量会被翻译成对应的值

#### 命令帮助

这个话题涉及几个常见的命令，`whatis`,`command --help`,`man`,`/usr/share/doc`

##### whatis

这个命令是通过系统自带的一个数据库来查询帮助的，这个数据库存储了命令的简单使用和介绍

使用`makewhatis`命令可以创建数据库

centos7 使用`mandb`

数据库位于`/var/cache/man/os6/mandb`

##### man

man帮助是每个linux系统使用者都很熟悉的命令，它包含了每个命令的使用手册。

man命令章节介绍：

- 第一章：用户命令
- 第二章：系统调用
- 第三章：C库调用
- 第四章：设备文件
- 第五章：配置格式
- 第六章：游戏
- 第七章：杂项
- 第八章：管理命令
- 第九章：linux内核API

