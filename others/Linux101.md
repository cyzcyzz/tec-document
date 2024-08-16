# LINUX 技巧 101

[TOC]

## 一：Powerful CD Command（强大的CD命令）

### 1.1 CDPATH 定义 CD命令的基础目录(great)

```bash
#centos7.9
[ramesh@dev-db ~]# pwd 
/home/ramesh 
[ramesh@dev-db ~]# cd mail
-bash: cd: mail: No such file or directory

[ramesh@dev-db ~]# export CDPATH=/var
[ramesh@dev-db ~]# cd mail 
/var/mail
```

如想永久生效，将 export CDPATH=/etc 加入到 ~/.bash_profile文件。

与PATH变量相似的，可以在CDPATH中加入多个目录，使用:分割它们，如下

```bash
export CDPATH=.:~:/etc:/var
```

这个技巧在类似情形很有用

- Oracle DBAs经常在$ORACLE_HOME下工作，可以将此目录加入CDPATH
- 系统管理员经常在 /etc 下工作，可以将/etc 加入CDPATH
- 开发人员经常在 /home/projects 目录下，可以将 /home/projects 加入CDPATH
- 最终用户经常访问自己的家目录，可以将家目录加入 CDPATH

### 1.2 通过CD alias 快速访问上级目录(great)

向上访问一个很深的目录时，常常使用像 cd ../../../ 这样的路径。介绍使用alias来实现快速的访问上级目录

```bash
#使用cat命令将配置写入 ~/.bash_profile
cat <<EOF >> ~/.bash_profile
#alias快速访问上级目录
alias ..="cd .."
alias ..2="cd ../.."
alias ..3="cd ../../.."
alias ..4="cd ../../../.."
alias ..5="cd ../../../../.."
EOF

#创建很深的目录层级
# mkdir -p /tmp/very/long/directory/structure/that/is/too/deep
# cd /tmp/very/long/directory/structure/that/is/too/deep

#如下命令向上4级目录
# ..4

# pwd
/tmp/very/long/directory/structure

#类似的命名方案还有，五个点可以表示上面第四级目录
alias ..="cd .."
alias ...="cd ../.."
alias ....="cd ../../.."
alias .....="cd ../../../.."  
alias ......="cd ../../../../.."  
#cd后面跟数字，数字表示向上的层级目录
alias cd1="cd .."
alias cd2="cd ../.."
alias cd3="cd ../../.."
alias cd4="cd ../../../.."
alias cd5="cd ../../../../.."
```

### 1.3 一条命令建立目录并进入该目录(good)

之前是这样创建目录并进入目录

```bash
# mkdir -p mkdircd  /tmp/subdir1/subdir2/subdir3
# cd /tmp/subdir1/subdir2/subdir3
# pwd
/tmp/subdir1/subdir2/subdir3
```

现在将mkdir 和 cd 组合成一个命令，放在 ~/.bash_profile文件中并重新登录。(.bash_profile是bash的配置文件)

```bash
# 加入内容
function mkdircd () { mkdir -p "$@" && eval cd "\"\$$#\""; }

# 可使用如下命令直接追加
cat <<EOF  >> ~/.bash_profile
function mkdircd () { mkdir -p "\$@" && eval cd "\"\\$\$#\""; }
EOF
```

重新登录后，最终效果如下

```bash
# mkdircd  /tmp/subdir1/subdir2/subdir3

# pwd
/tmp/subdir1/subdir2/subdir3
```

### 1.4 使用 ”cd -“在最后两个目录间切换

```shell
# cd /tmp/very/long/directory/structure/that/is/too/deep

# cd /tmp/subdir1/subdir2/subdir3

# cd -

# pwd 
/tmp/very/long/directory/structure/that/is/too/deep

# cd -

# pwd
/tmp/subdir1/subdir2/subdir3

# cd

# pwd
/tmp/very/long/directory/structure/that/is/too/deep
```

### 1.5 使用dirs，pushd和popd操作目录堆栈

使用目录堆栈，可以将目录推送到目录堆栈中，然后再将目录从堆栈中弹出。

- dirs ：显示目录堆栈
- pushd：将目录推送到目录堆栈中
- popd：将目录从目录堆栈中弹出并进入其中

<font color=red>dirs 将始终打印当前目录</font>，然后是堆栈的内容。即目录堆栈为空，dirs 命令将仅打印当前目录，如下所示。

```bash
# popd 
-bash: popd: directory stack empty 
# dirs 
~ 
# pwd 
/home/ramesh
```

先建立一些临时目录，然后将它们推送到目录堆栈中。

```bash
mkdir /tmp/dir1 
mkdir /tmp/dir2 
mkdir /tmp/dir3 
mkdir /tmp/dir4 

cd /tmp/dir1 
pushd . 

cd /tmp/dir2 
pushd . 

cd /tmp/dir3 
pushd . 

cd /tmp/dir4 
pushd .

dirs
/tmp/dir4 /tmp/dir4 /tmp/dir3 /tmp/dir2 /tmp/dir1
```

此时目录堆栈的包含以下目录

```ini
/tmp/dir4 
/tmp/dir3 
/tmp/dir2 
/tmp/dir1
```

推送到堆栈的最后一个目录将位于顶部。当执行 popd 时，它将 cd 到堆栈中的顶级目录条目并将其从堆栈中删除。如上所示，最后压入堆栈的目录是/tmp/dir4。因此，当我们执行 popd 时，它将 cd 到 /tmp/dir4 并将其从目录堆栈中删除，如下所示。

```bash
# popd 
# pwd 
/tmp/dir4 

[Note: After the above popd, directory Stack Contains: 
/tmp/dir3 
/tmp/dir2 
/tmp/dir1] 

# popd 
# pwd 
/tmp/dir3 

[Note: After the above popd, directory Stack Contains: 
/tmp/dir2 
/tmp/dir1] 

# popd
# pwd 
/tmp/dir2 

[Note: After the above popd, directory Stack Contains: 
/tmp/dir1] 

# popd 
# pwd 
/tmp/dir1 

[Note: After the above popd, directory Stack is empty!] 

# popd 
-bash: popd: directory stack empty
```

### 1.6 "shopt -s cdspell"自动修正输入错误的目录名(good)

```shell
##shopt属于 bash shell 的内建命令，是shell options的缩写。
##centos7.9
#如下进入一个不存在的目录会报错
cd /etc/nt
-bash: cd: /etc/nt: No such file or directory

## 配置shell cdspell
# shopt -s cdspell

# cd /etc/nt
/etc/nt

# pwd
/etc/ntp
```

## 二：Date Manipulation（日期操作）

### 2.1 设置系统日期和时间

```bash
#设置系统日期时间
date {mmddhhmiyyyy.ss}
#以默认格式’月日时分年秒‘设置日期时间
date 013113142020.00
```

- mm 月
- dd    日
- hh    24小时格式
- mi    分钟
- yyyy 年
- ss      秒

设置日期

```bash
#以指定格式‘年月日’设置日期
date +%Y%m%d -s "20200131"
#以多种形式设置日期
date -s "01/31/2020 13:14:00"
date -s "31 JAN 2020 13:14:00"
date set "31 JAN 2020 13:14:00"
```

设置时间

```bash
#设置时间'时分秒'
date +%T -s "13:14:00"
#%p选项设置AM或PM
date +%T%p -s "01:14:00PM"
```

### 2.2 设置硬件日期和时间

设置硬件时钟时需确保系统时间正确

使用 `hwclock` 不带任何参数，查看当前系统的硬件时间

```bash
hwclcok
```

以系统时间设置硬件时钟使用如下命令

```bash
hwclock --systohc  
#或者使用  
hwclock -w
```

### 2.3 按指定格式显示当前日期和时间

以不同方式不同格式显示当前日期和时间

```bash
# date
Mon Apr 18 09:03:10 CST 2022

# date --date="now"
Mon Apr 18 09:03:32 CST 2022

# date --date="today"
Mon Apr 18 09:03:56 CST 2022

# date --date='1970-01-01 00:00:01 UTC +5 hours' +%s
18001

# date '+Current Date: %m/%d/%y%nCurrent Time:%H:%M:%S'
Current Date: 04/18/22
Current Time:09:07:02

# date +"%d-%m-%Y"
18-04-2022

# date +"%d/%m/%Y"
18/04/2022

# date +"%A,%B %d %Y"
Monday,April 18 2022
```

以下是可以传递给date命令的格式选项

- %D date (mm/dd/yy)
- %d   date of month (01..31)
- %m  month (01..12)
- %y    last two digits of year (00..99)
- %a   语言环境下的星期名称缩写 （Sun..Sat)
- %A   语言环境下的星期名称全拼   (Sunday..Saturday)
- %b   语言环境下的月份名称缩写   (Jan..Dec)
- %B   语言环境下的月份名称全拼  (January..December)
- %H   hour (00..23)
- %I    hour (01..12)
- %Y   year  (1970..)

### 2.4 显示过去的日期和时间

```bash
# date --date='5 seconds ago'
Mon Apr 18 09:17:52 CST 2022

# date --date='5 minutes ago'
Mon Apr 18 09:13:14 CST 2022

# date --date='1 day ago'
Sun Apr 17 09:18:35 CST 2022

# date --date="1 days ago"
Sun Apr 17 09:18:35 CST 2022

# date --date='1 months ago'
Fri Mar 18 09:18:51 CST 2022

# date --date='1 years ago'
Sun Apr 18 09:19:32 CST 2021

# date --date="yesterday"
Sun Apr 17 09:20:08 CST 2022

# date --date="12 months ago 2 days ago"  #区别于之前流传的，需要在每个时间之后加ago
Fri 16 Apr 2021 09:26:45 AM CST
```

### 2.5 显示未来的日期和时间

```bash
# date
Mon Apr 18 09:30:33 CST 2022

# date --date='60 seconds'
Mon Apr 18 09:32:07 CST 2022

# date --date='4 hours'
Mon Apr 18 13:31:27 CST 2022

# date --date='tomorrow'
Tue Apr 19 09:32:15 CST 2022

# date --date='1 day';date --date='1 days'
Tue Apr 19 09:32:32 CST 2022
Tue Apr 19 09:32:32 CST 2022

# date --date='1 week'
Mon Apr 25 09:33:56 CST 2022

# date --date='1 month'
Wed May 18 09:34:10 CST 2022

# date --date='2 years'
Thu Apr 18 09:34:40 CST 2024

# date --date='next day'
Tue Apr 19 09:35:04 CST 2022

# date --date='-1 days ago'      #用负数表示未来时间
Tue Apr 19 09:35:51 CST 2022

# date --date='this wednesday'   
Wed Apr 20 00:00:00 CST 2022
```

## 三：SSH Client Commands（SSH客户端命令）

### 3.1 识别SSH客户端版本

下面的例子表明系统在使用openssh，还有ssh2等客户端类型。

```shell
# ssh -V
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017
```

### 3.2 SSH 登录远程主机

### 3.3 调试SSH客户端会话

有时需要查看调试消息来解决任何 SSH 连接问题。如下所示将 -v（小写 v）选项传递给 ssh 以查看 ssh 调试消息。

```shell
ssh -v tony@magedu
OpenSSH_8.6p1, LibreSSL 3.3.5
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 21: include /etc/ssh/ssh_config.d/* matched no files
debug1: /etc/ssh/ssh_config line 54: Applying options for *
debug1: Authenticator provider $SSH_SK_PROVIDER did not resolve; disabling
debug1: Connecting to magedu port 22.
debug1: Connection established.
```

### 3.4 使用SSH转义字符切换SSH会话

当使用 ssh 从本地主机登录到远程主机时，可能希望回到本地主机执行一些活动并再次回到远程主机。在这种情况下，不需要断开与远程主机的 ssh 会话。相反，请按照以下步骤操作。

通过在远程主机输入转义字符 `~ ` 和 \<Control-Z> ，当输入 `~`  时将不会立刻在屏幕上看到它，直到输入了 \<Control-Z> 并回车。

```shell
tony@magedu:~$ ~^Z [suspend ssh]
[1]  + 66526 suspended  ssh tony@magedu

localhost:~$
```

回到 localhost后，ssh remotehost 客户端会话作为典型的 UNIX 后台作业运行，可以如下所示进行检查：

```shell
localhost:~$ jobs
[1]  + suspended  ssh tony@magedu
```

通过将后台 ssh remotehost 会话作业带到 localhost 的前台，无需再次输入密码即可返回远程主机 ssh。

```shell
localhost:~$ fg %1
[1]  + 66526 continued  ssh tony@magedu

tony@magedu:~$
```

### 3.5 使用SSH转义字符统计SSH会话

以下仅适用于 SSH2 客户端。要获取有关当前 ssh 会话的一些有用统计信息，执行以下操作。

首先登录远程主机，然后输入转义字符 `~` 和 s 并回车。这将会显示关于当前SSH 连接的有用统计信息，转义字符 `~` 在刚刚输入时不会立刻出现，直到输入s并回车。

```shell
remote:~$ 
 remote host: remotehost 
 local host: localhost 
 remote version: SSH-1.99-OpenSSH_3.9p1 
 local version: SSH-2.0-3.2.9.1 SSH Secure 
Shell (non-commercial) 
 compressed bytes in: 1506 
 uncompressed bytes in: 1622 
 compressed bytes out: 4997 
 uncompressed bytes out: 5118 
 packets in: 15 
 packets out: 24 
 rekeys: 0 
 Algorithms: 
 Chosen key exchange algorithm: diffie-hellmangroup1-sha1 
 Chosen host key algorithm: ssh-dss 
 Common host key algorithms: ssh-dss,ssh-rsa 
 Algorithms client to server: 
 Cipher: aes128-cbc 
 MAC: hmac-sha1 
 Compression: zlib
 Algorithms server to client: 
 Cipher: aes128-cbc 
 MAC: hmac-sha1 
 Compression: zlib
```

## 四：Essential Linux Commands（基础Linux命令）

### 4.1 Grep 命令

grep 命令用于在文件中搜索特定文本。这是一个非常强大的命令，有很多选项。

```ini
语法: grep [options] pattern [files]
```

在下面示例中，grep 在 /etc/passwd 文件中查找文本 spool 并显示所有匹配的行。

```shell
grep spool /etc/passwd
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
```

选项 -v，将显示除匹配项之外的所有行。在下面的示例中，它显示了 /etc/password 中与 o 不匹配的所有记录。

```shell
grep -v o /etc/passwd
sync:x:4:65534:sync:/bin:/bin/sync
speech-dispatcher:x:119:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false
hplip:x:129:7:HPLIP system user,,,:/run/hplip:/bin/false
```

选项-c，可以将匹配的行数获取出来

```
grep -c spool /etc/passwd
3
```

选项-i ，忽略文本中的大小写

```shell
grep -i chrony /etc/passwd
_chrony:x:132:136:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
```

选项 -r (递归）。在下面的示例中，它将通过忽略 /home/users 下所有子目录中的大小写来搜索文本“John”。

```shell
grep -ri john /home/users 
/home/users/subdir1/letter.txt:John, Thanks for your 
contribution. 
/home/users/name_list.txt:John Smith
/home/users/name_list.txt:John Doe
```

选项 -l ，与 -ri 结合，将只显示符合条件的文件名称，如下：

```shell
grep -ril john /root 
/home/users/subdir1/letter.txt 
/home/users/name_list.txt
```

### 4.2 Find 命令

find 是常用的命令，用于根据众多条件在 UNIX 文件系统中查找文件。接下来介绍一些 find 命令的实践示例。

```ini
语法: find [pathnames] [conditions]
```

下面一条命令将查找/etc目录下所有文件名中包含mail的文件

```shell
find /etc/ -name "*mail*"
```

下面命令将查找系统中文件大小大于100MB的文件

```shell
find / -type f -size +100M
```

查找X天未修改的文件，下面命令将查找当前目录下60天前修改的文件

```shell
find . -mtime +60
```

查找最近X天修改过的文件，下面命令将查找当前目录下近2天修改过的所有文件

```shell
find . -mtime -2
```

删除文件扩展名为"*.tar.gz"且大于 100MB的存档

```shell
find / -type f -name *.tar.gz -size +100M -exec ls -l {} \;
find / -type f -name *.tar.gz -size +100M -exec rm -f {} \;
```

固定目录深度进行查找

```shell
find /dir -maxdepth 3 -mindepth 3 
```

如何归档过去X天未修改的文件呢？以下命令在 /home/jsmith 目录下查找过去 60 天内未修改的所有文件，并在 /tmp 下以 yyyymmdd_archive.tar 格式创建存档文件。

```shell
find /home/jsmith -type f -mtime +60 | xargs tar -cvf /tmp/`date '+%Y%m%d'_archive.tar`
```

### 4.3 抑制标准输出和错误信息

有时在调试 shell 脚本时，可能不想看到标准输出或标准错误消息。如下所示使用 /dev/null 来抑制输出。

- 抑制标准输出  >/dev/null
  
  ```shell
  #不想看到echo输出，只想看到错误输出
  ./shell-script.sh >/dev/null
  ```

- 抑制错误输出  2>/dev/null
  
  ```shell
  #只想看到标准输出，而不想看到错误输出
  ./shell-script.sh 2>/dev/null
  ```

### 4.4 Join 命令

Join 命令基于一个公共字段组合来自两个文件的行。在下面的例子中，我们有两个文件——employee.txt 和salary.txt。两者都将员工 ID 作为公共字段。所以，我们可以使用join命令通过employee-id公共字段来组合这两个文件中的数据，如下所示。

```shell
# cat employee.txt
100 Jason Smith
200 John Doe
300 Sanjay Gupta
400 Ashok Sharma

# cat salary.txt
100 ,$5000
200 ,$500
300 ,$3000
400 ,$1,250

# join employee.txt salary.txt
100 Jason Smith ,$5000
200 John Doe ,$500
300 Sanjay Gupta ,$3000
400 Ashok Sharma ,$1,250
```

### 4.5 字母大小写转换

```shell
###将字母转换为全大写
# cat <<EOF > employee.txt 
100 Jason Smith 
200 John Doe 
300 Sanjay Gupta 
400 Ashok Sharma
EOF

# tr a-z A-Z < employee.txt
100 JASON SMITH
200 JOHN DOE
300 SANJAY GUPTA
400 ASHOK SHARMA

###将字母转换为全小写
# cat <<EOF >department.txt
100 FINANCE
200 MARKETING
300 PRODUCT DEVELOPMENT
400 SALES
EOF

# tr A-Z a-z <department.txt
100 finance
200 marketing
300 product development
400 sales
```

### 4.6 Xargs 命令

xargs 是一个非常强大的命令，它获取命令的输出并将其作为另一个命令的参数传递。以下是一些关于如何有效使用xarg的实际示例。

1. 当您尝试使用 rm 删除太多文件时，可能会收到错误消息：/bin/rm 参数列表太长 。使用 xargs 来避免此问题。
   
   ```shell
   # find -print0 在标准输出上打印完整的文件名，后跟空字符（而不是 -print 使用的换行符）。 这样处理find输出的程序可以正确处理包含换行符或其他类型的空格的文件名。此选项对应于 xargs 的 -0 选项。
   # xargs -0 输入项以空字符而不是空格结尾，并且引号和反斜杠不是特殊的（每个字符都是按字面意思计算的）。 禁用文件字符串的结尾，该字符串被视为与任何其他参数一样。 当输入项可能包含空格、引号或反斜杠时很有用。 GNU find -print0 选项生成适合此模式的输入。
   
   find ~ -name ‘*.log’ -print0 | xargs -0 rm -f
   ```

2. 获取 /etc/ 下所有 *.conf 文件的列表。有不同的方法可以获得相同的结果。以下示例仅用于演示 xargs 的使用。 此示例中 find 命令的输出使用 xargs 逐个传递给 ls –l。
   
   ```shell
   find /etc/ -name "*.conf" | xargs ls -l
   ```

3. 如果有一个包含要下载的 URL 列表的文件，则可以使用 xargs，如下所示。
   
   ```shell
   cat url-list.txt | xargs wget –c
   ```

4. 找出所有jpg图像并将其存档。
   
   ```shell
   find / -name *.jpg -type f -print | xargs tar -cvzf images.tar.gz
   ```

5. 将所有图片复制到外部硬盘驱动器。
   
   ```shell
   # -n 每个命令行最多使用 max-args 参数
   # xargs -i选项已经弃用，建议使用-I选项替代。当 replace-str未指定，效果同 -I{}
   # -I 将初始参数中出现的 replace-str 替换为从标准输入读取的名称。此外，未加引号的空白不会终止输入项;相反，分隔符是换行符。  隐含 -x 和 -L 1 。
   # -x 如果超出大小则退出（参考-s选项指定每行接受的最大字符数，包括命令、参数等）
   # -L 指定每个命令行的非空白输入行
   ls *.jpg | xargs -n1 -i cp {} /external-hard-drive/directory
   ```

### 4.7 Sort 命令

排序命令对文本文件的行进行排序。以下是基于以下示例文本文件使用 sort 命令的几个实际示例，该文件包含员工信息格式如下

```ini
employee_name:employee_id:department_name
```

```shell
[root@mysql1 ~]# cat names.txt
Emma Thomas:100:Marketing
Alex Jason:200:Sales
Madison Randy:300:Product Development
Sanjay Gupta:400:Support
```

按升序对文本文件进行排序

```shell
[root@mysql1 ~]# sort names.txt
Alex Jason:200:Sales
Emma Thomas:100:Marketing
Madison Randy:300:Product Development
Sanjay Gupta:400:Support
```

按降序对文本文件进行排序

```shell
[root@mysql1 ~]# sort -r names.txt
Sanjay Gupta:400:Support
Madison Randy:300:Product Development
Emma Thomas:100:Marketing
Alex Jason:200:Sales
```

按分隔符:  分隔的第二列排序

```shell
[root@mysql1 ~]# sort -t: -k 2 names.txt
Emma Thomas:100:Marketing
Alex Jason:200:Sales
Madison Randy:300:Product Development
Sanjay Gupta:400:Support
```

对第 3 个字段（department_name）上以制表符分隔的文本文件进行排序并禁止重复项

```shell
使用 -c，检查严格的顺序；没有 -c，只输出第一个
[root@mysql1 ~]# cat names.txt
Emma Thomas:100:Marketing
Alex Jason:200:Sales
Alex Jason2:200:Sales
Madison Randy:300:Product Development
Sanjay Gupta:400:Support

[root@mysql1 ~]# sort -t: -u -k3 names.txt
Emma Thomas:100:Marketing
Madison Randy:300:Product Development
Alex Jason:200:Sales
Sanjay Gupta:400:Support
```

按第 3 个字段（(numeric userid）对 passwd 文件进行排序

```shell
##sort 如果不加入n选项，则会出现1后面是11，12的情况，即不按数字大小排序。
[root@mysql1 ~]# sort -t: -k 3n /etc/passwd | head -n10
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
```

将 Sort 与其他命令结合使用

- ps –ef | sort : Sort the output of process list 

### 4.8 Uniq 命令

Uniq 命令主要与 Sort 命令结合使用，因为 uniq 仅从排序文件中删除重复项。即，为了使uniq工作，所有重复的条目都应该在相邻的行中。以下是一些常见示例。

```ini
[root@mysql1 ~]# cat namesd.txt
Emma Thomas:100:Marketing
Alex Jason:200:Sales
Madison Randy:300:Product Development
Emma Thomas:100:Marketing
Alex Jason:200:Sales
Sanjay Gupta:400:Support
Nisha Singh:500:Sales
```

1. 当文件包含重复条目时，可以执行以下操作来删除重复项。
   
   ```shell
   sort names.txt | uniq
   #同下
   sort -u names.txt
   ```

2. 如果想知道有多少行是重复的，请执行以下操作。以下示例中的第一个字段指示为该特定行找到的重复项数。因此，在这个例子中，以Alex和Emma开头的行在命名.txt文件中被找到了两次。
   
   ```shell
   [root@mysql1 ~]# sort namesd.txt | uniq -c
         2 Alex Jason:200:Sales
         2 Emma Thomas:100:Marketing
         1 Madison Randy:300:Product Development
         1 Nisha Singh:500:Sales
         1 Sanjay Gupta:400:Support
   ```

3. 以下内容仅显示重复的条目。
   
   ```shell
   # -c 按出现次数在每行添加前缀
   # -d 仅打印重复行，每组一行
   [root@mysql1 ~]# sort namesd.txt | uniq -cd
         2 Alex Jason:200:Sales
         2 Emma Thomas:100:Marketing
   ```

### 4.9 Cut 命令

Cut 命令可用于仅显示文本文件或其他命令输出中的特定列。下面是一些示例

显示冒号分隔文件中的第 1 个字段

```shell
[root@mysql1 ~]# cut -d: -f 1 /etc/passwd
root
bin
daemon
adm
lp
```

显示冒号分隔文件中的第 1 个和第 3 个字段

```shell
[root@mysql1 ~]# cut -d: -f 1,3 /etc/passwd
root:0
bin:1
daemon:2
adm:3
lp:4
```

仅显示文件中每行的前 8 个字符

```shell
[root@mysql1 ~]# cut -c 1-8 /etc/passwd
root:x:0
bin:x:1:
daemon:x
adm:x:3:
lp:x:4:7
```

杂项：显示系统上可用的总内存

```shell
#free显示内存，tr去除重复的空格，sed保留Mem行,cut获取第二列
[root@mysql1 ~]# free  | tr -s ' ' | sed '/^Mem/!d' | cut -d" " -f2
32944252
```

### 4.10 Stat 命令

Stat 命令可用于检查单个文件或文件系统的状态/属性。

查看一个文件或目录的状态属性

```shell
[root@mysql1 ~]# stat /etc/my.cnf
  File: ‘/etc/my.cnf’
  Size: 370           Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d    Inode: 15991278    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-04-08 22:41:00.080934292 +0800
Modify: 2022-02-08 04:43:40.000000000 +0800
Change: 2022-04-06 01:27:34.811897462 +0800
 Birth: -

[root@mysql1 ~]# stat /root
  File: ‘/root’
  Size: 4096          Blocks: 8          IO Block: 4096   directory
Device: 802h/2050d    Inode: 9175041     Links: 4
Access: (0550/dr-xr-x---)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2022-04-24 08:35:57.358222416 +0800
Modify: 2022-04-12 16:33:29.660214993 +0800
Change: 2022-04-12 16:33:29.660214993 +0800
 Birth: -
```

查看文件系统，通过选项 -f 或 --file-system

```shell
#ext4显示的是 ext2/ext3
[root@mysql1 ~]# stat -f /
  File: "/"
    ID: 3bc5c6c66d8715e1 Namelen: 255     Type: ext2/ext3
Block size: 4096       Fundamental block size: 4096
Blocks: Total: 77117920   Free: 70154425   Available: 66231289
Inodes: Total: 19595264   Free: 19526880
```

查看权限，通过选项 -c  指定具体格式<font color=blue>(有用，有用)</font>

```shell
#%A 以人类可读的方式显示访问权限
[root@mysql1 ~]# stat -c %A /
dr-xr-xr-x
```

### 4.11 Diff 命令

diff 命令比较两个不同的文件并报告差异。输出非常隐晦，不能直接阅读。

> 语法：diff [options] file1 file2

**如何比对一个新文件和旧文件的修改差异呢？**

diff 命令中的选项 `-w` 将在执行比较时忽略空格。

在diff的输出中：

- 在 diff 命令中，---上面的行指示着第一个文件（即name_list.txt）中发生的更改。

- 在 diff 命令中，---下面的行指示着第二个文件（即name_list_new.txt）中所做的更改。属于第一个文件的行以 `<` 开头，第二个文件的行以 `>` 开头。

```shell
$ cat name_list.txt
John Doe

$ cat name_list_new.txt
John M Doe
Jason Bourne

$ diff -w name_list.txt name_list_new.txt
1c1,2
< John Doe
---
> John M Doe
> Jason Bourne
```

### 4.12 显示用户的总连接时间

Ac 命令将显示有关用户连接时间的统计信息。

**当前登录用户的连接时间**

使用选项 –d，它将分解各个天的输出。在此示例中，我今天已登录到系统超过 6 个小时。4月23日，我登录了大约17个小时！

```shell
$ ac -d
Apr 23    total       17.73
Apr 24    total       13.26
Today    total         6.37
```

**所有用户的连接时间**

```shell
$ ac -p 
  john                 3.64 
  madison         0.06 
  sanjay             88.17 
  nisha             105.92
  ramesh             111.42 
  total 309.21
```

**特定用户的连接时间**

若要获取特定用户的连接时间报告，执行以下命令：

```shell
$ ac -d sanjay 
Jul 2 total 12.85 
Aug 25 total 5.05 
Sep 3 total 1.03 
Sep 4 total 5.37 
Dec 24 total 8.15 
Dec 29 total 1.42 
Today total 2.95
```

## 五：PS1，PS2，PS3，PS4 And PROMPT_COMMAND

### 5.1 PS1-默认交互提示

Linux 上的默认交互式提示可以修改为有用且信息丰富的内容。在以下示例中，默认 PS1 是“[\u@\h \W]\\$”，它显示用户名、主机名和当前工作目录名称。

```shell
[root@centos-223-148 ~]# echo $PS1
[\u@\h \W]\$

##修改添加shell名称和版本号
[root@centos-223-148 ~]# export PS1="[\u@\h \s-\v \W]\$"
[root@centos-223-148 -bash-4.2 ~]$
```

- \u  用户名
- \h   主机名
- \w  当前目录的全路径，在主目录的时候只显示为~

请注意，PS1的值末尾有一个空格。个人而言，我更喜欢在提示符的末尾有一个空格，以提高可读性。

通过将 PS1=“\u@\h \w> ”添加到 .bash_profile （或） .bashrc，使设置永久化。

### 5.2 PS2-继续交互提示

一个很长的命令可以通过在行的末尾给出 \来分解为多行。多行命令的默认交互式提示是“>”。让我们使用 PS2 环境变量更改此默认行为，以显示“continue-> ”，如下所示。

```shell
[root@centos-223-148 ~]# kubeadm init --image-repository registry.aliyuncs.com/google_containers \
>      --kubernetes-version v1.20.0 \
>      --control-plane-endpoint k8s-api.ilinux.io \
>      --apiserver-advertise-address 172.29.9.5 \
>      --pod-network-cidr 10.244.0.0/16 \
>      --token-ttl 0

export PS2='continue-'

[root@centos-223-148 ~]# [root@centos-223-148 ~]# kubeadm init --image-repository registry.aliyuncs.com/google_containers \
continue->      --kubernetes-version v1.20.0 \
continue->      --control-plane-endpoint k8s-api.ilinux.io \
continue->      --apiserver-advertise-address 172.29.9.5 \
continue->      --pod-network-cidr 10.244.0.0/16 \
continue->      --token-ttl 0
```

当使用\将长命令分解为多行时，这个非常有用且易于阅读。当然也有人不喜欢分解长命令。

### 5.3 PS3-脚本中的选择提示符

使用 PS3 环境变量对 shell 脚本中的选择循环提示符进行自定义，如下所示。

```shell
[root@centos-223-148 ~]# cat ps3.sh
select i in mon tue wed exit
do
 case $i in
 mon) echo "Monday";;
 tue) echo "Tuesday";;
 wed) echo "Wednesday";;
 exit) exit;;
 esac
done

##如下默认显示"#?"作为选择提示符
[root@centos-223-148 ~]# bash ps3.sh
1) mon
2) tue
3) wed
4) exit
#? 1
Monday
#? 4

##接下来通过设置PS3环境变量调整选择提示符
[root@centos-223-148 ~]# cat ps3.sh
PS3="Select a day (1-4): "
select i in mon tue wed exit
do
 case $i in
 mon) echo "Monday";;
 tue) echo "Tuesday";;
 wed) echo "Wednesday";;
 exit) exit;;
 esac
done
#这里调整为显示"Select a day (1-4):"作为选择提示符
[root@centos-223-148 ~]# bash ps3.sh
1) mon
2) tue
3) wed
4) exit
Select a day (1-4): 1
Monday
Select a day (1-4): 4
```

### 5.4 PS4-由“set -x”用于作为跟踪输出的前缀

PS4 shell 变量定义了在调试模式下执行 shell 脚本时显示的提示，如下所示。

```shell
[root@centos-223-148 ~]# cat ps4.sh
set -x
echo "PS4 demo script"
ls -l /etc/ | wc -l
du -sh ~

[root@centos-223-148 ~]# ./ps4.sh
++ echo 'PS4 demo script'
PS4 demo script
++ ls -l /etc/
++ wc -l
176
++ du -sh /root
52K    /root

##当使用 set -x进行跟踪输出时显示输出默认的 "++" 
```

带有 PS4 的 Shell 脚本和输出:

下面在 ps4.sh 中定义的PS4具有以下两个代码：

- $0 - 表示脚本的名称 
- $LINENO - 显示脚本中的当前行号

```shell
[root@centos-223-148 ~]# cat ps4.sh
export PS4='$0.$LINENO+ '
set -x
echo "PS4 demo script"
ls -l /etc/ | wc -l
du -sh ~

[root@centos-223-148 ~]# ./ps4.sh
../ps4.sh.3+ echo 'PS4 demo script'
PS4 demo script
../ps4.sh.4+ ls -l /etc/
../ps4.sh.4+ wc -l
176
../ps4.sh.5+ du -sh /root
52K    /root
## 这里显示修改的脚本名称+脚本行号
```

### 5.5 PROMPT_COMMAND提示命令

Bash shell 在显示 PS1 变量之前执行PROMPT_COMMAND的内容。

```shell
[root@centos-223-148 ~]# export PROMPT_COMMAND="date +%k:%m:%S"
 9:05:16
[root@centos-223-148 ~]#

##这里将在不同的行显示PROMPT_COMMAND and PS1 的输出内容
```

如果要在与PS1相同的行中显示PROMPT_COMMAND的值，请使用echo -n，如下所示。

```shell
[root@centos-223-148 ~]# export PROMPT_COMMAND="echo -n [$(date +%k:%m:%S)]"
[ 9:05:58][root@centos-223-148 ~]#

##这里将在同一行显示PROMPT_COMMAND and PS1 的输出内容
```

## 六：Colorful and Functional shell prompt using PS1

### 6.1 在提示符下显示目录的用户名、主机名和目录基名

此示例中的 PS1 在提示符中显示以下三个信息：

- \u – 用户名
- \h – 主机名 
- \W – 当前工作目录的基本名称

```shell
-bash-5.0.17$ export PS1="\u@\h \W> "

saw@test ~> cd /tmp
saw@test tmp>
```

### 6.2 在提示符下显示当前时间

在PS1环境变量中，可以直接执行任何Linux命令，通过以 $(linux_command) 格式指定。在下面的例子，通过执行命令 $(date) 来在提示中显示当前时间。

```shell
[tony@dev-db ~]# export PS1="\$(date +%k:%m:%S) [\u@\h \W]#"
18:05:52 [root@centos-223-148 ~]#
```

还可以使用 \t 以 hh：mm：ss 格式显示当前时间，如下所示：

```shell
[tony@dev-db ~]# $export PS1="[\u@\h \W [\t]>"
[root@centos-223-148 ~ [18:07:02]>
```

还可以使用 \@ 以 12 小时 am/pm 格式显示当前时间，如下所示：

```shell
[tony@dev-db ~]# export PS1="[\@] \u@\h> "
[07:07 PM] root@centos-223-148>
```

### 6.3 在提示符下显示任何命令的输出

可以在提示符中显示任何 Linux 命令的输出。下面的示例显示三个在提示符下用|（管道符）分隔的项

- \\!  : 历史命令的记录编号
- \\h : 主机名
- $kernel_version：uname -r 命令从 $kernel_version 变量的输出
- \\$?  : 最后一个命令的状态

```shell
[root@app-test ~]# kernel_version=$(uname -r)
[root@app-test ~]# echo $kernel_version
3.10.0-1160.49.1.el7.x86_64
[root@app-test ~]# export PS1="\!|\h|$kernel_version|\$?> "
1001|app-test|3.10.0-1160.49.1.el7.x86_64|0>
```

### 6.4 更改提示符的前景色

以蓝色显示提示符，以及用户名、主机和当前目录信息

```shell
##显示亮蓝
$ export PS1="\e[0;34m\u@\h \w> \e[m "
##显示暗蓝
$ export PS1="\e[1;34m\u@\h \w> \e[m "
```

- \\e[    表示颜色提示的开始
- x;ym - 表示颜色代码。下面给出部分颜色代码值。
- \e[m  表示颜色提示的结束

**颜色代码表**

```ini
## 使用深色，可将0替换为1
Black 0;30 
Blue 0;34 
Green 0;32 
Cyan 0;36 
Red 0;31 
Purple 0;35 
Brown 0;33
```

通过添加以下行~/.bash_profile或~/.bashrc，使颜色更改永久化

```ini
$ vi ~/.bash_profile

STARTCOLOR='\e[0;34m'; 
ENDCOLOR="\e[0m" 
export PS1="$STARTCOLOR\u@\h \w> $ENDCOLOR"
```

### 6.5 更改提示符的背景色

通过在 PS1 提示符中指定  \e[{code}m  来更改背景颜色，如下所示。

```shell
$ export PS1="\e[47m\u@\h \w> \e[m "
```

前景与背景相结合

```shell
$ export PS1="\e[0;34m\e[47m\u@\h \w> \e[m "
```

将以下内容添加到您的 ~/.bash_profile 或 ~/.bashrc 以使上述背景和前景色永久化。

```ini
$ vi ~/.bash_profile 

STARTFGCOLOR='\e[0;34m'; 
STARTBGCOLOR="\e[47m" 
ENDCOLOR="\e[0m" 
export PS1="\[$STARTFGCOLOR$STARTBGCOLOR\u@\h \w>$ENDCOLOR \]"

##\[   \]  
#PS1定义前后使用\[和\]包裹起来，使回行工作正常。
```

使用以下背景颜色进行尝试，然后选择符合适合的颜色：

```ini
\e[40m 
\e[41m 
\e[42m 
\e[43m 
\e[44m  
\e[45m 
\e[46m 
\e[47m 
```

### 6.6 更改提示符显示多种颜色(超人色)

可以在同一提示中显示多种颜色。将以下函数添加到 ~/.bash_profile

```ini
function prompt {
  local BLUE="\[\033[0;34m\]"
  local DARK_BLUE="\[\033[1;34m\]"
  local RED="\[\033[0;31m\]"
  local DARK_RED="\[\033[1;31m\]"
  local NO_COLOR="\[\033[0m\]"
  case $TERM in
    xterm*|rxvt*)
      TITLEBAR='\[\033]0;\u@\h: \w\007\]'
      ;;
    *)
      TITLEBAR=""
      ;;
  esac
  PS1="\u@\h [\t]> "
  PS1="${TITLEBAR}$BLUE\u@\h $RED[\t]>$NO_COLOR "
  PS2=' continue-> '
  PS4=' $0.$LINENO+ '
}
prompt
```

然后重新登录以使更改生效或按如下所示 source  .bash_profile

```shell
source ~/.bash_profile
root@centos7 [09:27:06]>
```

### 6.7 使用 tput 更改提示符颜色

可以使用 tput 更改 PS1 提示的颜色，如下所示：

```shell
$  export PS1="\[$(tput bold)$(tput setb 4)$(tput setaf 7)\]\u@\h:\w $ \[$(tput sgr0)\]"
```

**tput 支持的颜色设置:**

```ini
tput setab [1-7] - 使用ANSI escape设置背景色
tput setb [1-7] - 设置背景色
tput setaf [1-7] - 使用ANSI escape设置前景色
tput setf [1-7] - 设置前景色
```

**tput 支持的文字属性:**

```ini
tput bold - 加粗
tput dim - 半亮模式
tput smul - 进入下划线模式
tput rmul - 退出下划线模式
tput rev - 开启反向模式
tput smso - 进入标准输出模式(bold on rxvt) 
tput rmso - 退出标准输出模式
tput sgr0 - 关闭所有文字属性
```

**tput 的颜色代码:**

```ini
0 – 黑
1 – 红
2 – 绿
3 – 黄
4 – 蓝
5 – 洋红
6 – 蓝绿
7 - 白
```

### 6.8 使用PS1变量的可用代码创建自己的提示符

使用以下代码并创建自己的个人 PS1 Linux 提示符，该提示符实用且符合自己的口味。

```ini
\a  ASCII 响铃字符(07)
\d  “Weekday Month Date”格式的日期（例如，“Tue May 26”）
\D{format}  将格式传递给 strftime(3) 并将结果插入到提示字符串中；空格式会产生特定于语言环境的时间表示。其中大括号是必需的。
\e  ASCII 转义字符(033)
\h  主机名的第一部分(.分隔符)
\H  主机名
\j  当前由 shell 管理的作业数
\l  shell 终端设备名称的基本名称
\n  换行符
\r  回车符
\s  shell 的名称，$0 的基本名称（最后一个斜杠后面的部分）
\t  24 小时制 HH:MM:SS 格式的当前时间
\T  12 小时制 HH:MM:SS 格式的当前时间
\@  12 小时上午/下午格式的当前时间
\A  24 小时制 HH:MM 格式的当前时间
\u  当前用户的用户名
\v  bash 的版本（例如，5.00）
\V  bash 的发布，版本 + 补丁级别（例如，5.00.0）
\w  当前工作目录，$HOME 缩写为波浪号
\W  当前工作目录的基本名称，$HOME 缩写为波浪号
\！ 该命令的history编号
\#  该命令的命令号
\$  如果有效 UID 为 0，则为 #，否则为 $
\nnn 八进制数 nnn 对应的字符
\\  反斜杠字符
\[  开始一系列非打印字符，用于将终端控制序列嵌入到prompt提示中
\]  结束一系列非打印字符
```

### 6.9 在 PS1 变量中使用 bash shell 函数

还可以在 PS1 中调用 bash shell 函数，如下所示。

### 6.10 在 PS1 变量中使用 shell 脚本

```ini
# 向上换行符
\n

# 中括号开始
\e[1;37m
[
\e[m

#用户
\e[1;32m
\u
\e[m

#@分隔符
\e[1;33m
@
\e[m

#主机全名
\e[1;35m
\H
\e[m

#此处有空格

#当前完整路径
\e[4m
`pwd`
\e[m

#中括号标示结束
\e[1;37m
]
\e[m

\e[1;36m
\e[m

# 结尾换行
\n

# 识别是root 或者是普通用户
\$
```

## 七：Archive and Compression（存档和压缩）

### 7.1 Zip命令基础知识

#### 7.1.1 如何压缩多个文件

```ini
语法：  zip {.zip file-name} {file-names}
```

```shell
# zip var-log-files.zip /var/log/*
  adding: var/log/anaconda/ (stored 0%)
  adding: var/log/audit/ (stored 0%)
  adding: var/log/boot.log (stored 0%)
  adding: var/log/boot.log-20220506 (deflated 94%)
  adding: var/log/btmp (stored 0%)
  adding: var/log/chrony/ (stored 0%)
  adding: var/log/cron (deflated 93%)
  adding: var/log/cron-20220508 (deflated 92%)
  adding: var/log/dmesg (deflated 70%)
  adding: var/log/dmesg.old (deflated 70%)
  adding: var/log/firewalld (deflated 76%)
  ......
```

#### 7.1.2 如何递归压缩目录及其文件

```shell
# zip -r var-log-dir.zip /var/log/
  adding: var/log/ (stored 0%)
  adding: var/log/tallylog (deflated 100%)
  adding: var/log/tuned/ (stored 0%)
  adding: var/log/tuned/tuned.log (deflated 88%)
  adding: var/log/anaconda/ (stored 0%)
  adding: var/log/anaconda/program.log (deflated 84%)
  adding: var/log/anaconda/journal.log (deflated 91%)
  adding: var/log/anaconda/storage.log (deflated 89%)
  ......
```

#### 7.1.3 如何解压缩 *.zip 压缩文件

```shell
# unzip var-log-dir.zip
Archive:  var-log-dir.zip
   creating: var/log/
  inflating: var/log/tallylog
   creating: var/log/tuned/
  inflating: var/log/tuned/tuned.log
   creating: var/log/anaconda/
  inflating: var/log/anaconda/program.log
  ......
```

要在解压缩过程中查看详细输出，请通过 -v 选项，如下所示。

```shell
# unzip -v var-log-files.zip
Archive:  var-log-files.zip
 Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
--------  ------  ------- ---- ---------- ----- --------  ----
       0  Stored        0   0% 03-08-2022 14:29 00000000  var/log/anaconda/
       0  Stored        0   0% 03-08-2022 20:30 00000000  var/log/audit/
       0  Stored        0   0% 05-06-2022 03:28 00000000  var/log/boot.log
......
       0  Stored        0   0% 03-08-2022 14:26 00000000  tmp/.X11-unix/
--------          -------  ---                            -------
 1582570           161655  90%                            32 files
```

#### 7.1.4 如何列出 zip 文件的内容

```shell
# unzip -l var-log-dir.zip
Archive:  var-log-dir.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  05-08-2022 03:37   var/log/
    64000  03-08-2022 20:47   var/log/tallylog
        0  03-08-2022 20:30   var/log/tuned/
    10407  05-05-2022 10:37   var/log/tuned/tuned.log
        0  03-08-2022 14:29   var/log/anaconda/
    50578  03-08-2022 14:29   var/log/anaconda/program.log
  1613279  03-08-2022 14:29   var/log/anaconda/journal.log
```

### 7.2 Zip命令的压缩级别

zip 命令提供了 10 级压缩

- 级别 0 是最低级别，它只是归档文件而不进行任何压缩。
- 级别 1 将执行很少的压缩。但是，会非常快。
- 级别 6 是默认的压缩级别。
- 9 级是最高压缩级别。与默认级别相比，这会更慢。除非压缩一个巨大的文件，否则应该总是使用 9 级。

在下面的示例中，在同一个目录上使用了 Level 0、默认 Level 6 和 Level 9 压缩。查看压缩文件的大小。

```shell
[root@centos-223-148 ~]# zip -0 var-log-0.zip /var/log/*
[root@centos-223-148 ~]# zip var-log-default.zip /var/log/*
[root@centos-223-148 ~]# zip -9 var-log-9.zip /var/log/*

[root@centos-223-148 ~]# ls -ltr var*
-rw-r--r-- 1 root root  178502 May 12 20:25 var-log-default.zip
-rw-r--r-- 1 root root  174158 May 12 20:26 var-log-9.zip
-rw-r--r-- 1 root root 1789401 May 12 20:26 var-log-0.zip
```

### 7.3 Zip文件的密码保护

将选项 -P 传递给 zip 命令为 zip 文件分配密码

```shell
zip -P mysecurepwd var-log-protected.zip /var/log/*
```

如果在 shell 脚本中使用命令来执行后台作业，则上述选项很好。但是，当在命令行上以交互方式执行压缩时，不希望密码在历史记录中可见。因此，使用选项 -e 来分配密码，如下所示

```shell
# zip -e var-log-passwd.zip /var/log/*
Enter password:
Verify password:
  adding: var/log/anaconda/ (stored 0%)
  adding: var/log/audit/ (stored 0%)
  adding: var/log/boot.log (stored 0%)
  adding: var/log/boot.log-20220506 (deflated 94%)
```

当解压缩受密码保护的文件时，它会要求输入密码，如下所示

```shell
# unzip var-log-passwd.zip
Archive:  var-log-passwd.zip
[var-log-passwd.zip] var/log/boot.log password:
```

### 7.4 验证Zip文件存档

有时可能想要验证 zip 存档而不解压缩它。要测试 zip 文件的有效性，可以通过选项 -t  如下所示

```shell
[root@centos-223-148 ~]# unzip -t var-log.zip
Archive:  var-log.zip
    testing: var/log/anaconda/        OK
    testing: var/log/audit/           OK
    ......
    No errors detected in compressed data of var-log.zip.
```

### 7.5 tar基础命令

tar 命令（磁带存档）用于将一组文件打包为存档。

```ini
语法： tar [options] [tar-archive-name]   [other-file-names]
```

如何在主目录下创建所有文件和子目录的单个备份文件？

以下命令在 /tmp 下创建一个名为 my_home_directory.tar 的存档备份文件。该存档将包含 /home/jack 下的所有文件和子目录。

- 选项 c，代表创建档案；
- 选项 v 代表详细模式，在执行命令时显示附加信息；
- 选项 f 表示命令中提到的归档文件名。

```shell
tar cvf /tmp/my_home_directory.tar /home/jack
```

如何查看 tar 存档中的所有文件？

- 选项 t 将显示 tar 存档中的所有文件。

```shell
 tar tvf /tmp/my_home_directory.tar
```

如何从 tar 存档中提取所有文件？

- 选项 x 将从 tar 存档中提取文件，如下所示。这会将内容提取到执行命令的当前目录位置。

```shell
tar xvf /tmp/my_home_directory.tar
```

如何将 tar.gz 文件提取到特定目录？

- 选项 C 将从 tar 存档文件到指定目录

```shell
tar xvfz /tmp/my_home_directory.tar.gz –C /home/ramesh
```

### 7.6 将 gzip、bzip2 与 tar 结合使用

1）如何将 gzip 与 tar 一起使用？

处理 tar.gz 压缩文件时，在 tar 命令中添加选项 z。

```shell
#创建
tar cvfz /tmp/my_home_directory.tar.gz /home/jsmith 
#解压
tar xvfz /tmp/my_home_directory.tar.gz 
#不解压查看文档
tar tvfz /tmp/my_home_directory.tar.gz
```

​      <font color=violetred>与 bzip2 相比，使用 gzip 更快</font>。

2）如何将 bzip2 与 tar 一起使用？

在处理 tar.bz2 压缩文件时，在 tar 命令中添加选项 j。

```shell
#创建
tar cvfj /tmp/my_home_directory.tar.bz2 /home/jsmith 
#解压
tar xvfj /tmp/my_home_directory.tar.bz2 
#不解压查看文档
tar tvfj /tmp/my_home_directory.tar.bz2
```

​      <font color=violetred>与 gzip 相比，使用 bizp2 可提供更高级别的压缩</font>。

## 八：Command Line History （命令行历史）

当经常使用 Linux 命令行时，有效地使用history可以大大提高生产力。事实上，一旦掌握了这里提供的 15 个示例，你会发现使用命令行更加享受和有趣。

### 8.1 使用 HISTTIMEFORMAT 在history记录中显示 TIMESTAMP

通常，当从命令行键入历史记录时，它会显示命令# 和命令。出于审计目的，将时间戳与命令一起显示可能是有益的，如下所示。

```shell
export HISTTIMEFORMAT=' %F %T '

    1   2022-05-13 09:42:30 ip a
    2   2022-05-13 09:42:30 cd /etc/sysconfig/network-scripts/
    3   2022-05-13 09:42:30 ls
    4   2022-05-13 09:42:30 mv ifcfg-ens3 ifcfg-eth0
```

还可以设置以下别名来查看最近的历史命令。

```shell
alias h1='history 10' 
alias h2='history 20' 
alias h3='history 30'
```

### 8.2 使用 Control+R 搜索历史记录

相信这应该是history最常用的功能，当已经执行了一个很长的命令时，可以简单地使用关键字搜索历史并重新执行相同的命令，而无需完全键入它。按 Control+R 并键入关键字。

在下面的示例中，搜索了 re，它查找history中包含字符串 re的命令， 显示了上一个命令“(reverse-i-search)`re': ps -ef | grep dble”。

```shell
(reverse-i-search)`re': ps -ef | grep dble
```

有时想在执行之前从历史记录中编辑命令。例如可以搜索httpd，从命令历史中会显示service httpd stop，选择这个命令，把stop改成start，然后重新执行，如下图。

```shell
# [Note: 按 Ctrl+R 显示 display the reverse-i-search 界面]
(reverse-i-search)`httpd‘: service httpd stop
service httpd start
```

### 8.3 使用 4 种不同的方法快速重复上一个命令

有时可能会因为各种原因重复前面的命令。以下是重复上次执行命令的 4 种不同方式。

1. 使用向上箭头查看上一个命令，然后按 enter 执行它；
2. 命令行输入 !! 并按回车
3. 命令行输入 !-1 并按 enter
4. 按  Control+P 会显示上一条命令，按回车执行

### 8.4 执行历史记录中的特定命令

在以下示例中，如果要重复命令 #4，执行 !4，如下所示。

```shell
# history | more 
 1 service network restart 
 2 exit 
 3 id 
 4 cat /etc/redhat-release

 # !4
cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
```

### 8.5 执行上一个以特定字母开头的命令

键入 ！后跟要重新执行的命令的开头几个字母。在下面的示例中，键入 !ps 并回车，执行前面以 ps 开头的命令，即 ps aux | grep yp

```shell
# !ps
ps aux | grep yp 
 root 16947 0.0 0.1 36516 1264 ? Sl 13:10 0:00 ypbind 
 root 17503 0.0 0.0 4124 740 pts/0 S+ 19:19 0:00 grep yp
```

### 8.6 使用 HISTSIZE 控制history记录的总行数

将以下两行附加到 .bash_profile 并再次重新登录到 bash shell 以查看更改。在此示例中，只有 450 个命令将存储在 bash 历史记录中。

```shell
# vi ~/.bash_profile 
HISTSIZE=450 
HISTFILESIZE=450
```

### 8.7 使用 HISTFILE 更改history文件名

默认情况下，历史记录存储在 ~/.bash_history 文件中。将以下行添加到 .bash_profile 并重新登录到 bash shell，以将历史命令存储在 .commandline_warrior 文件而不是 .bash_history 文件中。当想使用不同的历史文件名跟踪从不同终端执行的命令时，有一定使用场景。

```shell
# vi ~/.bash_profile 
HISTFILE=/root/.commandline_warrior
```

### 8.8 使用 HISTCONTROL 从history记录中消除连续重复的条目

在下面的例子中 pwd 被输入了 3 次，当输入history命令时，可以看到它连续出现的所有 3 次。要消除重复项，请将 HISTCONTROL 设置为 ignoreups，如下所示。

```shell
# pwd 
# pwd 
# pwd 
# history | tail -4 
 44 pwd 
 45 pwd 
 46 pwd 
 47 history | tail -4

# export HISTCONTROL=ignoredups
# pwd 
# pwd 
# pwd 
# history | tail -3 
 56 export HISTCONTROL=ignoredups 
 57 pwd 
 58 history | tail -4
 ##此时只出现一次pwd历史记录。我在测试过程中发现centos7.9已设置HISTCONTROL=ignoredups
```

### 8.9 使用 HISTCONTROL 擦除整个history记录中的重复项

上面显示的 ignoreups 仅在它们是连续命令时才会删除重复项。要消除整个历史记录中的重复项，请将 HISTCONTROL 设置为 erasedups，如下所示。

```shell
# export HISTCONTROL=erasedups 
# pwd 
# service httpd stop 
# history | tail -3 
 38 pwd 
 39 service httpd stop 
 40 history | tail -3

# ls -ltr 
# service httpd stop 
# history | tail -6 
 35 export HISTCONTROL=erasedups 
 36 pwd 
 37 history | tail -3 
 38 ls –ltr 
 39 service httpd stop 
 40 history | tail -6
```

可以看到pwd之前的 service httpd stop 命令记录被擦除了。

### 8.10 使用 HISTCONTROL 强制history不记住特定命令

执行命令时，可以通过将 HISTCONTROL 设置为 ignorespace 并在命令前面键入一个空格来指示history记录忽略该命令，如下所示。对此很多初级系统管理员感到兴奋，因为他们可以从history记录中隐藏命令。

了解ignorespace是如何运作的这个很好。但是，作为最佳实践，不要有目的地从历史中隐藏任何东西。

```shell
export HISTCONTROL=ignorespace 
ls –ltr 
pwd 
  service httpd stop 
# service命令前加入了空格，此条命令将在历史记录中被隐藏

# history | tail -3 
 67 ls –ltr 
 68 pwd 
 69 history | tail -3
```

### 8.11 使用选项 -c 清除所有以前的history记录

有时可能想清除所有以前的历史记录。但是，仍希望保持之后的历史记录。

```shell
history -c
```

### 8.12 替换history中的命令

在搜索历史记录时，可能希望使用刚刚搜索的命令中的参数但执行不同的命令。

在下面的示例中，vi 命令旁边的 !!:$ 获取从上一个命令的参数到当前命令。

```shell
# ls anaconda-ks.cfg
anaconda-ks.cfg

# vi !!:$
vi anaconda-ks.cfg
```

在下面的示例中，vi 命令旁边的 !^ 获取从上一个命令（即 cp 命令）的第一个参数到当前命令（即 vi 命令）。

```shell
# cp anaconda-ks.cfg anaconda-ks.cfg.bak 
anaconda-ks.cfg

# vi !^ 
vi anaconda-ks.cfg
```

### 8.13 用特定参数替换特定命令

在下面的示例中，!cp:2 搜索history中以 cp 开头的上一个命令，并采用 cp 的第二个参数并将其替换为 ls -l 命令，如下所示。

```shell
# cp ~/longname.txt /really/a/very/long/path/long-filename.txt

# ls -l !cp:2 
ls -l /really/a/very/long/path/long-filename.txt
```

在下面的示例中，!cp:$ 搜索history中以 cp 开头的上一个命令，并采用 cp 的最后一个参数（在本例中，也是如上所示的第二个参数）并将其替换为 ls -l命令如下图。

```shell
# ls -l !cp:$ 
ls -l /really/a/very/long/path/long-filename.txt
```

### 8.14 使用 HISTSIZE 禁用history

如果想禁用历史记录并且不希望 bash shell 记住输入的命令，请将 HISTSIZE 设置为 0，如下所示。

```shell
# export HISTSIZE=0
# history
```

### 8.15 使用 HISTIGNORE 忽略history中的特定命令

有时可能不想使用 pwd 和 ls 等基本命令来弄乱你的history记录。使用 HISTIGNORE 指定要从历史记录中忽略的所有命令。

请注意，将 ls 添加到 HISTIGNORE 只会忽略 ls 而不会忽略 ls -l。因此，必须提供每个想从历史记录中忽略的确切命令.

```shell
# export HISTIGNORE=”pwd:ls:ls –ltr:”
# pwd 
# ls 
# ls -ltr 
# service httpd stop 
# history | tail -3 
 79 export HISTIGNORE=”pwd:ls:ls -ltr:” 
 80 service httpd stop 
 81 history 
```

如上，这里没显示pwd和ls

## 九：System Administration Tasks（系统管理任务）

### 9.1 使用 fdisk 进行分区

在服务器上安装全新的磁盘后，必须使用 fdisk 等工具对其进行相应的分区。

以下在 fdisk 中执行的 5 个典型操作（命令）。

- n  -  创建新分区
- d  -  删除现有分区
- p  -  打印分区表
- w -  将更改写入分区表。即保存。
- q -  退出 fdisk 实用程序

**创建分区**

在以下示例中，创建了一个 /dev/sda1 主分区。

```bash
# fdisk /dev/sda
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel Building a new DOS disklabel. Changes will remain in memory only, until you decide to write them. After that, of course, 
the previous content won't be recoverable. 
The number of cylinders for this disk is set to 34893. There is nothing wrong with that, but this is larger than 1024, and could in certain setups cause problems with:

1) software that runs at boot time (e.g., old versions of LILO) 
2) booting and partitioning software from other OSs  (e.g., DOS FDISK, OS/2 FDISK) 
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): p
Disk /dev/sda: 287.0 GB, 287005343744 bytes 255 heads, 63 sectors/track, 34893 cylinders Units = cylinders of 16065 * 512 = 8225280 bytes 

Device Boot Start End Blocks Id System Command (m for help): n
Command action 
 e extended 
 p primary partition (1-4) 
p 
Partition number (1-4): 1
First cylinder (1-34893, default 1): 
Using default value 1 
Last cylinder or +size or +sizeM or +sizeK (1-34893, 
default 34893): 
Using default value 34893 

Command (m for help): w
The partition table has been altered! 

Calling ioctl() to re-read partition table. 
Syncing disks.


##验证分区是否已成功创建
Command (m for help): p

Disk /dev/sda: 287.0 GB, 287005343744 bytes 255 heads, 63 sectors/track, 34893 cylinders 
Units = cylinders of 16065 * 512 = 8225280 bytes 
Device Boot Start End     Blocks    Id  System 
/dev/sda1   1     34893   280277991 83  Linux
```

### 9.2 使用mke2fsk格式化分区

磁盘分区后，它还没有准备好使用，还需要格式化磁盘。在此阶段，如果尝试查看磁盘信息，则会给出以下错误消息，表明不存在有效的超级块。

```shell
# tune2fs -l /dev/sda1
tune2fs 1.35 (28-Feb-2004) 
tune2fs: Bad magic number in super-block while trying to open /dev/sda1 
Couldn't find valid filesystem superblock.
```

格式化磁盘，如下所示使用 mke2fs。

```shell
# mke2fs /dev/sda1
```

还可以将以下可选参数传递给 mke2fs。

- -m 0 : 保留块百分比 ：这表示为 root 用户保留的文件系统块的百分比。默认是5%。在这个示例中，它设置为0
- -b 4096 ： 以字节为单位指定的块大小。有效值为每块 1024、2048 和 4096 字节。

```shell
# mke2fs -m 0 -b 4096 /dev/sda1 
mke2fs 1.35 (28-Feb-2004) 
Filesystem label= 
OS type: Linux 
Block size=4096 (log=2) 
Fragment size=4096 (log=2) 
205344 inodes, 70069497 blocks 
0 blocks (0.00%) reserved for the super user 
First data block=0 
Maximum filesystem blocks=71303168 
2139 block groups 
32768 blocks per group, 32768 fragments per group 
96 inodes per group 
Superblock backups stored on blocks: 
32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 4096000, 7962624, 11239424, 20480000, 23887872 

Writing inode tables: done 
Writing superblocks and filesystem accounting 
information: done 

This filesystem will be automatically checked every 32 mounts or 180 days, whichever comes first. Use tune2fs -c or -i to override.
```

上面的命令将创建一个 ext2 文件系统。要创建 ext3 文件系统，请执行以下操作：

```shell
# mkfs.ext3 /dev/sda1
# mke2fs –j /dev/sda1
```

### 9.3 挂载分区

创建分区并格式化后，可以将其挂载到挂载点。

首先创建一个将挂载该分区的目录。

```shell
# mkdir /home/database
```

挂载文件系统。

```shell
# mount /dev/sda1 /home/database
```

要在重启后自动挂载文件系统，需要将以下内容添加到 /etc/fstab

```shell
/dev/sda1 /home/database ext3 defaults 0 2
```

### 9.4 使用 tune2fs 微调分区

使用 tune2fs –l /dev/sda1 查看文件系统信息，如下所示。

```shell
# tune2fs -l /dev/sda1
......
```

还可以使用 tune2fs 来调整 ex2/ext3 文件系统参数。例如，如果要更改 Filesystem 卷名，可以如下所示进行。

```shell
# tune2fs -l /dev/sda1 | grep volume 
Filesystem volume name: /home/database

# tune2fs -L database-home /dev/emcpowera1 
tune2fs 1.35 (28-Feb-2004)

# tune2fs -l /dev/sda1 | grep volume
Filesystem volume name: database-home
```

### 9.5 创建交换分区

创建一个用于交换使用的文件，如下所示。

```shell
# dd if=/dev/zero of=/home/swap-fs bs=1M count=512 
512+0 records in 
512+0 records out 

# ls -l /home/swap-fs 
-rw-r--r-- 1 root root 536870912 Jan 2 23:13 
/home/swap-fs
```

使用 mkswap 在创建的 /home/swap-fs 文件中设置 Linux 交换区。

```shell
# mkswap /home/swap-fs 
Setting up swapspace version 1, size = 536866 kB
```

一旦创建了文件并为 Linux 交换区域设置了文件，就可以使用 swapon 启用交换，如下所示。

```shell
# swapon /home/swap-fs
```

将以下行添加到 /etc/fstab 并重新启动系统以使交换生效。

```shell
/home/swap-fs swap swap defaults 0 0
```

### 9.6 创建新用户

**添加新用户 - 基本方法**

仅指定用户名

```shell
useradd jsmith
```

**添加具有附加参数的新用户**

指定以下参数

-c  用户描述

-e 用户的到期日期，格式为 mm/dd/yy

```shell
# adduser -c "John Smith - Oracle Developer" -e 12/31/21 jsmith
```

**验证用户是否已成功添加**

```shell
# grep jsmith /etc/passwd 
jsmith:x:510:510:John Smith - Oracle 
Developer:/home/jsmith:/bin/bash
```

**更改用户密码**

```shell
# passwd jsmith
```

**如何识别 useradd 使用的默认值？**

以下是创建用户时将使用的默认值。

```shell
# useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/sh
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
```

### 9.7 创建新组并分配给用户

**创建一个developers组**

```shell
# groupadd developers
```

**验证组是否创建成功**

```shell
# grep developer /etc/group 
developers:x:511:
```

**将用户添加到一个现有组**

不能使用 useradd 修改现有用户，因为将收到以下错误消息。使用usermod 修改用户的主组

```shell
# useradd -G developers jsmith 
useradd: user jsmith exists 

# usermod -g developers jsmith
```

**验证用户组是否已成功修改**

```shell
# grep jsmith /etc/passwd 
jsmith:x:510:511:Oracle Developer:/home/jsmith:/bin/bash

# id jsmith
uid=510(jsmith) gid=511(developers) groups=511(developers) 

# grep jsmith /etc/group 
jsmith:x:510: 
developers:x:511:jsmith
```

### 9.8 在OpenSSH 中设置 SSH 无密码登录

如本例所述，使用 ssky-keygen 和 ssh-copy-id 通过 3 个简单的步骤登录到远程 Linux 服务器而无需输入密码。 

ssh-keygen 创建公钥和私钥。 ssh-copy-id 将本地主机的公钥复制到远程主机的 authorized_keys 文件中。 ssh-copy-id 还为远程主机的 home、~/.ssh 和 ~/.ssh/authorized_key 分配了适当的权限。

**第 1 步：在本地主机上使用 ssh-key-gen 创建公钥和私钥**

```shell
saw@centos7 [08:56:26]> ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/saw/.ssh/id_rsa):
Created directory '/home/saw/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/saw/.ssh/id_rsa.
Your public key has been saved in /home/saw/.ssh/id_rsa.pub.
The key fingerprint is:
```

**第 2 步：使用 ssh-copy-id 将公钥复制到远程主机**

```shell
saw@centos7 [08:56:34]> ssh-copy-id -i ~/.ssh/id_rsa.pub  remote-host
```

**第 3 步：免密登录远程主机**

```shell
saw@centos7 [08:56:45]> ssh remote-host
```

### 9.9 将 ssh-copy-id 与 ssh-agent 一起使用

**将 ssh-copy-id 与 ssh-add/ssh-agent 一起使用**

如果没有为选项 -i 传递任何值并且如果 ~/.ssh/identity.pub 不可用，则 ssh-copy-id 将显示以下错误消息。

```shell
jsmith@local-host$ ssh-copy-id -i remote-host 
/usr/bin/ssh-copy-id: ERROR: No identities found
```

如果已使用 ssh-add 将密钥加载到 ssh-agent，则 ssh-copy-id 将从 ssh-agent 获取密钥以复制到远程主机。即当不将选项 -i 传递给 ssh-copy-id 时，它会将 ssh-add -L 命令提供的密钥复制到远程主机。

```shell
jsmith@local-host$ ssh-agent $SHELL 

jsmith@local-host$ ssh-add -L 
The agent has no identities. 

jsmith@local-host$ ssh-add 
Identity added: /home/jsmith/.ssh/id_rsa 
(/home/jsmith/.ssh/id_rsa) 

jsmith@local-host$ ssh-add -L 
ssh-rsa 
AAAAB3NzaC1yc2EAAAABIwAAAQEAsJIEILxftj8aSxMa3d8t6JvM79D 
aHrtPhTYpq7kIEMUNzApnyxsHpH1tQ/Ow== 
/home/jsmith/.ssh/id_rsa 

jsmith@local-host$ ssh-copy-id -i remote-host 
jsmith@remote-host’s password: 
Now try logging into the machine, with "ssh 'remote-host'", and check in: .ssh/authorized_keys to make sure we haven’t added extra keys that you weren’t expecting. 
[Note: This has added the key displayed by ssh-add -L]
### 提示，这里使用 ssh-add -L 查看已添加的key
```

以下是 ssh-copy-id 的三个小问题。

1. 默认公钥：ssh-copy-id 使用 ~/.ssh/identity.pub 作为默认公钥文件（即当没有值传递给选项 -i 时）。相反，我希望它使用 id_dsa.pub、id_rsa.pub 或 identity.pub 作为默认键。即如果其中任何一个存在，它应该将其复制到远程主机。如果存在两个或三个，则应默认复制 identity.pub。
2. 代理没有身份：当 ssh-agent 正在运行并且 ssh-add -L 返回“The agent has no identities”（即没有向 ssh-agent 添加密钥）时，ssh-copy-id 仍将复制消息“The agent has no identities”发送到远程主机的 authorized_keys 条目。
3. 重复添加条目：我希望 ssh-copy-id 验证远程主机的授权密钥上的重复条目。如果您在本地主机上多次执行 ssh-copy-id，它将继续在远程主机的 authorized_keys 文件上附加相同的密钥，而不检查重复项。

### 9.10 Crontab

使用 cron，可以在特定时间和日期执行 shell 脚本或 Linux 命令。例如，系统管理员可以安排每天运行的备份作业。

**如何向 cron 添加作业？**

```shell
# crontab –e 
0 5 * * * /root/bin/backup.sh
```

这将在每天早上 5 点执行 /root/bin/backup.sh。

**Cron 字段的描述。**

以下是 crontab 文件的格式。

{minute}   {hour}    {day-of-month}   {month}   {day-of-week}  {full-path-to-shell-script} 

- minute: 允许范围 0 – 59
- hour:  允许范围 0 – 23 
- day-of-month: 允许范围 0 – 31 
- month：允许范围 1-12
- Day-of-week：允许范围0-7，星期日是 0 或 7

**crontab 示例**

1. 每天午夜后 1 分钟在凌晨 12:01 运行。当系统没有负载时，这是运行备份的好时机。
   
   ```shell
   1 0 * * * /root/bin/backup.sh
   ```

2. 每个工作日（周一至周五）晚上 11:59 运行备份。
   
   ```shell
   59 11 * * 1,2,3,4,5 /root/bin/backup.sh
   ```
   
   下面这个也一样效果
   
   ```shell
   59 11 * * 1-5 /root/bin/backup.sh
   ```

3. 每 5 分钟执行一次命令。
   
   ```shell
   */5 * * * * /root/bin/check-status.sh
   ```

4. 每月 1 日下午 1:10 执行
   
   ```shell
   10 13 1 * * /root/bin/full-backup.sh
   ```

5. 在工作日晚上 11 点执行。
   
   ```shell
   0 23 * * 1-5 /root/bin/incremental-backup.sh
   ```

**crontab 选项**

以下是 crontab 的可用选项：

- crontab –e ：编辑 crontab 文件。如果它不存在，将创建一个 crontab。
- crontab –l ：显示 crontab 文件。
- crontab -r ：删除 crontab 文件。
- crontab -ir ：这将在删除 crontab 之前提示用户。

### 9.11 使用神奇的 SysRq 组合键安全重启 Linux

神奇的 SysRq 键是 Linux 内核中的一个组合键，它允许用户执行各种低级命令，而不管系统的状态如何。

它通常用于从冻结中恢复，或在不破坏文件系统的情况下重新启动计算机。组合键包括   `Alt + SysRq + commandkey`  。在多数系统中，SysRq 是打印屏幕键。

**首先需要启用 SysRq 键**，如下

```shell
echo "1" > /proc/sys/kernel/sysrq
```

**SysRq 命令键列表**

以下是 Alt+SysRq+commandkey 可用的命令键。

- 'k' - 杀死当前虚拟控制台上运行的所有进程
- 's' - 这将尝试同步所有已挂载的文件系统
- 'b' - 立即重新启动系统，无需卸载分区或同步数据
- 'e' - 向除 init 之外的所有进程发送 SIGT**E**RM。
- 'm' - 将当前内存信息输出到控制台。
- 'i' - 向除 init 之外的所有进程发送 SIGK**I**LL 信号
- 'r' -  将键盘从原始模式（X11 等程序使用的模式）切换到 XLATE 模式。
- 's' - 同步所有挂载的文件系统
- 't' - 将当前任务列表及其信息输出到控制台
- 'u' - 以只读模式重新挂载所有已挂载的文件系统
- 'o' - 立即关闭系统
- 'p' - 将当前寄存器和标志打印到控制台。
- '0-9' - 设置控制台日志级别，控制哪些内核消息将打印到您的控制台
- 'f' - 将调用 oom_kill 来杀死占用更多内存的进程
- 'h' - 用于显示帮助。但是上面列出的任何其他键都将打印帮助

也可以通过将keys 输入到 /proc/sysrq-trigger 文件达到相同的效果。例如，要重新引导系统，可以执行以下操作。

```shell
echo "b" > /proc/sysrq-trigger
```

**使用 Magic SysRq Key 安全重启 Linux**

要对挂起的 Linux 计算机执行安全重启，请执行以下操作。这将在下次重新引导期间避免 fsck。即按 Alt+SysRq+下面突出显示的字母。

- un**R**aw (take control of keyboard back from X11),
- t**E**rminate (send SIGTERM to all processes, allowing them to terminate gracefully), 
- k**I**ll (send SIGILL to all processes, forcing them to terminate immediately), 
- **S**ync (flush data to disk), 
- **U**nmount (remount all filesystems read-only), 
- re**B**oot.

## 十：Apachectl And Httpd Examples（Apache和Httpd示例）

## 十一：Bash Scripting（bash脚本）

### 11.1 .bash_* 文件的执行顺序

以下文件的执行顺序是什么？

- /etc/profile
- ~/.bash_profile
- ~/.bash_login
- ~/.profile
- ~/.bash_logout
- ~/.bashrc

<font color=blue>**登录交互式 shell 时的执行顺序**</font>

以下伪代码解释了这些文件的执行顺序。

```
execute /etc/profile 
IF ~/.bash_profile exists THEN 
   execute ~/.bash_profile 
ELSE 
   IF ~/.bash_login exist THEN 
       execute ~/.bash_login 
   ELSE 
       IF ~/.profile exist THEN 
           execute ~/.profile 
       END IF 
   END IF 
END IF
```

<font color=blue>**注销交互式 shell 时的执行顺序**</font>

```
IF ~/.bash_logout exists THEN 
    execute ~/.bash_logout 
END IF
```

请注意 /etc/bashrc 由 ~/.bashrc 执行，如下所示：

```
# cat ~/.bashrc 
if [ -f /etc/bashrc ]; then 
    . /etc/bashrc 
Fi
```

<font color=brown>**非交互式登录 shell 的执行顺序**</font>

启动非登录交互式 shell 时，执行顺序如下：

```
IF ~/.bashrc exists THEN 
   execute ~/.bashrc 
END IF
```

注意：当非交互式 shell 启动时，它会查找 ENV 环境变量，并执行 ENV 变量中提到的文件名值。

### 11.2 如何在 bash shell 中生成随机数

使用 $RANDOM bash 内置函数生成 0 - 32767 之间的随机数，如下所示

```shell
$ echo $RANDOM
22543

$ echo $RANDOM
25387

$ echo $RANDOM
647
```

### 11.3 调试 shell 脚本

要调试 shell 脚本，请在 shell 脚本的顶部使用 set –xv

**没有调试命令的 Shell 脚本**：

```shell
$ cat filesize.sh 
#!/bin/bash 
for filesize in $(ls -l . | grep "^-" | awk '{print $5}') 
do 
 let totalsize=$totalsize+$filesize 
done 
echo "Total file size in current directory: $totalsize"
```

**没有调试命令的 Shell 脚本的输出**：

```shell
$ ./filesize.sh 
Total file size in current directory: 652
```

**内部带有 Debug 命令的 Shell 脚本**

现在在 shell 脚本中添加 set –xv 以调试输出，如下所示

```shell
$ cat filesize.sh 
#!/bin/bash 
set -xv 
for filesize in $(ls -l . | grep "^-" | awk '{print $5}') 
do 
 let totalsize=$totalsize+$filesize 
done 
echo "Total file size in current directory: $totalsize"
```

**带有 Debug 命令的 Shell 脚本的输出**：

```shell
$ ./fs.sh 
++ ls -l . 
++ grep '^-' 
++ awk '{print $5}' 
+ for filesize in '$(ls -l . | grep "^-" | awk 
'\''{print $5}'\'')' 
+ let totalsize=+178 
+ for filesize in '$(ls -l . | grep "^-" | awk 
'\''{print $5}'\'')' 
+ let totalsize=178+285 
+ for filesize in '$(ls -l . | grep "^-" | awk 
'\''{print $5}'\'')' 
+ let totalsize=463+189 
+ echo 'Total file size in current directory: 652' 
Total file size in current directory: 652
```

**使用调试选项执行 Shell 脚本：**

除了在 shell 脚本中提供 set –xv 之外，还可以在执行 shell 脚本时提供它，如下所示。

```shell
$ bash -xv filesize.sh
```

### 11.4 引用

没有任何特殊字符的 echo 语句

```shell
$ echo The Geek Stuff
The Geek Stuff
```

<font color=red>**带有特殊字符;的echo语句**</font>。分号是 bash 中的命令终止符。在以下示例中，“The Geek”适用于 echo，“Stuff”被视为单独的 Linux 命令并给出未找到的命令。

```shell
$ echo The Geek; Stuff
The Geek
-bash: Stuff: command not found
```

为避免这种情况，可以在分号前添加一个逃脱字符 \，这将删除分号的特殊含义，只需将其打印如下所示。

```shell
$ echo The Geek\; Stuff
The Geek; Stuff
```

<font color=red>**单引号**</font>

当想从字面上打印所有内容时，请使用单引号。甚至像 $HOSTNAME 这样的特殊变量也会打印为 $HOSTNAME 而不是打印 Linux 主机的名称。

```shell
$ echo 'Hostname=$HOSTNAME ; Current User=`whoami` ; Message=\$ is USD' 
Hostname=$HOSTNAME ; Current User=`whoami` ; Message=\$ is USD
```

<font color=red>**双引号**</font>

当想显示特殊变量的真正含义时，请使用双引号。

```shell
$ echo "Hostname=$HOSTNAME ; Current User=`whoami` ; Message=\$ is USD"
Hostname=dev-db ; Current User=ramesh ; Message=$ is USD
```

双引号将删除除以下字符之外的所有字符的特殊含义：

- $ 参数替换
- ` 反引号
- \\$   文字美元符号
- \\`   文字反引号 
- \\"    嵌入式双引号
- \\\    嵌入式反斜线

### 11.5 在shell 脚本中读取数据文件

下面示例显示如何从数据文件中读取特定字段并在 shell 脚本中对其进行操作。例如，假设employee.txt 文件的格式为 {employee-name}:{employee id}:{department-name}，文件以冒号分隔，如下所示。

```shell
$ cat employees.txt 
Emma Thomas:100:Marketing 
Alex Jason:200:Sales 
Madison Randy:300:Product Development 
Sanjay Gupta:400:Support 
Nisha Singh:500:Sales
```

下面的 shell 脚本解释了如何从这个employee.txt 文件中读取特定的字段。

```shell
$ vi read-employees.sh 
#!/bin/bash 
IFS=: 
echo "Employee Names:" 
echo "---------------" 
while read name empid dept
do 
  echo "$name is part of $dept department" 
done < ~/employees.txt
```

为 shell 脚本分配执行权限并执行它。

```shell
$ chmod u+x read-employees.sh 
$ ./read-employees.sh 
Employee Names:
---------------
Emma Thomas is part of Marketing  department
Alex Jason is part of Sales  department
Madison Randy is part of Product Development  department
Sanjay Gupta is part of Support  department
Nisha Singh is part of Sales department
```

## 十二：System Monitoring and Performance(系统监控和性能)

### 12.1 free 命令

free 命令显示有关系统物理 (RAM) 和交换内存的所有必要信息

系统上的总 RAM 是多少？在下面的示例中，此系统上的总物理内存为8GB。下面显示的值以 KB 为单位。

```shell
# free
              total        used        free      shared  buff/cache   available
Mem:        8173832     2570964     4654780       74328      948088     5258668
Swap:             0           0           0
```

系统上的总内存是多少，包括 RAM 和交换？

- 选项 m 以 MB 为单位显示值

- 选项 t 显示“Total”行，它是物理和交换内存值的总和

- 选项 o 是从上面的示例中隐藏缓冲区/缓存行
  
  ```shell
  # free -mt
                total        used        free      shared  buff/cache   available
  Mem:           7982        2510        4545          72         925        5135
  Swap:             0           0           0
  Total:         7982        2510        4545
  ```

### 12.2 top 命令

top 命令显示有关系统各种性能指标的实时信息，例如 CPU 负载、内存使用情况、进程列表等

**如何查看当前的系统状态，包括 CPU 使用率？**

命令行不带任何选项执行 top，将显示如下所示的输出。top命令输出将持续显示实时值，直到您按“Control + c”或 q 退出命令输出

```shell
# top
top - 08:30:40 up 39 days, 21:43,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  95 total,   1 running,  94 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.5 sy,  0.0 ni, 99.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  8173832 total,  4558632 free,  2596540 used,  1018660 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  5223672 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 3218 root      20   0 8514364   1.9g  17052 S   1.3 24.8 652:18.30 java
 1253 mysql     20   0 1587180 243424  13680 S   0.3  3.0  47:59.41 mysqld
```

**如何阅读上面显示的 top 命令的输出？**

- 第 1 行“top”，表示系统已经启动并运行了 39 天。

- 第 2 行“tasks”，显示进程总数以及运行、睡眠、停止和僵尸进程计数的细分。

- 第 3 行“%CPU”，显示了当前的系统CPU使用率，这里CPU的idle空闲率是99.2%

- 第 4行“Mem”和第 5行“Swap"，提供内存信息.这与 free 命令中的信息相同。

- 其余行显示系统上的所有活动进程，

有几个命令行选项和交互选项可用于top命令。让我们回顾一下 top 命令的几个基本选项

**如何识别内存最密集的进程？**

在显示top命令的输出时，按 f，这将显示以下消息并显示所有可用于排序的字段，使用 Up/Dn 导航，右选择移动，然后或左提交，'d' 或 空格键切换显示，'s' 设置默认排序。使用 'q' 或 ESC 退出。

**如何在top输出中添加其他字段（例如 CPU 时间）？**

当 top 命令运行时，按 f，这将显示以下消息并显示所有可显示的字段，按 l，这会将 CPU 时间添加到顶部输出的显示列中。

**如何获取正在运行的进程的完整路径名和参数？**

在 top 命令运行时，按 c，这将在命令列中显示正在运行的进程的完整路径名，如下所示。即它显示 /usr/local/apache2/bin/httpd 而不是 httpd。

### 12.3 ps 命令

ps 命令（进程状态）将显示所有活动进程的快照信息。

**如何显示系统中运行的所有进程？**

使用“ps aux”，如下图所示。

```shell
# ps aux | more
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND 
root 1 0.0 0.0 2044 588 ? Ss Jun27 0:00 init [5] 
apache 31186 0.0 1.6 23736 17556 ? S Jul26 0:40 /usr/local/apache2/bin/httpd 
apache 31187 0.0 1.3 20640 14444 ? S Jul26 0:37 /usr/local/apache2/bin/httpd 
```

也可以使用“ps -ef | more”来获得类似的输出

**打印进程树**

可以使用 ps axuf 或 ps –ejH 以树格式显示进程。树形结构将有助于立即可视化进程及其父进程。为清楚起见，以下输出中的几列已被截断。

```shell
# ps axuf
root Oct14 0:00 /opt/VRTSralus/bin/beremote 
root Oct14 0:00 \_ /opt/VRTSralus/bin/beremote 
root Oct14 0:00    \_ /opt/VRTSralus/bin/beremote 
root Oct14 0:00    \_ /opt/VRTSralus/bin/beremote 
root Oct14 0:01    \_ /opt/VRTSralus/bin/beremote 
root Oct 14 0:00   \_ /opt/VRTSralus/bin/beremote
```

也可以使用 pstree 命令以树形结构显示进程。

**查看特定用户拥有的进程**

以下命令显示 Linux 用户名oracle拥有的所有进程。

```shell
$ ps U oracle
PID TTY STAT TIME COMMAND 
5014 ? Ss 0:01 /oracle/bin/tnslsnr 
 7124 ? Ss 0:00 ora_q002_med 
8206 ? Ss 0:00 ora_cjq0_med 
8852 ? Ss 0:01 ora_pmon_med 
8854 ? Ss 0:00 ora_psp0_med 
8911 ? Ss 0:02 oraclemed (LOCAL=NO)
```

**查看当前用户拥有的进程**

```shell
 ps U $USER
    PID TTY      STAT   TIME COMMAND
1884572 ?        Ss     0:01 /lib/systemd/systemd --user
1884580 ?        S      0:00 (sd-pam)
1884738 ?        R      0:06 sshd: saw@pts/0
1884739 pts/0    Ss     0:00 -bash
1903892 pts/0    R+     0:00 ps U saw
```

### 12.4 df 命令

df 命令（可用磁盘）显示已安装文件系统上可用的总磁盘空间和可用磁盘空间。

**系统上有多少 GB 的可用磁盘空间？**

使用 df -h 如下所示。选项 -h 以人类可读的格式显示值（例如：K 表示 Kb，M 表示 Mb，G 表示 Gb）。在下面的示例输出中，/文件系统有 17GB 的可用磁盘空间，/home/user 文件系统有 70GB 的可用空间。

```shell
# df –h
Filesystem Size Used Avail Use% Mounted on 
/dev/sda1 64G 44G 17G 73% / 
/dev/sdb1 137G 67G 70G 49% /home/user
```

**系统上有什么类型的文件系统？**

选项 -T 将显示有关文件系统类型的信息。在这个例子中 / 和 /home/user 文件系统是 ext2。选项 -a 将显示所有文件系统，包括系统使用的 0 大小的特殊文件系统。

```shell
# df -Tha
Filesystem Type Size Used Avail Use% Mounted on
/dev/sda1 ext2 64G 44G 17G 73% / 
/dev/sdb1 ext2 137G 67G 70G 49% /home/user 
none proc 0 0 0 - /proc 
none sysfs 0 0 0 - /sys 
none devpts 0 0 0 - /dev/pts 
none tmpfs 2.0G 0 2.0G 0% /dev/shm
```

### 12.5 kill 命令

kill 命令可用于终止正在运行的进程。通常，此命令用于终止挂起且未响应的进程。

**如何杀死一个挂起的进程？**

首先，使用 ps 命令确定要终止的特定进程的进程 ID。知道进程 ID 后，将其作为参数传递给 kill 命令。下面的例子展示了如何杀死挂起的 apache httpd 进程。请注意，通常您应该使用“apachectl stop”来停止 apache。

```shell
# ps aux | grep httpd 
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND 
apache 31186 0.0 1.6 23736 17556 ? S Jul26 0:40 /usr/local/apache2/bin/httpd 
apache 31187 0.0 1.3 20640 14444 ? S Jul26 0:37 /usr/local/apache2/bin/httpd

# kill 31186 31187
```

请注意，上面的命令试图通过发送一个名为 SIGTERM 的信号来优雅地终止进程。如果进程没有终止，可以通过传递一个名为 SIGKILL 的信号强制终止进程，使用选项 -9 如下所示。这应该是进程的所有者或特权用户来终止进程。

```shell
# kill -9 31186 31187
```

**另一种**轻松杀死多个进程的方法是将以下两个函数添加到 .bash_profile。

```shell
function psgrep () 
{ 
 ps aux | grep "$1" | grep -v 'grep' 
} 

function psterm () 
{ 
 [ ${#} -eq 0 ] && echo "usage: $FUNCNAME STRING" && return 0 
 local pid 
 pid=$(ps ax | grep "$1" | grep -v grep | awk '{ print $1 }') 
 echo -e "terminating '$1' / process(es):\n$pid" 
 kill -SIGTERM $pid 
}
```

现在执行以下操作，以识别并终止所有 httpd 进程。

```shell
# psgrep http
USER PID %CPU %MEM VSZ RSS TTY STAT START TIME 
COMMAND 
apache 31186 0.0 1.6 23736 17556 ? S Jul26 0:40 /usr/local/apache2/bin/httpd 
apache 31187 0.0 1.3 20640 14444 ? S Jul26 0:37 /usr/local/apache2/bin/httpd

# psterm httpd
terminating 'httpd' / process(es): 
31186
31187
```

### 12.6 du 命令

### 12.7 lsof命令

lsof 名称代表 ls open files，会列出系统中所有打开的文件，包括网络连接、设备和目录。 lsof 命令的输出将包含以下列：

- COMMAND 进程名称
- PID 进程号
- USER 用户名
- FD 文件描述符
- TYPE 文件的节点类型
- DEVICE 设备号
- SIZE 文件大小
- NODE节点号
- NAME 文件名的完整路径

**查看系统所有打开的文件**

执行不带任何参数的 lsof 命令，如下所示。

```shell
# lsof | more
COMMAND       PID     TID TASKCMD       USER   FD      TYPE             DEVICE     SIZE/OFF       NODE NAME
systemd         1                       root  cwd       DIR                8,3         4096          2 /
systemd         1                       root  rtd       DIR                8,3         4096          2 /
systemd         1                       root  txt       REG                8,3      1620224   48633603 /usr/lib/systemd/systemd
systemd         1                       root  mem       REG                8,3      1369352   48634793 /usr/lib/x86_64-linux-gnu/libm-2.31.so
```

lsof 命令本身可能会返回大量记录作为输出，这可能没有多大意义，除了让您大致了解在任何给定角度下系统中打开了多少文件，如下所示。

```shell
lsof | wc -l
82413
```

**查看特定用户打开的文件**

使用 lsof –u 选项显示特定用户打开的所有文件

```shell
# lsof -u saw
COMMAND      PID USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
systemd  3388847  saw  cwd       DIR                8,3     4096          2 /
systemd  3388847  saw  rtd       DIR                8,3     4096          2 /
systemd  3388847  saw  txt       REG                8,3  1620224   48633603 /usr/lib/systemd/systemd
systemd  3388847  saw  mem       REG                8,3  1369352   48634793 /usr/lib/x86_64-linux-gnu/libm-2.31.so
```

<font color=violetred>系统管理员可以使用此命令了解用户在系统上执行的操作</font>。

### 12.8 sar 命令

Sar 命令随 sysstat 软件包一起提供。确保已安装 sysstat。如果您的系统上没有安装 sar，请从 [Sysstat](http://sebastien.godard.pagesperso-orange.fr/) 项目中获取。

Sar 是一个出色的监控工具，可以显示系统几乎所有资源的性能数据，包括 CPU、内存、IO、分页、网络、中断等，

Sar 收集、报告（显示）和保存性能数据。接下来分别看一下这三个方面

**Sadc - System activity data collector** 

/usr/lib/sadc（系统活动数据收集器）命令以指定的时间间隔收集系统数据。这使用位于 /va/log/sa/sa[dd] 下的每日活动数据文件，其中 dd 是当天。

**Sa1 shell-script**

/usr/lib/sa1 又调用 /usr/lib/sadcs。 sa1 从 crontab 调用，如下所示。根据需要，每 5 分钟或 15 分钟运行一次。我更喜欢在 cron 选项卡中每 5 分钟安排一次，如下所示。

```shell
*/5 * * * * root /usr/lib/sa/sa1 1 1
```

**Sa2 shell-script** 

/usr/lib/sa2 是一个 shell 脚本，它将在 /var/log/sa/sa[dd] 文件中写入每日报告，其中 dd 是当天。每天午夜从 crontab 调用一次 sa2。

```shell
59 23 * * * root /usr/lib/sa/sa2 -A
```

注意：/etc/cron.d/sysstat 文件随 sysstat 包提供，其中包含 sa1 和 sa2 的一些默认值，可以相应地更改它们。

**使用 Sar 命令显示 CPU 统计信息**

### 12.9 vmstat 命令

对于典型的性能监控，只需要 vmstat 命令。将显示内存、swap、IO、系统和 cpu 性能信息。

以下命令每 1 秒执行 vmstat 100 次。

```shell
vmstat 1 100
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 4655400 168016 780196    0    0     0     2    6    4  1  0 99  0  0
 0  0      0 4654776 168016 780228    0    0     0     0  370  609  1  1 98  0  0
 0  0      0 4654920 168016 780228    0    0     0     0  271  526  0  1 99  0  0
 0  0      0 4654320 168016 780228    0    0     0    32  358  611  1  1 98  1  0
```

**procs 部分**

- r 字段：可运行进程的总数
- b 字段：阻塞进程的总数

**Memory 部分**

- Swpd 字段：已用交换空间

- 空闲字段：可用的空闲 RAM

- Buff 字段：用于缓冲区的 RAM

- 缓存字段：用于文件系统缓存的 RAM

**Swap 部分**

- Si 字段：每秒从磁盘交换到内存的总量
- So 字段：每秒从内存交换到磁盘的总量

**IO 部分**

- Bi 字段：从磁盘接收的块
- Bo 字段：发送到磁盘的块

**System 部分**

- in 字段：每秒的中断数
- cs 字段：每秒上下文切换的次数
- us 字段：运行用户代码的时间（非内核代码）
- sy 字段：运行内核代码所花费的时间
- Id 字段：空闲时间
- wa 字段: 等待 IO 所花费的时间

### 12.10 netstat 命令

Netstat 命令显示网络相关信息，如网络连接、路由表、接口统计信息。以下是有关如何使用 netstat 命令的几个示例。

**使用 netstat 显示活动的 Internet 连接和域套接字**

```shell
# netstat -an
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp6       0      0 192.168.229.157:55784   192.168.229.180:3306    ESTABLISHED
udp        0      0 127.0.0.1:323           0.0.0.0:*
udp6       0      0 ::1:323                 :::*
raw   213504      0 0.0.0.0:112             0.0.0.0:*               7
raw        0      0 0.0.0.0:112             0.0.0.0:*               7
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     SEQPACKET  LISTENING     10768    /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     33925    /tmp/tmux-0/default
unix  2      [ ]         DGRAM                    10788    /run/systemd/shutdownd
unix  2      [ ACC ]     STREAM     LISTENING     17911    /data/mysql/mysql.sock
```

**显示带有进程 ID 和程序名称的活动连接**

这对于识别哪个程序启动了特定的网络连接非常有帮助。

```shell
# netstat -tap
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      915/sshd
tcp        0      0 0.0.0.0:radan-http      0.0.0.0:*               LISTEN      3218/java
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN      1394/master
tcp        0      0 localhost:32000         0.0.0.0:*               LISTEN      3218/java
```

**显示路由表**

```shell
# netstat --route
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         gateway         0.0.0.0         UG        0 0          0 eth1
172.168.229.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
192.168.229.0   0.0.0.0         255.255.255.0   U         0 0          0 eth1
```

**显示 RAW 网络统计信息**

```shell
# netstat --raw --statistics
Ip:
    32067048 total packets received
    0 forwarded
    1759 with unknown protocol
    0 incoming packets discarded
    28240156 incoming packets delivered
    25296625 requests sent out
    8 outgoing packets dropped
Icmp:
    59848 ICMP messages received
    0 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 59642
        echo requests: 206
    2012 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 1804
        echo request: 2
        echo replies: 206
```

**杂项 Netstat 命令**

```shell
# netstat –-tcp –-numeric   
获取与机器之间的 TCP 连接列表。

# netstat --tcp --listening --programs
显示服务器正在侦听的 TCP 端口以及正在侦听该特定端口的程序

# netstat –rnC 
显示路由缓存
```

### 12.11 sysctl 命令

### 12.12 nice 命令

内核根据 nice 值决定一个进程需要多少处理器时间。可能的 nice 值范围是：-20 到 20。nice 值为 -20 的进程具有很高的优先级。 nice 值为 20 的进程的优先级非常低。

使用 ps axl 显示所有正在运行的进程的 nice 值，如下所示。

```shell
$  ps axl
F   UID     PID    PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
4     0       1       0  20   0 169352 11620 -      Ss   ?          8:43 /sbin/init
1     0       2       0  20   0      0     0 -      S    ?          0:02 [kthreadd]
1     0       3       2   0 -20      0     0 -      I<   ?          0:00 [rcu_gp]
1     0       4       2   0 -20      0     0 -      I<   ?          0:00 [rcu_par_gp]
1     0       6       2   0 -20      0     0 -      I<   ?          0:00 [kworker/0:0H-kblockd]
```

**如何为 shell 脚本分配低优先级？ （较高的nice值）**

在下面的示例中，当后台启动 nice-test.sh 脚本时，它的 nice 值为 0。

```shell
$ ./nice-test.sh & 
[3] 13009 

$ ps axl | grep nice-test 
0 509 13009 12863 17 0 4652 972 wait S pts/1 0:00 /bin/bash ./nice-test.sh
#第六列是nice值
```

现在，使用不同的 nice 值执行相同的 shell 脚本，如下所示。

```shell
$ nice -10 ./nice-test.sh & 
[1] 13016 

$ ps axl | grep nice-test 
0 509 13016 12863 30 10 4236 968 wait SN pts/1 0:00 /bin/bash ./nice-test.sh 
#第六列是nice值
```

**如何为 shell 脚本分配高优先级？ （较低的nice值）**

在下面的示例中，为 nice-test.sh脚本分配一个 nice 值 -10（减 10）。

```shell
$ nice --10 ./nice-test.sh & 
[1] 13021 
$ nice：cannot set priority: Permission denied
```

<font color=red>注意：只有 root 用户可以设置负的 nice 值。以 root 身份登录并尝试相同的操作。请注意，在下面的 nice 命令中，10 之前有一个双破折号</font>。

```shell
# nice --10 ./nice-test.sh &
[1] 13060 

# ps axl | grep nice-test 
4 0 13060 13024 10 -10 5388 964 wait S< pts/1 0:00 /bin/bash ./nice-test.sh
#第六列是nice值
```

### 12.13 renice 命令

[shell脚本利用expect实现非交互](https://blog.51cto.com/jiangxl/4638242)

- ssh登录
- 推送主机公钥
- 批量修改主机基本配置
