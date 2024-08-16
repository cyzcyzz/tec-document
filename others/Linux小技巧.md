# Linux小技巧

[TOC]

## 一：SHELL

### 1.1：快速清空文件

```shell
#最简单，使用shell重定向null到文件来清空
> filename 
#/dev/null设备是一个特殊设备，会吞噬任何给它的输入内容。仍然是基于shell重定向
cat /dev/null > filename 
#使用dd命令，if指的输入，of指的输出
dd if=/dev/null of=filename 
```

### 1.2：Bang(!)命令- 快速执行相关命令

```shell
# 执行最近以ssh开头的命令
$ !ssh
# 仅仅打印最近以ssh开头的命令，不执行
$ !ssh:p

#执行上一条命令
$ !!

# 上一条命令的所有参数
$ !*

# !$ 表示最近一次命令的最后一个参数
$ vim /root/sniffer/src/main.c
$ mv !$ !$.bak
# 相当于
$ mv /root/sniffer/src/main.c /root/sniffer/src/main.c.bak
```

### 1.3：查找历史命令

```shell
Ctrl – p ：显示当前命令的上一条历史命令
Ctrl – n ：显示当前命令的下一条历史命令
Ctrl – r ：搜索历史命令，随着输入会显示历史命令中的一条匹配命令，Enter键执行匹配命令；ESC键在命令行显示而不执行匹配命令。
Ctrl – g ：从历史搜索模式（Ctrl – r）退出。
示例：
ssh uer@centos
#……然后执行其他若干命令
#使用 Ctrl+R 查找 centos 或者 ssh 等能唯一定位的内容，快速查找该命令
(reverse-i-search)`centos': ssh root@centos
```

### 1.4：使用cd-在目录中移动

shell中通过 <font color=violetred>`cd -`</font> 命令回到之前目录 ， <font color=violetred>`cd -`</font> 中  <font color=violetred>`-`</font>  相当于环境变量$OLDPWD

```shell
$ pwd
/home/magedu
$ cd /tmp
$ pwd
/tmp
$ cd -
/home/magedu
```

### 1.5：使用 pushd，popd 和 dirs在目录中移动

<font color=violetred>`pushd`</font>  和  <font color=violetred>`popd`</font>  是对一个目录栈进行操作，而  <font color=violetred>`dirs`</font>  是显示目录栈的内容。而目录栈就是一个保存目录的栈结构，该栈结构的顶端永远都存放着当前目录（这里点从下面可以进一步看到）。

- **dirs**
  
  常用参数
  
  | 选项  | 含义                     |
  | --- | ---------------------- |
  | -p  | 每行显示一条记录               |
  | -v  | 每行显示一条记录，同时展示该记录在栈中的内容 |
  | -c  | 清空目录栈                  |

- **pushd**
  
  每次pushd命令执行完成之后，默认都会执行一个dirs命令来显示目录栈的内容。pushd的用法主要有如下几种：
  
  1. pushd 目录
     
     pushd后面如果直接跟目录使用，会切换到该目录并且将该目录置于目录栈的栈顶。(时时刻刻都要记住，目录栈的栈顶永远存放的是当前目录。如果当前目录发生变化，那么目录栈的栈顶元素肯定也变了；反过来，如果栈顶元素发生变化，那么当前目录肯定也变了。)下面是一个例子：
  
  ```shell
  
  ```

- **popd**
  
  ```shell
  
  ```

### 1.6：计算程序运行时间

```shell
#一些程序，我们希望能知道它的运行时间。利用time命令帮我们计算，例如：
$ time ./fobar 30
the 30 result is 832040
real 0m0.088s
user 0m0.084s
sys 0m0.004s
```

`real`、`user`  和  `sys`  三个参数的含义

- `real`：表示的钟表时间，也就是从程序执行到结束花费的时间；
- `user`：表示运行期间，cpu 在用户空间所消耗的时间；
- `sys`：表示运行期间，cpu 在内核空间所消耗的时间；

由于 `user` 和 `sys` 只统计 cpu 消耗的时间，程序运行期间会调用 sleep 发生阻塞，也可能会等待网络或磁盘 IO，都会消耗大量时间。因此对于类似情况，<u>`real` 的值就会大于其它两项之和</u>。

如果程序在多个 cpu 上并行，那么 `user` 和 `sys` 统计时间是多个 cpu 时间，实际消耗时间 `real` 很可能就比其它两个之和要小了。

### 1.7：删除乱码文件

Linux 系统中，会经常碰到名称乱码的文件。想要删除它，却无法通过键盘输入名字，有时候复制粘贴乱码名称，终端可能识别不了，该怎么办？

```shell
# -i 显示 inode 号
$ ls  -i
138957 a.txt  138959 T.txt  132395 ڹ��.txt

# -inum 指定文件 inode 号
$ find . -inum 132395 -exec rm {} \;
```

### 1.8：查看ASCII码

有时候需要使用ASCII码，不用百度谷歌查找，直接在linux命令输入 `man ascii` 

```shell
$ man ascii
```

centos7默认未支持，使用命令  ` yum install -y man-page`  后可用。

### 1.9：动态实时查看日志

如果想在日志中出现 `Failed` 等信息时立刻停止 tail 监控，可以通过如下命令来实现：

```shell
$ tail -f test.log | sed '/Failed/ q'
```

### 1.10：ps查看进程的特定字段

```shell
#通过 etime 获取该进程的运行时间，可以很直观地看到，进程运行了 13 个小时 38分钟
#关于字段名称格式，可以通过 man ps 查看，大概在601行，或者 /^STANDARD FORMAT SPECIFIERS$ 快速查找。
$ ps -p 5293 -o etimes, etime
ELAPSED     ELAPSED
  49098    13:38:18
```

### 1.11：快速生成大文件、制作系统盘、安全擦除硬盘数据

```shell
# 生成一个 1G 的文件
$ dd if=/dev/zero of=file.img bs=1M count=1024

# 使用 /dev/urandom 生成随机数据，将生成的数据写入 sda 硬盘中，相当于安全的擦除了硬盘数据。
$ dd if=/dev/urandom of=/dev/sda

# sdb 可以是 U 盘，也可以是普通硬盘
$ dd if=ubuntu-server-amd64.iso if=/dev/sdb
```

### 1.12：命令不保存进history

有时候，执行的命令中包含敏感信息。在所要执行的命令前，加一个**空格**，那这条命令就不会被 `history` 保存到历史记录，新功能？

```shell
#Ubuntu 20.04.2 LTS 可行
bash --version
GNU bash, version 5.0.17(1)-release (x86_64-pc-linux-gnu)

#macOS Monterey 可行
/bin/zsh  --version
zsh 5.8 (x86_64-apple-darwin21.0)

#CentOS 7.9 不可行
bash --version
GNU bash, version 4.2.46(2)-release (x86_64-redhat-linux-gnu)

#CentOS 8.5 不可行
bash --version
GNU bash, version 4.4.20(1)-release (x86_64-redhat-linux-gnu)
```

### 1.13：花括号自动扩展字符串

用花括号括起来的字符串用逗号连接，可以自动扩展；花括号中的每个字符都可以和之后（或之前）的字符串进行组合拼接。

```shell
$ echo {one,two,three}file
onefile twofile threefile

$ echo {one,two,three}{1,2,3}
one1 one2 one3 two1 two2 two3 three1 three2 three3

# 给 file 复制一个叫做 file.bak 的副本
$ cp /very/long/path/file{,.bak}

# 删除 file1.txt file3.txt file5.txt
$ rm file{1,3,5}.txt

# 将所有 .c 和 .cpp 为后缀的文件移入 src 文件夹
$ mv *.{c,cpp} src/
```

### 1.14：环境变量CDPATH简化路径输入

在常用的bash shell中，支持一个环境变量CDPATH，通过声明它，从而达到简化输入完整路径的目的。

比如常去 /tmp 目录下查找。

```shell
# cd 命令将在 ~ 目录和 /tmp目录扩展搜索

saw@kvm:~$ export CDPATH='~:/tmp'

saw@kvm:/home$ pwd
/home
saw@kvm:/home$ cd Documents/
/home/saw/Documents
saw@kvm:~/Documents$ cd mysql
/tmp/mysql
saw@kvm:/tmp/mysql$
```

### 1.15：watch监测命令执行

watch可以监测一个命令的运行结果，省得一遍遍的手动运行。

格式： `watch [参数] [命令]`

参数：`-n 或者 --interval`  默认2秒执行一次

​           `-d 或者 --differences` 高亮显示变化的区域

```shell
#每隔一秒高亮显示网络链接数的变化
watch -n 1 -d netstat -ant
```

### 1.16：curl 获取公网IP

服务器或虚拟机上配置的通常是内网 IP 地址。在外部交互时，有时候需要对方授权我们的公网IP以访问对方，那么如何知道我们的公网出口 IP 是什么呢？

```shell
#通过使用curl命令访问网站，由网站返回我们的出口ip地址。
curl ip.sb
curl ifconfig.me
#下一条返回信息更全
curl cip.cc
```

### 1.17：Linux命令行快捷键移动光标

说明: Ctrl - k ：先按住 Ctrl 键，然后再按 k 键

​          Alt -  k：先按住 Alt 键，然后再按 k 键

​          M - K ：先单击Esc，然后再按 k 键

```shell
#以下内容因shell不同略有区别，此处默认使用bash shell
Ctrl – a ：移到行首
Ctrl – e ：移到行尾
Ctrl – b ：往回(左)移动一个字符
Ctrl – f ：往后(右)移动一个字符
Alt – b ：往回(左)移动一个单词
Alt – f ：往后(右)移动一个单词
M-b ：往回(左)移动一个单词
M-f ：往后(右)移动一个单词
Ctrl – xx ：在命令行尾和光标之间移动
```

### 1.18：Linux命令行快捷键编辑

说明: Ctrl - k ：先按住 Ctrl 键，然后再按 k 键

​          Alt -  k：先按住 Alt 键，然后再按 k 键

​          M - K ：先单击Esc，然后再按 k 键

```shell
#以下内容因shell不同略有区别，此处默认使用bash shell
Ctrl – h ：删除光标左方位置的字符
Ctrl – d ：删除光标右方位置的字符（注意：当前命令行没有任何字符时，会注销系统或结束终端）
Ctrl – w ：由光标位置开始，往行首删。
Alt – d ：由光标位置开始，往行尾删。
M – d ：  由光标位置开始，往行尾删。
Ctrl – k ：删除当前行光标之后的全部内容。
Ctrl – u ：删除当前行光标之前的全部内容。
Ctrl – y ：粘贴之前删除的内容到光标后，可执行多次。可作为撤销最近一次删除的操作。
ctrl – t ：交换光标处字符和之前字符的位置。
Alt + . ：使用上一条命令的最后一个参数。如果不行使用Esc + .
Ctrl – _ ：回复之前的状态。撤销操作。可执行多次。
```

### 1.19：屏幕控制

```shell
#以下内容因shell不同略有区别，此处默认使用bash shell
Ctrl – l ：清除屏幕，然后，在最上面重新显示目前光标所在的这一行的内容。
Ctrl – o ：执行当前命令，并选择上一条命令。
Ctrl – c ：终止命令
Ctrl – z ：挂起命令
#下面命令不一定有效果，shell命令供测试效果：for ((i=1;i<=20;i++));do echo ${i};sleep 0.5;done
Ctrl – s ：阻止屏幕输出
Ctrl – q ：允许屏幕输出
```

### 1.20：检测远程TCP端口是否开放

```shell
#如果没有安装网络工具，无法使用telnet命令，可使用下面方式，以谷歌域名服务为例,ip 8.8.8.8 ，端口 53
echo > /dev/tcp/8.8.8.8/53 && echo "open"

#telnet命令，仅支持tcp协议
telnet remote-ip port
```

### 1.21：用curl & wget命令执行 ftp下载

```shell
# CURL 
# 列出目录列表
curl ftp://www.xxx.com/ --user name:passwd
curl ftp://www.xxx.com/ –u name:passwd #简洁写法
curl ftp://name:passwd@www.xxx.com #简洁写法2
# 下载一个文件
curl ftp://user:passwd@www.magedu.com/test.c -o test.c

# WGET
# -m -- mirror 此选项打开递归和时间戳，设置无限递归深度和保留 FTP 目录列表。目前相当于 -r -N -l inf --no-remove-listing。
wget -m ftp://username:password@hostname
```

### 1.22：命令重命名

```shell
# 可在各用户的.bash_profile中添加重命名配置，如下
alias ll='ls -alF'
# 定义了一个ll 命令， 对应命令 ls -a 显示所有文件包含隐藏文件，-l 长列表格式，-F 添加指示符（*/=>@|之一）附加到条目，比如目录后会有/
# 通过重命名命令，可以将一个很长的命令缩短，减少输入，创建需要的快捷命令
```

### 1.23：column 输出清晰的表格

这里直接给命令，通过实践来理解这个column的作用。

```shell
# -t 默认的分隔符为空格
mount | column -t
# 也可以通过 -s 选项指定分隔符
cat /etc/passwd | column -t -s :
```

### 1.24：重复执行一个命令，直到它成功运行

### 1.25：按内存资源的使用量对进程进行排序

```bash
ps aux | sort  -rnk 4
```

### 1.26：同时查看多个日志文件

有时候想着能同时查看多个日志文件，这是 tail 就比较麻烦。这时就可以使用 multi-tail 命令，既支持文本的高亮显示，还支持内容过滤等更多功能：

如果系统里还没有这个命令，运行apt-get install multitail 命令就可以把它给装上了

```shell
#同时查看/var/log/syslog /var/log/auth.log日志
#查看命令如下，更多命令使用 ctrl+h 查看
multitail /var/log/syslog /var/log/auth.log
```

<img src="https://raw.githubusercontent.com/9206284/Linux/main/img/multitail.png" style="zoom:50%;" />

### 1.27：cp 命令

-a ：相当于 -pdr；

-d ： 若来源文件为连接文件的属性(link file)，则复制连接文件属性而非文档本身；

-f  ： 强制 force的意思；

-i  :    若目标文件(destination)已经存在时，在覆盖时会先询问；

-l  ： 进行硬连接(hard link)的连接建立，而非拷贝档案本身；

-p ： 连同档案的属性一起复制过去，而非使用预设属性；

-r  :   递归持续复制

-s ：  复制称为符号连接文件(synbolic link)，即快捷方式档案；

-u： 若目标文件比源文件旧才更新；

最后一点，如果来源有两个及以上，最后一个目标一定要是目录才行。

小技巧： 我们使用的默认cp其实是做了 alias，完整命令是 `cp -i` ，所以如果想直接使用非交互的cp，可使用转义字符\， 即使用时前面加上反斜杠，如 `\cp`。

### 1.28：wget 断点续传和限速下载

```shell
#使用选项 -c 实现断电续传、--limit-rate 实现限速下载，如果在tmux里执行那更好啦。不会因为会话中断而停止下载。
wget -c --limit-rate=1024k https://releases.ubuntu.com/18.04/ubuntu-18.04.6-live-server-amd64.iso
```

### 1.29：bash中的短横线(-)用法

在Linux中短横线(-)可以表示输出流，具体用法如下。

1. cat -
   
   ```shell
   ##输入的内容会立刻输出
   saw@testserv-229-150:~$ cat -
   a
   a
   bb
   bb
   cc
   cc
   ^C
   ```

2. 搭配管道 |
   
   ```shell
   saw@testserv-229-150:~$ echo 123 | cat -
   123
   ```

3. 利用ssh完成scp命令
   
   ```shell
   ##利用ssh+cat - 传输
   cat minicom.log | ssh -p10086 saw@testsrv1 'cat - > /tmp/minicom.log'
   ## 确认文件已传输成功
   saw@testserv-229-150:~$ cat /tmp/minicom.log
   20220331 11:43:17 Hangup (0:00:00)
   20220331 11:43:50 Hangup (0:00:00)
   20220331 11:46:08 Hangup (0:00:00)
   20220331 11:48:07 Hangup (0:00:00)
   20220331 18:17:20 Hangup (0:00:00)
   
   ##含义是 cat minicom.log 产生输出流，在管道后的 - 则接收输出流，并重定向到/tmp/minicom.log
   ```

4. 并发打包
   
   ```shell
   tar -cvf - /data/test | pigz -6 -p 10 -k > test.tar.gz
   
   ##正常使用tar命令，是 tar -cvf /data/test.tar.gz /data/test
   ##上面命令，对/data/test 目录进行归档，输出流通过 - 来表示，然后管道传递给pigz命令。
   ##pigz优秀的压缩解压缩工具，-1或--fast最快压缩方式，-9或--best表示最高压缩比速度也最慢，默认是-6； -p指定并发线程，这里指定了10个，-k 选项表示压缩后不删除原文档。
   ```

### 1.30：awk转换123abc456为456ABC123

这里使用了awk的split、gsub以及print

```shell
# cat test.log
123abc456

# awk '{split($0,a,"[a-z]{3}");gsub("[0-9]{3}","",$1);print toupper(a[2]$1a[1])}' test.log
456ABC123

# awk '{match($0,"([0-9]+)([a-z]+)([0-9]+)",res);print res[3]toupper(res[2])res[1] }' test.log
456ABC123
```

### 1.31：xfsdump 备份还原数据

使用 nfs + xfsdump + xfsrestore 工具对磁盘进行备份和还原。

nfs服务器：192.168.229.72

备份主机：192.168.229.71

```shell
### 1. 服务端
#服务端安装并配置自启动
yum install nfs-utils -y && systemctl enable --now rpcbind.service && systemctl enable --now nfs.service && systemctl status rpcbind.service && systemctl status nfs.service
#服务端修改配置，添加客户端地址和共享目录
vim /etc/exports
/data/    192.168.229.0/24(rw,sync,no_root_squash,no_all_squash)
#使用exportfs命令将新配置写入内核，同时写入文件/var/lib/nfs/etab，使新配置生效。-r 表示重新导出所有目录，-a 表示导出或卸载所有目录，-u 表示卸载一个或多个目录，-v输出详细信息。
exportfs -rav
exporting 192.168.229.0/24:/data
#安装xfsdump和xfsrestore命令工具
yum -y install xfsdump



### 2. 客户端
#客户端安装并配置自启动
yum install nfs-utils -y && systemctl enable --now rpcbind.service && systemctl status rpcbind.service 
#挂载服务端共享目录
mount -t nfs 192.168.229.72:/data /data

xfsdump 
-f：指定备份文件目录
-L：指定标签session label
-M：指定设备标签media label
-s：备份单个文件，-s后面不能直接跟路径。

#使用xfsdump备份磁盘数据，命令中/data目录后的dump_vda3是数据的文件名。接着会出现两次对话，第一次输入备份的会话标签，这里是dump_vda3，第二次输入备份的设备描述，这里输入vda3。
xfsdump -f /data/dump_vda3 /dev/vda3
......
 ============================= dump label dialog ==============================

please enter label for this dump session (timeout in 300 sec)
 -> dump_vda3
session label entered: "dump_vda3"

 --------------------------------- end dialog ---------------------------------

 ============================= media label dialog =============================

please enter label for media in drive 0 (timeout in 300 sec)
 -> vda3
media label entered: "vda3"

 --------------------------------- end dialog ---------------------------------
......
xfsdump: Dump Summary:
xfsdump:   stream 0 /data/dump_vda3 OK (success)
xfsdump: Dump Status: SUCCESS


#模拟数据丢失
rm -rf /etc/nginx
#使用xfsrestore还原磁盘分区数据
xfsrestore -f /data/dump_vda3  /
......
xfsrestore: Restore Summary:
xfsrestore:   stream 0 /data/dump_vda3 OK (success)
xfsrestore: Restore Status: SUCCESS
#验证nginx服务
systemctl start nginx

### 实验验证，docker容器同样可以，只是还原后需要再启动下容器。
```

### 1.32：调试查看内存数据

通过pmap找到内存泄漏的地址和范围；

通过gcore对进程内存进行快照；

通过gdb加载内存信息；

通过dump binary 导出泄漏内存的内容；

通过vim查看内存内容；

https://panzhongxian.cn/cn/2020/12/memory-leak-problem-1/

```shell
#这里是一个redis服务，通过pmap加pid的方式查看内存映射，-x 表示要显示扩展信息
pmap -x 83514 
......
00007fc7bac3a000 1313280  450336  450336 rw---   [ anon ]   
#产生core dump文件。要产生 core dump 文件，可以通过man core 指令查看对 core dump 文件的介绍，它包含有进程在某时刻的进程内存情况。
gcore 83514
#确认内存地址起止范围，后面会用。
cat /proc/83514/maps  | grep 7fc7bac3a000
7fc7bac3a000-7fc80aeba000 rw-p 00000000 00:00 0

#使用 GDB关联bin文件和core dump文件
gdb redis-server core.83514
#dump 指定内存块内容，因为是16进制内容，所以前面加上0x
(gdb) dump binary memory result.bin 0x7fc7bac3a000 0x7fc80aeba000

#vim 查看，内容中多数是乱码，看可以识别的内容来定位
vim result.bin
```

## 二：Vim

### 2.1：打开文件的同时定位到指定行

```shell
vim +行号 filename 
例如，使用了grep -n查找到匹配字符串在文件中的行数，然后使用vim在打开文件的同时定位到指定行
grep -n ^SELINUX /etc/selinux/config
7:SELINUX=enforcing
12:SELINUXTYPE=targeted

vim +7 /etc/selinux/config 
```

### 2.2：配置vim

通过修改 Vim 的 `.vimrc` 配置文件来实现代码缩进、语法高亮、显示行号等功能。在个人的 Home 目录下创建一个 .vimrc 文件，添加下面的代码来设置行号、代码缩进等。

```ini
# 显示行号
set number
# 自动缩进
set autoindent
# 不换行
set nowrap
```

### 2.3：把命令的结果读入VIM

有时候你需要把某个命令的结果复制到 Vim 中，这在 Vim 也非常简单。切换到正常模式，然后输入 `:read !command` 即可把 `command` 的结果输入到 vim 中。

```shell
:read !ls -l
```

### 2.4：VIM快速添加注释及智能换行

打开文件后，不用进入编辑模式，直接按F4就会自动添加注释，帅气！

```shell
#打开vim配置文件，添加相关函数
# vi ~/.vimrc 
set autoindent
set tabstop=4
set shiftwidth=4
function AddTitle()
call setline(1,"#!/bin/bash")
call append(1,"#====================================================")
call append(2,"# Author: Magedu-tony")
call append(3,"# Create Date: " . strftime("%Y-%m-%d"))
call append(4,"# Description: ")
call append(5,"#====================================================")
endf
map <F4> :call AddTitle()<cr>
```

### 2.5：VIM连续行注释和取消注释

连续行添加注释命令：命令模式下<font color=green>`:n1,n2s/^/#/g`</font>    解释："^"表示行首，"#"是替换的字符，n1,n2s 在n1和n2行之间搜索;

连续行取消注释命令：命令模式下<font color=green>`:n1,n2s/^#//g`</font>    解释：n1,n2s 在n1 和n2 行之间搜索， 行首后为#的 "^#"， 替换为空，同删除。

### 2.6：VIM自动替换

命令如下，在vim命令模式下输入<font color=green> ab `将替换的内容`   `替换后的内容`</font>，然后回车。此时在vim插入模式中，输入`输入的内容`，都会被替换成`替换的内容`。

可以使用 **ab** 这个命令，把一些经常用的，比较长的东西，像学校名、公司名、邮箱地址等等。如果比较长的信息会大大简化操作，提高工作效率。

```shell
:ab 将替换的内容  替换后的内容
```

如果需要长期保存自己定义的设置，可以将相关命令保存在 ~/.vimrc文件中。

## 三：Linux基本操作

### 3.1：开关机

```shell
# 关机
shutdown -h now
# 重启
shutdown -r now
```

### 3.2：查看系统，CPU信息

```shell
# 查看系统内核信息
uname -a
# 查看系统内核版本
cat /proc/version
# 查看当前用户环境变量
env

# 查看CPU信息
cat /proc/cpuinfo
# 查看有几个逻辑cpu, 包括cpu型号
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
# 查看有几颗cpu,每颗分别是几核
cat /proc/cpuinfo | grep 'physical id' | sort | uniq -c
# 查看当前CPU运行在32bit还是64bit模式下, 如果是运行在32bit下也不代表CPU不支持64bit
getconf LONG_BIT
# 结果大于0, 说明支持64bit计算. lm指long mode, 支持lm则是64bit
cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l
```

### 3.3：建立软链接

```shell
ln -sn /usr/local/jdk1.8/ jdk
```

### 3.4：rpm相关

```shell
# 查看是否通过rpm安装了该软件
rpm -qa | grep 软件名
```

### 3.5：同步时间

```shell
#因使用的同步软件不同命令略有区别
#ntpd
ntpdate -u ntp.api.bz

#chronyd
chronyc makestep
```

### 3.6：后台运行命令

```shell
# 后台运行,并且有nohup.out输出
nohup xxx &

# 后台运行, 不输出任何日志
nohup xxx > /dev/null &

# 后台运行, 并将错误信息做标准输出到日志中 
nohup xxx >out.log 2>&1 &
```

### 3.7：强制活动用户退出

```shell
# 命令来完成强制活动用户退出.其中TTY表示终端名称
pkill -kill -t [TTY]
```

### 3.8：查看命令路径

```shell
which <命令名称>
```

### 3.9：配置dns

```shell
vim /etc/resolv.conf
```

### 3.10：查看域名路由表

```shell
nslookup google.com
```

### 3.11：last 最近登录信息列表

```shell
last -n 5
```

### 3.12：配置固定ip

```shell
ifconfig eth0 192.168.223.100 netmask 255.255.255.0
```

### 3.13：查看进程内加载的环境变量

```shell
# 也可以去 cd /proc 目录下, 查看进程内存中加载的东西
ps eww -p  XXXXX(进程号)
```

### 3.14：查看进程树找到服务器进程

```shell
ps auwxf
```

### 3.15：查看进程启动路径

```shell
cd /proc/xxx(进程号)
ls -all
#cwd对应的是启动路径
```

### 3.16：scp命令

```shell
# “-p”参数会帮到把预计的时间和连接速度会显示在屏幕上。
scp -p Label.pdf mrarianto@202.x.x.x:.

# 用-C参数来让文件传输更快
# “-C”参数，它的作用是不停压缩所传输的文件。它特别之处在于压缩是在网络传输中进行，当文件传到目标服务器时，它会变回压缩之前的原始大小。
scp -Cpv messages.log mrarianto@202.x.x.x:.

# 选择其它加密算法来加密文件
# SCP默认是用“AES-128”加密算法来加密传输的。如果你想要改用其它加密算法来加密传输，你可以用“-c”参数。
scp -c 3des Label.pdf mrarianto@202.x.x.x:.

# 限制带宽使用,“-l”参数，它能限制使用带宽。SCP是用千字节/秒 (KB/s)计算的，所以如果你想要限制SCP的最大带宽只有50 KB/s，你就需要设置成50 x 8 = 400。
scp -l 400 Label.pdf mrarianto@202.x.x.x:.
```

### 3.17：nethogs命令查看进程流量

`nethogs` 的界面中会显示进程名称、进程 ID (PID)、用户、上传流量、下载流量等信息，按照流量大小进行排序，用户可以根据需要查看特定进程的网络流量情况。`nethogs`是一个强大的网络流量监控工具，可以帮助用户实时了解进程的网络流量使用情况，方便进行网络流量管理和排查网络问题。

```shell
# 通过 -d 参数指定要监控的网络接口，例如只监控 eth0 接口
nethogs -d eth0

# 通过 -s 参数设置刷新频率，单位为秒，默认为 1 秒
sudo nethogs -s 5  # 设置刷新频率为 5 秒

# 通过 -v 参数设置显示的流量单位，例如以 KB/s 显示
nethogs -v KB/s
```

## 四：正经小技巧

### 4.1：linux下网速测试

Speedtest强大而知名的全球宽带网络速度测试网站，采用Flash载入界面，Alexa世界排名非常高，Speedtest.net在全球有数百个测试节点，国内有测速节点几十个。

官网地址：http://www.speedtest.net/

```shell
wget https://github.com/sivel/speedtest-cli/archive/master.zip
unzip master.zip
cd speedtest-cli-master
./speedtest.py --share

Retrieving speedtest.net configuration...

Testing from China Telecom (58.49.16.6)...

Retrieving speedtest.net server list...

Selecting best server based on ping...

Hosted by wuhan hello 5G (yangzhou) [221.65 km]: 9.74 ms

Testing download speed................................................................................

Download: 353.29 Mbit/s

Testing upload speed................................................................................................

Upload: 770.79 Mbit/s

Share results: https://www.speedtest.net/result/14510888539.png

# 查看https://www.speedtest.net/result/14510888539.png ，有图文显示
```

### 4.2：zabbix监控iptables规则变动

以下内容添加zabbix-agent

```shell
#整体规则做md5校验
UserParameter=iptables.md5,sudo /sbin/iptables-save | egrep -v '^#.' | egrep -v "^:.+\[.+\]$"|md5sum|cut -d " " -f 1

#页面传参
UserParameter=iptables.md5.table.[*],sudo /sbin/iptables -S -t $1 -w 3|md5sum|cut -d " " -f 1
```

### 4.3：! 符号在删除时排除某些文件或目录

 rm -rf !(keep) #删除keep文件之外的所有文件

 rm -rf !(keep1 | keep2) #删除keep1和keep2文件之外的所有文件

要在 Bash 中使用 `!(keep1 | keep2)` 这样的模式匹配，可以通过设置 `extglob` 选项来启用扩展的模式匹配功能。`shopt -s extglob` 命令会打开 Bash 的 `extglob` 选项，使得 `!(pattern)` 这样的模式匹配能够正常工作。

```shell
# 1.开启bash的extglob选项
shopt -s extglob
# 2.删除除keep1、keep2之外的文件目录
rm -rf !(keep1|keep2)
```

### 4.4：raid0更换硬盘&非启动硬盘损坏如何正常重启

1. 首先修改/etc/fstab文件，所有非启动分区的硬盘配置上<font color=red>nofail</font>选项，关机。
   
   ```shell
   UUID=68026528-6ce0-4283-b884-fe63479b5f30 /                       xfs     defaults        0 0
   UUID=cf278087-f06c-40c2-bbc9-5f66b1ad8c29 /boot                   xfs     defaults        0 0
   UUID=25daa4c0-01f4-424f-adf0-ef6cf00f4bae /home                   xfs     defaults,nofail         0 0
   ```

2. 更换故障硬盘，升级bios。

3. 开机、找到更换的硬盘做格式化
   
   ```shell
   parted -l
   # 可以看到某块磁盘状态为error，没有相关parted信息。
   parted /dev/sdx #处理对应的磁盘
   mkfs.xfs  /dev/sdx1 #这里只对/dev/sdx磁盘分了一个区。
   ```

4. 重新加载系统服务
   
   ```shell
   systemctl daemon-reload
   # 下面根因分析解释为什么需要做此动作
   ```

5. 挂载磁盘
   
   ```shell
   # 1. 临时挂载
   mount /dev/sdx1 /home
   # 2. 永久挂载
   lsblk /dev/sdx1 #获取uuid，然后编写/etc/fstab文件，替换原内容。
   # 3. 验证是否正常
   mount -a        
   ```
   
   现象：关于更换、新增硬盘，修改/etc/fstab将新硬盘安装到旧挂载点，然后umount旧磁盘，执行**mount -a**后使用**df**查看没有挂载成功。
   
   <font color=red>根因分析</font>：
   
   ```shell
   # 1.查看挂载失败的挂载点
   systemctl list-units --type=mount| grep failed 
   test1.mount             loaded failed failed  /test1
   
   # 2.查询该unit的状态
   systemctl status test1.mount  
   Loaded: loaded (/etc/fstab; bad; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2019-08-28 15:32:53 CST; 3min 27s ago
   Where: /test1
   What: /dev/vdb1
   Docs: man:fstab(5)
   man:systemd-fstab-generator(8)
   Process: 4601 ExecUnmount=/bin/umount /test1 (code=exited, status=0/SUCCESS)
   Process: 3129 ExecMount=/bin/mount /dev/vdb1 /test1 -t ext4 (code=exited, status=0/SUCCESS)
   ... ...
   Warning: test1.mount changed on disk. Run 'systemctl daemon-reload' to reload units.
   # 通过回显可以看到，因为磁盘发生改变，所以需要执行'systemctl daemon-reload'。在运行该命令之前，systemd不读取fstab并生成装载单元。
   ```

### 4.5：以普通用户身份免密切换另一个普通用户

```shell
# root用户身份配置普通用户身份切换另一个普通用户的免密
# 这里我们有两个普通用户mei和derrick，现在以mei免密切换derrick
# 1.编辑新增一个文件mei-sudo-derrick，内容如下
visudo -f /etc/sudoers.d/mei-sudo-derrick   
mei ALL=(derrick) NOPASSWD: ALL

# 2.免密切换验证，这里的bash可以替换为其他command，即以derrick的身份执行命令，包括打开新的shell。
mei > sudo -u derrick bash
#或者
mei > sudo -u derrick -s
#  -s, --shell                   以目标用户运行shell；可同时指定一条命令
```

### 4.6：开启性能模式

```shell
# 查看当前的模式
tuned-adm active 

# 查看支持的模式
tuned-adm list

# 优化磁盘吞吐量，针对数据库常用
tuned-adm profile throughput-performance
# 磁盘低延迟
latency-performance 
# 网络吞吐量
network-throughput
# 网络低延迟
network-latency
# 优化虚拟机宿主
virtual-host
# 优化虚拟客户机
virtual-guest

# 禁用所有优化模式
tuned-adm off
```

性能调优详细介绍
https://talk.linuxtoy.org/tuned-adm-slides/

### 4.7：查看当前主机是物理机还是虚拟机？

```shell
dmidecode -s system-product-name
#
dmesg | grep -i virtual
```

### 4.8：查看进程启动时间、运行时长

```shell
# 首先获取进程的PID
# 查看指定pid为16323 的进程启动日期时间，运行时长，cpu占用时长，uid，gid和CMD
ps -p 16323 -o lstart,etime,time,uid,gid,cmd
```

### 4.9：关于raid写惩罚和IOPS计算

这里从原理上浅谈不同raid级别的写惩罚值，以及通过写惩罚计算可用IOPS的方法。

- 写惩罚值
  
  raid-0：直接的条带，数据每次写入对应物理盘上的一次写入，写惩罚值是 <font color=green>**1**</font>；
  
  raid-1和raid10：因为镜像的存在，所以会有两次写入，写惩罚值是 <font color=green>**2**</font>；
  
  raid-5：因为校验位的存在，需要读数据、读校验位、写数据、写校验位四个步骤，所以写惩罚值是<font color=green> **4**</font>；
  
  raid-6：由于两个校验位的存在，需要读两次校验位和写入两次校验位，所以写惩罚值是 <font color=green> **6**</font>；

- 计算IOPS
  
  在实际存储方案设计的过程中，计算实际可用IOPS的过程中必须纳入RAID的写惩罚计算。计算的公式如下：
  
  物理磁盘总的IOPS = 物理磁盘的IOPS × 磁盘数目
  
  可用的IOPS = （物理磁盘总的IOPS × 写百分比 ÷ RAID写惩罚） + （物理磁盘总的IOPS × 读百分比）
  
  假设组成RAID-5的物理磁盘总共可以提供500 IOPS，使用该存储的应用程序读写比例是50%/50%，那么对于前端主机而言，实际可用的IOPS是：
  
  （500 ×50% ÷ 4）+ ( 500 * 50%) = 312.5 IOPS

### 4.10：shell脚本

#### 4.10.1 使用nc命令检测nginx的代理server是否可达

需求背景：在做nginx的高可用升级时，我们通过keepalived完成VIP地址的漂移，但在此之前需要验证线上代理配置的后端服务是否都可以达，另外也存在配置中已经失效的后端server服务，这个代理配置本身没有启用，所以也不影响生产，这种怎么检验呢？因此加入了不通的服务ip和端口。通过排序，我们可以方便的对比两边节点内容是否一致，不通的部分也一致。那么这两边的nginx配置文件一定是一致的，且后端服务可达。

前提需要支持nc命令，`yum install nmap` 命令安装nmap。

```shell
#!/bin/bash
#脚本使用nc命令检查nginx配置中代理server是否可达，并记录到/tmp/proxy-check.txt文件中
#相关server变量从nginx配置文件/etc/nginx/conf.d/中检索获取

# 1.清空清单文件
> /tmp/proxy-check.txt

# 2. 获取目标目录/etc/nginx/conf.d下所有配置文件的server，并排除server配置块，处理配置行末的分号
lists=`grep -rinE '^\s*server [^#]+' /etc/nginx/conf.d/ | grep -v '{'  | awk '{print $3}' | sed 's/;$//'`
# 对获取到的server列表进行循环
for i in $lists 
do
# 以:为分隔符，前部分是ip，后部分是端口
  IFS=':' read -ra parts  <<< ${i}
  ip=${parts[0]}
  port=${parts[1]}
  nc -z -w 5 "${ip}" "${port}" >> /dev/null 2>&1
  if [ $? -eq 0 ]; then
    #echo "${ip}:${port} 连接成功" | tee -a /tmp/proxy-check.txt
        echo "${ip}:${port} 连接成功"
  else 
# 将失败的ip+端口存放到清单文件中
    echo "${ip}:${port} 连接失败 --" | tee -a /tmp/proxy-check.txt
  fi
done

echo '================================================'
# 获取无法访问的清单文件内容排序后展示
cat /tmp/proxy-check.txt | sort
```

#### 4.10.2 通过nslookup命令解析nginx的server_name对应IP

nginx-servername-resolv.sh：此脚本对server_name下配置多个域名的会自动分割循环解析。

```shell
#!/bin/bash

# 定义Nginx配置文件目录路径
nginx_config_dir="/etc/nginx/conf.d"  # 请根据你的实际路径进行修改

# 定义输出文件路径
output_file="nginx_server_ips.txt"

# 定义超时时间（秒）
timeout_seconds=5

# 用于存储域名和对应IP的文件
echo "" > "$output_file"

# 查找Nginx配置文件目录下的所有配置文件
config_files=$(find "$nginx_config_dir" -type f -name "*.conf")

# 使用awk命令从配置文件中提取域名
for config_file in $config_files; do
    awk '/server_name/ {
        for (i=2; i<=NF; i++) {
            gsub(";", "", $i)
            split($i, domains, " ")
            for (j in domains) {
                print domains[j]
            }
        }
    }' "$config_file" | while read -r domain; do
        # 使用timeout命令设置超时时间来执行nslookup命令
        ip_info=$(timeout "$timeout_seconds" nslookup "$domain")

        if [ $? -eq 0 ]; then
            # 域名解析成功
            echo "File: $config_file" >> "$output_file"
            echo "Domain: $domain" >> "$output_file"
            echo "$ip_info" | awk '/^Server:/ { print "  DNS Server:", $2 }' >> "$output_file"
            echo "$ip_info" | awk '/^Address: / { print "  IP Address:", $2 }' >> "$output_file"
        else
            # 域名解析超时或失败
            echo "File: $config_file" >> "$output_file"
            echo "Domain: $domain" >> "$output_file"
            echo "  DNS Resolution: Timeout or Error" >> "$output_file"
        fi

        echo "" >> "$output_file"
    done
done

echo "域名和IP信息已写入$output_file"
```

#### 4.10.3 备份ngx配置文件含失败通知

ngx-conf-backup.sh

```shell
#!/bin/bash

# 定义Nginx配置文件路径和备份目录
nginx_config="/etc/nginx/conf.d"
backup_dir="/data/backups"

# 定义通知人员邮箱
contact='wushun@raipeng.com,shaohui@raipeng.com,jinwenming@raipeng.com,cuihenglai@raipeng.com,zwzhou@dawntech-info.com'

# 生成备份文件名
backup_file="$backup_dir/nginx_config_$(date +\%Y\%m\%d_\%H\%M\%S).bak"

# 备份Nginx配置文件
if cp -r "$nginx_config" "$backup_file"; then
  echo "Nginx配置文件备份成功: $backup_file" >> /tmp/ngx-config-backup.log
else
  echo "Nginx配置文件备份失败" >> /tmp/ngx-config-backup.log
  # 发送通知
  curl -X POST --header 'Content-Type: application/json' --header 'Accept: text/json' "http://192.168.213.52:10002/email-api/email/v1/sendEmail?scene=warn&mark=common&content=ngx-conf-backup-fault&title=ngx-conf-backup-fault&receivers=${contact}"
fi
```

### 4.11：打印tomcat工作内存堆栈信息

系统跑的java tomcat，要触发tomcat thread dump非常简单，首先找到tomcat对应的pid

`kill -3 对应pid `  这个命令会发送一个信号，将触发线程的dump，内容记录在catalina.out中。

kill -3 pid 后文件的保存路径为：/proc/${pid}/cwd， 文件名为：antBuilderOutput.log ，看看有没有dump格式文件

### 4.12：tcp_timestamps和tcp_tw_recycle在NAT情况下引发的弃包

情况是在web主机开启了以上两个参数之后，前方nat主机又主动断开客户端连接，端口再次使用因为连接socket文件已删除，只能根据包时间戳来判断是否进行下一步处理，一旦包时间戳早于当前时间戳就会进入60s的丢弃包行为。所以web服务前面在有nat的情况下，不能同时开启以上两个参数，直接关闭tcp_tw_recycle。

https://www.cnblogs.com/charlieroro/p/11593410.html

### 4.13：主机多个ip地址，指定源ip

Linux 2.2 选择源 IP 地址使用以下三种机制：

1. 应用程序可以通过<font color=blue>bind(2) 系统</font>调用，应用至 sendmsg(2) 调用上，并通过辅助数据对象 IP_PKTINFO ，从而显式指定源 IP 地址。在这种情况下，操作系统内核仅仅检查其源 IP 地址是否正确，否则产生相应的错误。
2. 如果应用程序没有指定源IP 地址，包含源 IP 的路由表将决定数据包源 IP 地址，1) 通过设置 ip route 命令的 src参数，从而指定源 IP 地址；2) 如果路由表没有包含 src 属性，则使用主要 IP 地址。
3. 其它情况下内核搜寻绑定定数据包路由接口上的IP 地址， IPv6 选择第一个可用的 IP 地址。 IPv4 情况下，尽量选择与目标 IP 处于同一子网的源 IP ，如果目标 IP 与自己的所有 ip 没有处于同一子网，则使用第二种算法。

源ip选择的处理方法有两种，路由方式或iptables修改数据包源地址，推荐路由方式。

```shell
# 如果绑定的几个IP 处于同一个子网内，那么主要 IP 地址将被使用（如 eth0 接口上的 IP ） , 
# 方案一. 使用[iptables]修改数据包的源地址，如：
# (https://so.csdn.net/so/search?q=iptables&spm=1001.2101.3001.7020) 
iptables -t nat -I POSTROUTING -o eth0 -d 1.2.3.4/0 -s 192.168.229.237 -j SNAT --to-source 192.168.229.7


# 方案二修改路由
# 2.1查看ip地址
# 请注意 192.168.229.237 是 global 状态，而其它地址是 global secondary 状态。
ip addr  
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:66:da:10 brd ff:ff:ff:ff:ff:ff
    inet 192.168.229.237/24 brd 192.168.229.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.229.7/24 brd 192.168.229.255 scope global secondary eth0:0
       valid_lft forever preferred_lft forever
    inet 192.168.229.8/24 brd 192.168.229.255 scope global secondary eth0:1
       valid_lft forever preferred_lft forever
    inet 192.168.229.9/24 brd 192.168.229.255 scope global secondary eth0:2
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe66:da10/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:8e:97:e0:b5 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
# 2.2查看路由表
ip route
default via 192.168.229.1 dev eth0 proto static
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
192.168.1.0/24 via 192.168.229.5 dev eth0
192.168.200.0/24 via 192.168.229.5 dev eth0
192.168.229.0/24 dev eth0 proto kernel scope link src 192.168.229.237

# 3. 修改路由表，指定ip 192.168.229.7 作为源地址 ✅
ip route change default dev eth0 src 192.168.229.7
ip route change to 192.168.200.0/24 dev eth0 src 192.168.229.7
```

### 4.14：屏幕滚动显示pcie bus error: severity=corrected, type=physical layer, id

这可能是由于 PCIe 活动状态电源管理将链路转换为较低的功耗状态，并可能导致设备触发这些错误。

```shell
# 1. 加入系统启动参数
vim /etc/default/grub
GRUB_CMDLINE_LINUX="...... pcie_aspm=off"

# 2. 更新启动文件
# centos
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
# ubuntu
update-grub
reboot

# 3. 确认pcie_aspm的配置
dmesg|grep pcie
```

### 4.15：导入GPG公私钥

```shell
#导入公钥
gpg --list-keys
gpg --import pub.asc
#查看是否导入
gpg --list-keys
#信任此公钥，bris-test是uid
gpg --edit-key bris-test 
gpg> trust
Your decision? 5
Do you .... ? (y/N) y

#导入私钥
gpg --list-secret-keys
gpg --import jsyd.gpg
#查看私钥是否导入成功
gpg --list-secret-keys
#测试私钥解密
gpg --batch --passphrase Leit52140 --recipient xxxxx -output 输出的文件名 --decrypt 待解密目标文件名
```

### 4.16：删除特殊字符的文件或目录等

```shell
# 1.获取文件、目录、软链等的inum
ls -i

# 2.删除文件或目录，目录不为空也不能删除。符号链接也不能使用这个选项。
find  ./ -inum xxxx -delete
```

### 4.17：日志自动分隔及压缩logrotate

默认情况下，logrotate 命令作为放在 `/etc/cron.daily` 中的 cron 任务，每天运行一次，它会帮助你设置一个策略，其中超过某个时间或大小的日志文件被轮换。

命令： `/usr/sbin/logrotate`     ， 配置文件： `/etc/logrotate.conf`  ，这是 logrotate 的主配置文件。logrotate 还在`/etc/logrotate.d/` 中存储了特定服务的配置。确保此内容  "` include  /etc/logrotate.d`"  包含在 `/etc/logrotate.conf` 中，以读取特定服务日志配置。

重要的 logrotate 选项：

```ini
compress             --> 压缩日志文件的所有非当前版本
daily,weekly,monthly --> 按指定计划轮换日志文件
delaycompress        --> 压缩所有版本，除了当前和下一个最近的
endscript            --> 标记 prerotate 或 postrotate 脚本的结束
errors "emailid"     --> 给指定邮箱发送错误通知
missingok            --> 如果日志文件丢失，不要显示错误
notifempty           --> 如果日志文件为空，则不轮换日志文件
olddir "dir"         --> 指定日志文件的旧版本放在 “dir” 中
postrotate           --> 引入一个在日志被轮换后执行的脚本
prerotate            --> 引入一个在日志被轮换前执行的脚本
rotate 'n'           --> 在轮换方案中包含日志的 n 个版本
sharedscripts        --> 对于整个日志组只运行一次脚本
size='logsize'       --> 在日志大小大于 logsize（例如 100K，4M）时轮换
create 700 root root --> 轮换原始文件，同时创建具有指定权限、用户和组的新文件。
```

追加logrotate 的配置文件 /etc/logrotate.d/tomcat ，内容如下

```ini
/home/soft/tomcat_*/logs/*.out
{
    create 0644 root root
    daily
    rotate 365
    compress
    dateext
    dateformat .%Y-%m-%d
    missingok
    copytruncate
}
```

立即执行 `/usr/sbin/logrotate  /etc/logrotate.conf`

使用命令查看 logrotate 历史:  cat /var/lib/logrotate.status

### 4.18：添加硬盘后不重启系统识别设备

linux系统在运行状态，添加了磁盘设备，不重启系统的条件下如何将磁盘设备添加到系统呢？看如下操作：

```shell
# 一条for循环处理：
for i in $(ls /sys/class/scsi_host/);do echo $i; echo "- - -" > /sys/class/scsi_host/${i}/scan ;done

# 分步处理：
# 1、查看host个数
ls /sys/class/scsi_host/
# 2、重新扫描
echo "- - -" > /sys/class/scsi_host/host编号/scan
```

### 4.19：配置systemd的service服务

在 Linux 系统中，`/usr/lib/systemd/system/` 和 `/etc/systemd/system/` 是两个存放 Systemd 服务单元文件的目录，它们之间有一些差异：

1. **`/usr/lib/systemd/system/` 目录：**
   - 包含系统安装的软件包提供的默认服务单元文件。
   - 这里的服务单元文件通常由软件包管理系统（如包管理器）安装，是软件包的一部分。
   - 修改这里的文件可能会被软件包管理器覆盖，因此不建议直接在这里编辑文件。
2. **`/etc/systemd/system/` 目录：**
   - 包含本地系统管理员（用户）自定义的服务单元文件或对系统默认服务单元文件的修改。
   - 这里的文件会覆盖 `/usr/lib/systemd/system/` 目录下相同名称的文件，因为 Systemd 会按照覆盖的优先级加载服务单元。
   - 修改这里的文件不会被软件包管理器覆盖，因为这是系统管理员的自定义配置空间。

总体来说，`/etc/systemd/system/` 目录更适合用于用户自定义的服务单元文件，而 `/usr/lib/systemd/system/` 目录则包含系统安装的默认服务单元文件。在实践中，为了避免与软件包管理器的冲突，最好将用户自定义的服务单元文件放置在 `/etc/systemd/system/` 目录下。

<font color=red>**正确删除service服务**</font>

```shell
#删除服务名称
systemctl stop your-service
systemctl disable your-service
rm /etc/systemd/system/your-service.service
systemctl daemon-reload
systemctl reset-failed
```

tomcat服务示例：这里将成功的退出码设置为143，因为实际操作时发现kill -15 退出码是143。因此修改默认的退出码0为143。

```ini
[Unit]
Description=tomcat_special_http_status_push_2
After=syslog.target network.target

[Service]
Type=forking
LimitNOFILE=102400
LimitNPROC=102400
ExecStart=/apps/tomcat_special_http_status_push_2/bin/startup.sh
ExecStop=/bin/kill -15 
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

jar服务示例

```ini
[Unit]
Description=special stats
After=network.target

[Service]
Type=forking
StandardOutput=null
StandardError=null
LimitNOFILE=102400
LimitNPROC=102400
ExecStart=/data/apps/special-stats/deploy/start.sh
ExecStop=/data/apps/special-stats/deploy/stop.sh
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

### 4.20：systemd的环境变量

首先systemd的环境变量与系统登陆用户的环境变量是没有联系的，使用命令 `systemctl show-environment` 查看systemd的环境变量。

1. 方法一：变量参数引入，`Environment=`
   
   编辑 systemd 的 service 文件，添加 `Environment=` 形如下：
   
   ```ini
   [Service]
   Environment="MY_VAR_1=VAR_1_VALUE"
   Environment="MY_VAR_2=VAR_2_VALUE"
   ```

2. 方法二：变量文件引入，`EnvironmentFile=`
   
   ```ini
   [Service]
   EnvironmentFile=/opt/workspace/env_test.env
   EnvironmentFile=-/opt/workspace/env_test_not_exist.env
   ```
   
   上述指定了两个设置环境变量的文件：`/opt/workspace/env_test.env` 和 `/opt/workspace/env_test_not_exist.env`。
   需要注意的是，第二个 EnvironmentFile 使用 `-` 在目录前，作用是忽略文件不存在。
   
   文件`/opt/workspace/env_test.env` 内容格式如下
   
   ```ini
   MY_VAR_1=VAR_1_VALUE
   MY_VAR_2=VAR_2_VALUE
   ```

3. 方法三：配置覆盖引入，`xxx.service.d/override.conf`
   
   在 xxx.service 同目录下，创建 xxx.service.d 文件夹，在该文件夹下，创建 override.conf （文件名随便，一般为 override.conf ）
   
   创建的文件格式如下：
   
   ```ini
   [Service]
   Environment="MY_VAR_1=VAR_1_VALUE"
   Environment="MY_VAR_2=VAR_2_VALUE"
   ```

### 4.21：linux内核模块查看、加载、删除及依赖性

内核模块一般放置在/lib/modules/$(uname -r)/kernel目录下。

- 内核模块的查看
  
  lsmod ： 查看当前内核加载了哪些模块；
  
  modinfo [-adin] [module_name] ： 查看指定模块的信息；
  
         - -a (author) 作者信息
         - -d (description) 模块的描述信息
         - -l (license) 模块的授权信息
         - -n 模块的路径信息

- 内核模块的加载与删除
  
  modprobe [-rfcv] module_name：
  
  - -r 删除指定的模块；
  - -f 强制加载或删除指定的模块；
  - -c 打印已知的配置信息
  - -v 显示更多信息
  - 示例
    - modprobe ipv6.ko
    - modprobe -r ipv6.ko
  
  insmod /full_path/module_name ： 加载指定模块需要输入模块的完整路径名，如果待加载的模块有依赖模块时，加载可能失败，作为替代，modprobe 会自动解决依赖问题。
  
  rmmod [-fw] module_name： 删除指定模块；
  
  - -f  强制删除模块，不论模块是否正被使用
  - -w 如果模块正被使用，则等待模块使用完毕后删除；

- 内核模块的依赖性：文件/lib/modules/$(uname -r)/modules.dep，记录了内核所支持的各个模块的依赖性
  
  depmod [-Ane]命令用来生成modules.dep文件。不加任何参数：depmod分析所有的内核模块，然后重新写入modules.dep文件。
  
  - -A: 快速查找比modules.dep新的模块，如果找到新模块，才会更新modules.dep；
  
  - -n: 将结果直接显示在屏幕上，而不写入modules.dep文件；
  
  - -e: 显示出目前已加载的，不可执行的模块名称。
