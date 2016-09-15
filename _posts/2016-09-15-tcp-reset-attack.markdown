---
layout: post
title:  "TCP复位攻击"
date:   "2016-05-08"
keywords: ["tcp", "复位攻击", "rst"]
description: "tcp rst 复位攻击"
category: "TCP/IP attack series"
tags: ["TCP/IP"]
---
{% include JB/setup %}
虽然TCP/IP是一个可靠的面向连接的协议，但是仍然有许多的漏洞，我将会写一个专题来详细的阐述这些相关的漏洞，本文为开篇，将要讲解rst复位攻击。
在讲解之前让我们先回顾一下TCP的基本信息

### TCP概览

TCP数据报被封装在一个IP数据报中，如图［1］所示。

图（2）展示的是TCP首部的数据格式，如果不计任选字段，它将会是20个字节。

由图（2）可以清楚的看到TCP数据报的头部包含一个16位的源端口地址和一个16位的目的端口地址，用于寻找发送端和接收端的应用程序。这两个端口号和IP首部中的发送端IP地址和接收端IP地址结合起来就能够唯一确定一个TCP连接。

![Alt "TCP包首部"](/assets/images/D1DEF2E2-02AF-4915-BDF0-4D0D05A15B9C.png)

### TCP连接的建立

接下来讲解TCP是如何建立连接的，也就是通常所说的三次握手的过程。为了简便起见，我直接使用rails new blog创建一个app，然后运行rails s启动服务，然后本机运行`telnet localhost 3000`。再开一个窗口运行`sudo tcpdump -i lo0 port 3000`，得到如下输出。

    16:44:23.751241 IP6 localhost.59405 > localhost.hbci: Flags [S], seq 396289691, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 896927289 ecr 0,sackOK,eol], length 0
    16:44:23.751315 IP6 localhost.hbci > localhost.59405: Flags [S.], seq 588037038, ack 396289692, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 896927289 ecr 896927289,sackOK,eol], length 0
    16:44:23.751331 IP6 localhost.59405 > localhost.hbci: Flags [.], ack 1, win 12743, options [nop,nop,TS val 896927289 ecr 896927289], length 0
    16:44:23.751345 IP6 localhost.hbci > localhost.59405: Flags [.], ack 1, win 12743, options [nop,nop,TS val 896927289 ecr 896927289], length 0
    16:44:53.758011 IP6 localhost.hbci > localhost.59405: Flags [F.], seq 1, ack 1, win 12743, options [nop,nop,TS val 896957275 ecr 896927289], length 0
    16:44:53.758085 IP6 localhost.59405 > localhost.hbci: Flags [.], ack 2, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0
    16:44:53.758101 IP6 localhost.hbci > localhost.59405: Flags [.], ack 1, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0
    16:44:53.758188 IP6 localhost.59405 > localhost.hbci: Flags [F.], seq 1, ack 2, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0
    16:44:53.758260 IP6 localhost.hbci > localhost.59405: Flags [.], ack 2, win 12743, options [nop,nop,TS val 896957275 ecr 896957275], length 0

查看第三条可看出客户端的确认序号与描述不符，可能是mac os telnet程序采用了不同的实现？（待确认）。

改用浏览器直接请求，为避免不必要的干扰仅截取一部分进行说明：

    17:31:09.583497 IP6 localhost.59700 > localhost.hbci: Flags [S], seq 608092820, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 899727695 ecr 0,sackOK,eol], length 0
    17:31:09.583645 IP6 localhost.hbci > localhost.59700: Flags [S.], seq 2325301428, ack 608092821, win 65535, options [mss 16324,nop,wscale 5,nop,nop,TS val 899727695 ecr 899727695,sackOK,eol], length 0
    17:31:09.583673 IP6 localhost.59700 > localhost.hbci: Flags [.], ack 2325301429, win 12743, options [nop,nop,TS val 899727695 ecr 899727695], length 0

现在让我们来逐条分析以上的输出来看一下为了建立一条TCP连接要经过那些步骤：

* 客户端也就是telnet程序首先发送一个SYN段表明其打算连接的服务端的端口以及初始序号ISN（seq 608092820), 此为报文段1。
* 服务器发回包含服务器的初始序号的SYN(seq 2325301428)（报文段2）, 同时设置确认序号客户端初始序号ISN ＋ 1， ACK ＝ ISN ＋ 1(ack 608092821), 对客户端发送的SYN进行确认。
* 客户端设置确认序号为服务端初始序号ISN ＋ 1,  ACK ＝ ISN ＋ 1(ack 2325301429), 对服务端的SYN报文段进行确认。
