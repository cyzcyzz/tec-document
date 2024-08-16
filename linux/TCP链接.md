---
title: "TCP问题"
date: 2020-03-06T23:03:27+08:00
description: ""
draft: false
tags: []
categories: [network]
---

# TCP连接总结
&emsp;&emsp;最近一些时间线上的服务器会报TcpListenDrop和TcpOverFlowed的警告，一开始是不知道这究竟什么含义,然后开始研究tcp队列的相关含义，下面进行一些总结。
&emsp;&emsp;TCP三次握手，四次分手是大家耳熟能详的过程，甚至多数人都能说明“为什么三次握手和四次握手”，但是当我们真正去实际的解决问题的时候，我们往往不能将这些理论和实际结合起来，我相信大多数的技术人员甚至不能说明整个连接过程的11种状态，这也就是为什么我们不能实际的解决问题，因为状态的变化，可以为我们提供非常多的有效信息，下面一张图搞定这些状态的变化和出现的终端。
![net-one](http://cdn.oyfacc.cn/tcp-3link.png)
这张图很清晰展现了tcp从建立连接到数据传输完成断开连接的整个状态的变化，这里对每个状态简要的进行说明下：

#### 服务端
&emsp;这里一般是指被动接受的一方，不一定非得是服务器的一端；
&emsp;`LISTEN`:这个状态是服务端最开始状态，此时等待客户端进行连接；
&emsp;`SYN_RCVD`:这个状态是等待客户端进行确认的状态，此时客户端发来确认位，即可建立连接；
&emsp;`ESTABLISHED`: 著名的连接建立状态；
&emsp;`CLOSE_WAIT`:客户端主动发起分手，服务端收到请求，并且确认后的状态；
&emsp;`LAST_ACK`:服务端发起最后一次分手后的状态，此时等待客户端确认;
&emsp;`CLOSE`:关闭状态，客户端回复最后一次分手后的状态；

#### 客户端
&emsp;这里一般是指主动发起连接的一方，不一定是终端用户；
&emsp;`CLOSED`:被动关闭状态，此时可以调connect方法连接服务端；
&emsp;`SYN_SEND`:发送完连接后的状态,此时等待服务端回复；
&emsp;`FIN_WAIT1`:主动调用close方法后的状态，此时发送了一个fin信号；
&emsp;`FIN_WAIT2`: 收到服务端对第一次发起fin状态的回复后的状态，此时等待服务端调用close方法，发起最后一次fin；
&emsp;`TIME_WAIT`:此时服务端主动发送了最后一次FIN,客户端接收后，恢复了确认位，进入这个状态，等待最后的超市关闭,下面一段记录了几个内核参数，用于调整TCP的一些状态；
```linux
net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_fin_timeout 修改系默认的 TIMEOUT 时间
```


