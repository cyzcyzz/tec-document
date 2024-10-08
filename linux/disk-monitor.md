---
title: "磁盘性能监控"
date: 2021-12-30T09:30:46+08:00
description: ""
draft: true
tags: []
categories: []
---

## 一. 磁盘性能衡量指标

- IOPS：Input/Output Operations per Second，即每秒能处理的I/O个数，用于表示块存储处理读写（输出/输入）的能力。
- 吞吐量：吞吐量是指单位时间内可以成功传输的数据数量。
- 阿里云块存储性能：

| 参数     | ES SD云盘   | SSD云盘     | 高效云盘      | 普通云盘       | SSD共享块存储  | 高效共享块存储   |
| ------ | --------- | --------- | --------- | ---------- | --------- | --------- |
| 单盘最大容量 | 32768 GiB | 32768 GiB | 32768 GiB | 2000 GiB   | 32768 GiB | 32768 GiB |
| 最大IOPS | 1000000   | 25000*    | 5000      | 数百         | 30000     | 5000      |
| 最大吞吐量  | 4000 MBps | 300 MBps* | 140 MBps  | 30−40 MBps | 512 MBps  | 160 MBps  |

## 二. 性能测试：

- 可用dd测试块存储性能

```
例：
[root@jenkins tmp]# dd if=/dev/zero of=/tmp/testfile bs=1M count=2048

2048+0 records in
2048+0 records out
2147483648 bytes (2.1 GB) copied, 9.90132 s, 217 MB/s
```

- 测试随机写IOPS，运行以下命令：

```
fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
```

- 测试随机读IOPS，运行以下命令：

```
fio -direct=1 -iodepth=128 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Read_Testing
```

- 测试顺序写吞吐量，运行以下命令：

```
fio -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Write_PPS_Testing
```

- 测试顺序读吞吐量，运行以下命令：

```
fio -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=1024k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Read_PPS_Testing
```

各种参数的含义：

```
-direct=1       表示测试时忽略I/O缓存，数据直写。
-iodepth=128    表示使用AIO时，同时发出I/O数的上限为128。
-rw=randwrite   表示测试时的读写策略为随机写（random writes）。作其它测试时可以设置为：
                    randread（随机读random reads）
                    read（顺序读sequential reads）
                    write（顺序写sequential writes）
                    randrw（混合随机读写mixed random reads and writes）
-ioengine=libaio    表示测试方式为libaio（Linux AIO，异步I/O）。应用程序使用I/O通常有两种方式：
                        同步
                        同步的I/O一次只能发出一个I/O请求，等待内核完成才返回。这样对于单个线程iodepth总是小于1，但是可以透过多个线程并发执行来解决。通常会用16−32根线程同时工作将iodepth塞满。

                        异步
                        异步的I/O通常使用libaio这样的方式一次提交一批I/O请求，然后等待一批的完成，减少交互的次数，会更有效率。

-bs=4k          表示单次I/O的块文件大小为4 KB。未指定该参数时的默认大小也是4 KB。
                    测试IOPS时，建议将bs设置为一个比较小的值，如本示例中的4k。
                    测试吞吐量时，建议将bs设置为一个较大的值，如本示例中的1024k。
-size=1G        表示测试文件大小为1 GiB。
-numjobs=1      表示测试线程数为1。
-runtime=1000       表示测试时间为1000秒。如果未配置，则持续将前述-size指定大小的文件，以每次-bs值为分块大小写完。
-group_reporting    表示测试结果里汇总每个进程的统计信息，而非以不同job汇总展示信息。
-filename=iotest    指定测试文件的名称，比如iotest。测试裸盘可以获得真实的硬盘性能，但直接测试裸盘会破坏文件系统结构，请在测试前提前做好数据备份。
-name=Rand_Write_Testing    表示测试任务名称为Rand_Write_Testing，可以随意设定。
```

例：测试随机写IOPS

```
[root@lxk tmp]# fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_Write_Testing
Rand_Write_Testing: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=libaio, iodepth=128
fio-3.1
Starting 1 process
Rand_Write_Testing: Laying out IO file (1 file / 1024MiB)
Jobs: 1 (f=1): [w(1)][100.0%][r=0KiB/s,w=9696KiB/s][r=0,w=2424 IOPS][eta 00m:00s]
Rand_Write_Testing: (groupid=0, jobs=1): err= 0: pid=29264: Tue Oct  9 09:18:01 2018
  write: IOPS=2131, BW=8525KiB/s (8730kB/s)(1024MiB/122993msec)
    slat (usec): min=3, max=111464, avg=20.62, stdev=881.32
    clat (usec): min=495, max=635660, avg=60031.26, stdev=45220.47
     lat (usec): min=510, max=635664, avg=60052.28, stdev=45221.82
    clat percentiles (msec):
     |  1.00th=[    3],  5.00th=[    4], 10.00th=[    5], 20.00th=[    6],
     | 30.00th=[    8], 40.00th=[   34], 50.00th=[   91], 60.00th=[   94],
     | 70.00th=[   96], 80.00th=[   97], 90.00th=[  100], 95.00th=[  102],
     | 99.00th=[  115], 99.50th=[  125], 99.90th=[  209], 99.95th=[  313],
     | 99.99th=[  634]
   bw (  KiB/s): min= 4104, max=10544, per=100.00%, avg=8525.44, stdev=423.40, samples=245
   iops        : min= 1026, max= 2636, avg=2131.36, stdev=105.86, samples=245
  lat (usec)   : 500=0.01%, 750=0.03%, 1000=0.04%
  lat (msec)   : 2=0.18%, 4=5.16%, 10=28.87%, 20=3.94%, 50=2.20%
  lat (msec)   : 100=52.60%, 250=6.92%, 500=0.02%, 750=0.04%
  cpu          : usr=0.66%, sys=2.54%, ctx=30595, majf=0, minf=26
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=0.1%, 32=0.1%, >=64=100.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.1%
     issued rwt: total=0,262144,0, short=0,0,0, dropped=0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=128

Run status group 0 (all jobs):
  WRITE: bw=8525KiB/s (8730kB/s), 8525KiB/s-8525KiB/s (8730kB/s-8730kB/s), io=1024MiB (1074MB), run=122993-122993msec

Disk stats (read/write):
  vda: ios=1/262860, merge=0/13134, ticks=15/15500516, in_queue=15511938, util=99.95%
```

## 三. 系统级磁盘IO监控

### 1. top

```
top - 16:59:14 up 15:40,  2 users,  load average: 0.00, 0.00, 0.00
Tasks: 100 total,   1 running,  99 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.2%sy,  0.0%ni, 99.8%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   2037260k total,  1342560k used,   694700k free,    69060k buffers
Swap:  4095996k total,        0k used,  4095996k free,  1018516k cached
```

上面的Cpu(s)中,0.0%wa为CPU等待磁盘IO所占的时间，若该值持续过高，则表示磁盘IO性是系统的瓶颈。

### 2. iostat

- 用在磁盘IO监控中的参数

```
OPTIONS：
-d 显示磁盘利用报告 
-m 某些使用block或Kilobytes为单位的列强制使用megabytes为单位
 
-x 显示详细状态  
 CPU Utilization Report
 %iowait：Show the percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request.
 CPU等待未完成的磁盘读写请求所耗费的CPU时钟周期的百分比
 Device Utilization Report
 rrqm/s
 The number of read requests merged per second that were queued to the device.
 每秒发送给设备排队的读请求的合并数量
 wrqm/s
 The number of write requests merged per second that were queued to the device.
 每秒发送给设备排队的写请求的合并数量
 r/s
 The number of read requests that were issued to the device per second.
 每秒读请求数量
 w/s
 The number of write requests that were issued to the device per second.
 每秒写请求数量
 rsec/s
 The number of sectors read from the device per second.
 每秒读的磁盘扇区数量
 wsec/s
 The number of sectors written to the device per second.
 每秒写到磁盘的扇区数量
 avgrq-sz
 The average size (in sectors) of the requests that were issued to the device.
 请求发送给设备的平均扇区大小
 avgqu-sz
 The average queue length of the requests that were issued to the device.
 发送给设备的请求的平均队列长度
 await
 The average time (in milliseconds) for I/O requests issued to the device to be served. This includes the time spent by the requests in queue and the time spent servicing them.
 I/O请求发送给目标磁盘设备所需的平均时间(以毫秒为单位)。包括排队等待和处理请求的时间。（即IO响应时长，一般应低于5s）
 svctm
 The average service time (in milliseconds) for I/O requests that were issued to the device. Warning! Do not trust this field any more. This field will be removed in a future sysstat version.
 分发给设备的 I/O 请求的平均服务时间。（单位是毫秒）警告！不要再相信这列值了。这一列将会在一个未来的版本中移除。
 一次 IO 请求的服务时间，对于单块盘，完全随机读时，基本在 7ms 左右，即寻道 + 旋转延迟时间
 %util
 Percentage of elapsed time during which I/O requests were issued to the device (bandwidth utilization for the device). Device saturation occurs when this value is close to 100%.
 分发给设备的 I/O 请求的运行时间所占的百分比。（设备的带宽利用率）这个值接近 100%表明设备饱和。
```

 

```
例：
[root@gitlab ~]# iostat -d -x 1 1
Linux 2.6.32-696.16.1.el6.x86_64 (gitlab)     10/09/2018     _x86_64_    (2 CPU)

Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda              22.57    10.39  108.92    2.50  7615.36   102.80    69.27     0.69    6.23    6.28    4.30   0.27   3.04

[root@gitlab ~]# iostat -x 1 1
Linux 2.6.32-696.16.1.el6.x86_64 (gitlab)     10/09/2018     _x86_64_    (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.37    0.00    1.75   16.51    0.00   79.36

Device:         rrqm/s   wrqm/s     r/s     w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda              22.57    10.39  108.91    2.50  7614.57   102.80    69.27     0.69    6.23    6.28    4.30   0.27   3.04

[root@gitlab ~]# iostat
Linux 2.6.32-696.16.1.el6.x86_64 (gitlab)     10/09/2018     _x86_64_    (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.37    0.00    1.75   16.51    0.00   79.36

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
vda             111.41      7613.95       102.79  504368410    6809368
```

## 四. 进程级磁盘IO监控

### 1. iotop

```
Total DISK READ : 0.00 B/s | Total DISK WRITE : 0.00 B/s
Actual DISK READ: 0.00 B/s | Actual DISK WRITE: 0.00 B/s
 TID PRIO USER DISK READ DISK WRITE SWAPIN IO> COMMAND  
 1 be/4 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % systemd --switched-root --system --deserialize 22
 2 be/4 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [kthreadd]
 3 be/4 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [ksoftirqd/0]
 5 be/0 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [kworker/0:0H]
 7 rt/4 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [migration/0]
 8 be/4 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [rcu_bh]
 9 be/4 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [rcu_sched]
 10 be/0 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [lru-add-drain]
...以下省略
```

```
选项：
-b ：批量显示，无交互，主要用作记录到文件。
-o ：只显示有io操作的进程
-n # ：总共显示几次
-d # ：显示的时间间隔
-u USERNAME ：显示指定用户打开进程的IO状况
-p PID ：显示指定进程的IO状况
```

```
快捷键：
左右箭头：改变排序方式，默认是按IO排序。
r：改变排序顺序。
o：只显示有IO输出的进程。
p：进程/线程的显示方式的切换。
a：显示累积使用量。
q：退出。
```

例：显示进程ID为2372的IO状况，输出至终端，显示一次（性能监控时常用）。

```
[root@lxk ~]# iotop -b -p 2372 -n 1
Total DISK READ : 0.00 B/s | Total DISK WRITE : 0.00 B/s
Actual DISK READ: 0.00 B/s | Actual DISK WRITE: 0.00 B/s
 TID PRIO USER DISK READ DISK WRITE SWAPIN IO COMMAND
 2372 be/4 root 0.00 B/s 0.00 B/s 0.00 % 0.00 % [kworker/0:1]
```

### 2. pidstat

##### pidstat - Report statistics for Linux tasks.

##### OPTIONS

```
-C comm
 Display only tasks whose command name includes the string comm. This string can be a regular expression.
 只显示进程名中包含comm的进程，comm可以是正则表达式
-d Report I/O statistics (kernels 2.6.20 and later only). The following values may be displayed:
 报告I/O的统计信息（2.6.20版后内核版本支持该功能）。会显示以下值：
 UID
 The real user identification number of the task being monitored.
 用户ID号
 USER
 The name of the real user owning the task being monitored.
 运行进程的用户名
 PID
 The identification number of the task being monitored.
 进程号
 kB_rd/s
 Number of kilobytes the task has caused to be read from disk per second.
 每秒此进程从磁盘读取的千字节数
 kB_wr/s
 Number of kilobytes the task has caused, or shall cause to be written to disk per second.
 此进程已经或者将要写入磁盘的每秒千字节数
 kB_ccwr/s
 Number of kilobytes whose writing to disk has been cancelled by the task. This may occur when the task truncates some dirty pagecache. In this case, some IO which another task has been accounted for will not be happening.
 由任务取消的写入磁盘的千字节数
 Command
 The command name of the task.
 命令的名字
-u Report CPU utilization. 报告cpu使用情况，会显示以下内容：
 UID
 The real user identification number of the task being monitored.
 被监视任务的真实用户标识号。
 USER
 The name of the real user owning the task being monitored.
 被监视任务用户的真实用户名
 PID
 The identification number of the task being monitored.
 被监控任务的pid号
 %usr
 Percentage of CPU used by the task while executing at the user level (application), with or without nice priority. Note that this field does NOT include time spent running a virtual processor.
 CPU在用户空间使用情况（百分比）。包含或不包含nice优先级。这个字段不包括运行虚拟处理器的时间。
 %system
 Percentage of CPU used by the task while executing at the system level (kernel).
 任务在内核空间执行时使用的CPU百分比。
 %guest
 Percentage of CPU spent by the task in virtual machine (running a virtual processor).
 任务在虚拟机中消耗的CPU百分比（运行在虚拟处理器之上）
 %CPU
 Total percentage of CPU time used by the task. In an SMP environment, the task's CPU usage will be divided by the total number of CPU's if option -I has been entered on the command line.
 任务使用的总的CPU百分比。
 CPU
 Processor number to which the task is attached.
 任务在哪个CPU上执行
 Command
 The command name of the task. 执行任务的命令
   在报告任务及其所有子任务的全局统计数据时，可能会显示以下值:
    UID             被监视任务的真实用户标识号。
    USER            与子任务一起被监视的任务所属的实际用户的名称。
    PID             与子任务一起被监视的任务所属的用户的PID号
    usr-ms          CPU在处理任务及其子任务时，在用户空间花费的毫秒数。
    system-ms       CPU在处理任务及其子任务时，在内核空间花费的毫秒数。
    guest-ms        CPU在处理任务及其子任务时，在虚拟机上花费的毫秒数。
    Command         执行任务的命令

-l Display the process command name and all its arguments.
 显示进程的命令名和它的全部参数
-I In an SMP environment, indicate that tasks CPU usage (as displayed by option -u ) should be divided by the total number of processors.
 显示
-p { pid [,...] | SELF | ALL }
 Select tasks (processes) for which statistics are to be reported.  
 显示指定pid进程的报告

-r Report page faults and memory utilization. 报告页面错误及内存使用量，会显示以下值：
 minflt/s
 Total number of minor faults the task has made per second, those which have not required loading a memory page from disk.
 每秒次缺页错误次数(minor page faults)，次缺页错误次数意即虚拟内存地址映射成物理内存地址产生的page fault次数
 majflt/s
 Total number of major faults the task has made per second, those which have required loading a memory page from disk.
 每秒主缺页错误次数(major page faults)，当虚拟内存地址映射成物理内存地址时，相应的page在swap中，这样的page fault为major page fault，一般在内存使用紧张时产生
 minflt-nr
 Total number of minor faults made by the task and all its children, and collected during the interval of time.
 在指定的时间间隔内收集的进程和其子进程的次缺页错误次数
 majflt-nr
 Total number of major faults made by the task and all its children, and collected during the interval of time.
 在指定的时间间隔内收集的进程和其子进程的主缺页错误次数
-t Also display statistics for threads associated with selected tasks.This option adds the following values to the reports:
 显示任务线程
 TGID
 The identification number of the thread group leader.
 线程组父进程的ID号
 TID
 The identification number of the thread being monitored.
 被监视线程的ID号。
-s Report stack utilization. 报告堆栈的利用率
 StkSize
 The amount of memory in kilobytes reserved for the task as stack, but not necessarily used.
 为任务保留的以千字节为单位的内存量，但不一定使用。
 StkRef
 The amount of memory in kilobytes used as stack, referenced by the task.
 stack使用的总内存。

-w Report task switching activity (kernels 2.6.23 and later only). The following values may be displayed:
 cswch/s
 Total number of voluntary context switches the task made per second. A voluntary context switch occurs when a task blocks because it requires a resource that is unavailable.
 每秒切换任务的自愿上下文总数。当任务因资源不足而阻塞时，就会发生自愿上下文切换。
 nvcswch/s
 Total number of non voluntary context switches the task made per second. A involuntary context switch takes place when a task executes for the duration of its time slice and then is forced to relinquish the processor.
 每秒完成的非自愿上下文切换任务的总数。当任务在其时间片期间执行，被迫放弃处理器时，会发生非自愿的上下文切换。如大量进程在争取CPU、进程时间片已经到等原因
```

##### EXAMPLES：

```
~]# pidstat 2 5
 Display five reports of CPU statistics for every active task in the system at two second intervals.
 每2秒显示5次
 ~]# pidstat -r -p 1643 2 5
 Display five reports of page faults and memory statistics for PID 1643 at two second intervals.
 每2秒显示5次PID号为1643进程的页面错误报告和内存统计数据。
 ~]# pidstat -C "fox|bird" -r -p ALL
 Display global page faults and memory statistics for all the processes whose command name includes the string "fox" or "bird".
 显示进程名中包含fox或bird的所有进程的页面错误和内存使用量
```
