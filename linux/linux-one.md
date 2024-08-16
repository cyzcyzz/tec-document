---
title: "Linux基础入门(1)"
date: 2020-03-16T23:33:11+08:00
description: ""
draft: false
tags: []
categories: [linux]
---

## 概述

![one-one](http://cdn.oyfacc.cn/linux/linux-one.png)

&ensp;&ensp;冯诺伊曼体系结构的计算机是现在计算机系统的基石。现代计算机系统，在整体上划分为硬件系统和软件系统。硬件系统又包括内部的硬件和外部的硬件。内部硬件主要包含CPU和内存，cpu就包含传统的控制器和运算器还有寄存器这些，内存分为RAM（随机存取存储器），ROM（只读存储器）。外部设备又包含硬盘存储，输入和输出设备等。而软件主要分为系统软件和应用软件。系统软件有操作系统，数据库系统这些。应用软件主要是通用软件和专用软件，如360杀毒是应用软件，如erp系统就属于专用软件。

操作系统是基石，也是最重要的系统软件，是和硬件交互的主要渠道，也是各类应用软件的载体。Linux操作系统是主流的操作系统，是开源领域软件的杰出代表，占据服务器的操作系统的大部分市场，可以说Linux系统是非常重要的。

通常所说的linux系统，主要指带Linux内核，根据开源软件协议，各发行商将工具融合和linux一起打包成便于操作的系统，这就是不同的linux发型版的由来。linux的发型版是非常多的，而主流的主要分为如下的几个：

| 系统名称 | 代表1  | 代表2  |
| -------- | ------ | ------ |
| Fedora系 | redhat | centos |
| debian系 | ubuntu |        |

发型版系统= linux内核+GNU工具

计算机体系结构方面的知识可以参照上图，具体和详细的可以推荐参考《计算机组成原理》这本书，作者是唐朔飞，这是很多大学计算机专业课用书，专业度毋庸置疑。链接在这里[组成原理链接](https://item.jd.com/12271404.html)

### 命令总结

##### lscpu命令：查看当前机器的cpu信息

```shell
[root@ops ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    2
Core(s) per socket:    2
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Gold 6151 CPU @ 3.00GHz
Stepping:              4
CPU MHz:               3000.000
BogoMIPS:              6000.00
Hypervisor vendor:     KVM
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              25344K
NUMA node0 CPU(s):     0-3
```

##### file命令：查看文件类型

```shell
[root@ops ~]# file alertmanager 
alertmanager: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped 可执行文件
[root@ops ~]# file a.json 
a.json: ASCII text 文本文件
[root@ops ~]# file control.sh 
control.sh: Bourne-Again shell script, ASCII text executable shell文件
```

##### hexdump命令：查看二进制文件的内容

```shell
[root@ops bin]# hexdump -c -n 100 redis-cli    
0000000 177   E   L   F 002 001 001  \0  \0  \0  \0  \0  \0  \0  \0  \0
0000010 002  \0   >  \0 001  \0  \0  \0   4   e   @  \0  \0  \0  \0  \0
0000020   @  \0  \0  \0  \0  \0  \0  \0   ` 345  \a  \0  \0  \0  \0  \0
0000030  \0  \0  \0  \0   @  \0   8  \0  \t  \0   @  \0   &  \0   %  \0
0000040 006  \0  \0  \0 005  \0  \0  \0   @  \0  \0  \0  \0  \0  \0  \0
0000050   @  \0   @  \0  \0  \0  \0  \0   @  \0   @  \0  \0  \0  \0  \0
0000060 370 001  \0  \0                                                
0000064
-c: 16进制格式查看
-n 100: 显示前100位
```

##### lsblk: 查看所有的块设备

```shell
[root@ops bin]# lsblk 
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  500G  0 disk 
└─vdb1 253:17   0  500G  0 part 
vda    253:0    0  100G  0 disk 
└─vda1 253:1    0  100G  0 part /
名字 主设备号:次设备号 移动设备 大小 只读 类型 挂载点
移动设备 标记为1
只读设备 标记为1
```

**注意观察size的大小，size小于1K的基本是扩展分区，关于分区，后续会讲到**

##### sha1sum：查看哈希校验码

```shell
[root@ops bin]# sha1sum redis-cli 
f03302d1760eba294b8996e59b9b56f519499869  redis-cli
```

##### ldd：查看命令使用的链接库

```shell
[root@ops bin]# ldd redis-cli 
        linux-vdso.so.1 =>  (0x00007ffd40ba2000)
        libm.so.6 => /lib64/libm.so.6 (0x00007f68af811000)
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f68af60d000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f68af3f1000)
        libc.so.6 => /lib64/libc.so.6 (0x00007f68af024000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f68afb13000)
```

##### dd: 转换和复制一个文件

```shell
[root@ops ~]# dd if=/dev/zero of=f1 bs=1M count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB) copied, 0.0613801 s, 1.7 GB/s
if: 文件源
of: 目的文件
bs: 单个块大小
count: 总共写入多少块
总大小=bs乘count
```

### 其他相关总结

cpu指令集：x86架构主要使用CISC复杂指令集，价格相对便宜，还有 RISC精简指令集和EPIC并行指令代码，主要用于专用的服务器，价格相对高昂

bit和byte：位和字节，b和B，1B=8b，也就是一字节等于8位

#### Linux哲学思想

一切都是一个文件

小型，单一用途的程序

链接程序，共同完成复杂的操作

避免令人困惑的用户界面

配置数据存储在文本中