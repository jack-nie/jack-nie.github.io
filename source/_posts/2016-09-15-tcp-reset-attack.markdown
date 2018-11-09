---
layout: post
title:  "TCP复位攻击"
date:   "2016-09-15"
keywords: ["tcp", "复位攻击", "rst"]
description: "tcp 复位攻击"
category: "TCP/IP"
tags: ["TCP/IP"]
---
虽然TCP/IP是一个可靠的面向连接的协议，但是仍然有许多的漏洞。本文将要讲解其中的一种: rst复位攻击, 在这之前让我们先回顾一下TCP的基础知识。

###  TCP概览

TCP数据报被封装在一个IP数据报中，如图［1］所示。
![Alt "TCP包首部"](/assets/images/2BE3318E-3915-4602-9E18-5ED74CFCD274.png)

图[2]展示的是TCP首部的数据格式，如果不计任选字段，它将会是20个字节。
![Alt "TCP数据在IP数据报中的封装"](/assets/images/438E3D81-ADF4-473C-BFED-163687225A04.png)

由图[2]可以清楚的看到TCP数据报的头部包含一个16位的源端口地址和一个16位的目的端口地址，用于寻找发送端和接收端的应用程序。这两个端口号和IP首部中的发送端IP地址和接收端IP地址结合起来就能够唯一确定一个TCP连接。

###  TCP连接的建立

接下来讲解TCP是如何建立连接的，也就是通常所说的三次握手的过程。为了简便起见，我直接使用`rails new blog`创建一个app，然后运行`rails s`启动服务，然后本机运行`telnet localhost 3000`。再开一个窗口运行`sudo tcpdump -i lo0 port 3000`，得到如下输出。

    16:44:23.751241 IP6 localhost.59405 > localhost.hbci: Flags [S], seq 396289691, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 896927289 ecr 0,sackOK,eol], length 0
    16:44:23.751315 IP6 localhost.hbci > localhost.59405: Flags [S.], seq 588037038, ack 396289692, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 896927289 ecr 896927289,sackOK,eol], length 0
    16:44:23.751331 IP6 localhost.59405 > localhost.hbci: Flags [.], ack 1, win 12743, options [nop,nop,TS val 896927289 ecr 896927289], length 0
    16:44:23.751345 IP6 localhost.hbci > localhost.59405: Flags [.], ack 1, win 12743, options [nop,nop,TS val 896927289 ecr 896927289], length 0
    16:44:53.758011 IP6 localhost.hbci > localhost.59405: Flags [F.], seq 1, ack 1, win 12743, options [nop,nop,TS val 896957275 ecr 896927289], length 0
    16:44:53.758085 IP6 localhost.59405 > localhost.hbci: Flags [.], ack 2, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0
    16:44:53.758101 IP6 localhost.hbci > localhost.59405: Flags [.], ack 1, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0
    16:44:53.758188 IP6 localhost.59405 > localhost.hbci: Flags [F.], seq 1, ack 2, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0
    16:44:53.758260 IP6 localhost.hbci > localhost.59405: Flags [.], ack 2, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0

查看第三条可看出客户端的确认序号与描述不符，查阅资料发现正确的telnet命令应该是'sudo tcpdump -i lo0 port 3000 -s'。

> the ack sequence number is a small integer (1). The first time tcpdump sees a tcp 'conversation', it prints the sequence number from the packet. On subsequent packets of the conversation, the difference between the current packet's sequence number and this initial sequence number is printed. This means that sequence numbers after the first can be interpreted as relative byte positions in the conversation's data stream (with the first data byte each direction being '1'). '-S' will override this feature, causing the original sequence numbers to be output.

改用浏览器直接请求，为避免不必要的干扰仅截取一部分进行说明：

    17:31:09.583497 IP6 localhost.59700 > localhost.hbci: Flags [S], seq 608092820, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 899727695 ecr 0,sackOK,eol], length 0
    17:31:09.583645 IP6 localhost.hbci > localhost.59700: Flags [S.], seq 2325301428, ack 608092821, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 899727695 ecr 899727695,sackOK,eol], length 0
    17:31:09.583673 IP6 localhost.59700 > localhost.hbci: Flags [.], ack 2325301429, win 12743, options [nop,nop,TS val 899727695 ecr 899727695], length 0

现在让我们来逐条分析以上的输出来看一下为了建立一条TCP连接要经过那些步骤：

* 客户端首先发送一个SYN段表明其打算连接的服务端的端口以及初始序号ISN（seq 608092820), 此为报文段1。
* 服务器发回包含服务器的初始序号的SYN(seq 2325301428)（报文段2）, 同时设置确认序号客户端初始序号ISN ＋ 1， ACK ＝ ISN ＋ 1(ack 608092821), 对客户端发送的SYN进行确认。
* 客户端设置确认序号为服务端初始序号ISN ＋ 1,  ACK ＝ ISN ＋ 1(ack 2325301429), 对服务端的SYN报文段进行确认。

###  RST复位标志

TCP首部中的RST比特是用于"复位"的，发送RST包关闭连接时，不必等缓冲区的包都发出去，直接就丢弃缓存区的包发送RST包。而接收端收到RST包后，也不必发送ACK包来确认。

###  TCP Reset Attacks

* 首先攻击者需要劫持TCP session。
* 攻击者发送RST标志位置1的包到主机A和主机B，或者二者之一。
* 因为主机A和主机B并不知道这些包是由攻击者发出的，所以A，B正常的处理这些包。
* 因为这些包包含RST为1的标志位，所以A和B之间的连接就关闭了。

下面我们来讲一下攻击者如何劫持TCP session。

假设主机A和主机B之间已经建立了TCP连接，那么一个攻击者能够监控主机A和主机B之间的数据包, 那么劫持TCP session就可以采用如下步骤：

* 攻击者使用Dos攻击主机B，中断主机B和主机A之间的通信。
* 现在攻击者就能够预测主机A期望主机B发送的包所包含的序列号。
* 攻击者准备一个这样的包发送给主机A。
* 主机A不知道这个包是假冒的，仍然认为该包来自主机B。
* 攻击者可以利用这个包做出各种神奇的攻击。

###  参考文献

- [TCP Attacks: TCP Sequence Number Prediction and TCP Reset Attacks](http://www.thegeekstuff.com/2012/01/tcp-sequence-number-attacks/)
- [TCP/IP详解卷1: 协议](http://item.jd.com/10057317.html)

