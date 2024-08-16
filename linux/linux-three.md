---
title: "Linux基础入门(3)"
date: 2020-06-06T12:09:40+08:00
description: ""
draft: false
tags: []
categories: [linux]
---

# 命令简介

通常在命令行执行的操作，我们称之为命令。linux中，命令分为两种类型，内部命令和外部命令。

- 内部命令：随着bash加载近内存的命令
- 外部命令：后来安装的命令或者存储在磁盘上的命令

那外部命令是怎么被查找到的呢？其实是因为系统中存在PATH的环境变量，bash会按照PATH路径来查找命令。

```shell
[root@ops ~]# echo $PATH
/root/.pyenv/bin:/home/jdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/bin/:/root/bin 
# 可以看到PATH变量其实存储的是以冒号分割的一个个路径，查找的过程中是按照顺序依次查找，第一个查找到的生效
```

如何查看一个命令的类型？可以使用type命令查看

```shell
[root@ops ~]# type cd
cd is a shell builtin
[root@ops ~]# type python
python is /usr/bin/python
# 这里可以很清晰的看到内建命令直接显示的是内建，外部显示的是他的安装路径
```

内部命令查看帮助手册

```shell
[root@ops ~]# help cd
cd: cd [-L|[-P [-e]]] [dir]
    Change the shell working directory.
    
    Change the current directory to DIR.  The default DIR is the value of the
    HOME shell variable.
```

列出全部的内建命令

```shell
[root@ops ~]# enable
enable .
enable :
enable alias
enable cd
enable hash
enable help
enable history
enable kill
... # 这里仅展示了一部分，
```

但其实有部分命令是内建命令但同时也存在一个外部命令，为了防止内建被禁用时的使用。如：echo

```shell
[root@ops ~]# type echo
echo is a shell builtin # 非常清晰的看到时内建命令
[root@ops ~]# enable -n echo # 这里禁用了echo命令
[root@ops ~]# type echo
echo is /usr/bin/echo 
# 此时echo成为了一个外部命令

```

通常我们要查找一个命令的详细路径，一般有两个命令

- which
- whereis

 这两个命令的区别就是whereis本身会显示更详细的信息

```shell
[root@ops ~]# which python
/usr/bin/python
[root@ops ~]# whereis python
python: /usr/bin/python3.6m /usr/bin/python2.7 /usr/bin/python /usr/bin/python2.7-config /usr/bin/python3.6 /usr/lib/python2.6 /usr/lib/python2.7 /usr/lib/python3.6 /usr/lib64/python2.7 /usr/lib64/python3.6 /etc/python /usr/local/bin/python3.7m-config /usr/local/bin/python3.7-config /usr/local/bin/python3.7m /usr/local/bin/python3.7 /usr/include/python3.6m /usr/include/python2.7 /usr/share/man/man1/python.1.gz
```

这里存在一个问题，我们每次要使用一个外部命令的时候，都要查找一遍PATH吗？其实试试不需要的，当第一次找到的时候，这个命令的路径就会被存储在内存中，通常我们称之为hashed

```shell
[root@ops ~]# hash
hits    command
   1    /usr/bin/whereis
   左侧时命中次数 ，右侧显示的是命令的绝对路径
   
[root@ops ~]# hash -d python # 删除python命令的hash
[root@ops ~]# hash -r # 清除全部的hash
```

每次退出终端后hash会全部清空

命令本身可以定义别名

```shell
[root@ops ~]# alias h=hostname #定义一个别名h，命令为hostname
[root@ops ~]# h
ops
[root@ops ~]# alias  # 列出全部的别名
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias h='hostname'
alias k='kubectl'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

别名的优先级最高，大于内建命令，大于外部命令

# 时间和时区

常用命令总结

```shell
[root@ops ~]# date # 显示当前机器的时间
Sat Jun  6 13:36:27 CST 2020
[root@ops ~]# date "+%F" # 格式化当前时间
2020-06-06 
[root@ops ~]# date -d "-10 days" "+%F" # 求十天前的日期
2020-05-27
```

硬件时间显示

```shell
[root@ops ~]# hwclock 
Sat 06 Jun 2020 01:39:46 PM CST  -0.037121 seconds
[root@ops ~]# clock
Sat 06 Jun 2020 01:39:52 PM CST  -0.322863 seconds
# 上边的两个命令都用来显示硬件时间
```

时区的相关设置

```shell
[root@ops ~]# timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
............ # 列出全部的时区
[root@ops ~]# timedatectl set-timezone Africa/Accra # 设置时区
```

日历的相关显示,在Linux中使用cal命令来显示日历

```shell
[root@ops ~]# cal # 显示现在的日期 这里今天的日志在终端是加了底色的，因为复制出来所有没有了
      June 2020     
Su Mo Tu We Th Fr Sa
    1  2  3  4  5  6
 7  8  9 10 11 12 13
14 15 16 17 18 19 20
21 22 23 24 25 26 27
28 29 30
[root@ops ~]# cal 2020 # 显示2020年的全部月份
                               2020                               

       January               February                 March       
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
          1  2  3  4                      1    1  2  3  4  5  6  7
 5  6  7  8  9 10 11    2  3  4  5  6  7  8    8  9 10 11 12 13 14
12 13 14 15 16 17 18    9 10 11 12 13 14 15   15 16 17 18 19 20 21
19 20 21 22 23 24 25   16 17 18 19 20 21 22   22 23 24 25 26 27 28
26 27 28 29 30 31      23 24 25 26 27 28 29   29 30 31 。。。。。。。
```

