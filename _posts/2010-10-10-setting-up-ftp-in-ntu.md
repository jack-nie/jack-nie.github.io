---
layout: post
title:  "在 NTU 内网架设 FTP"
date:   2010-10-10 14:17:59
categories: 
- Notes 
tags:
- FTP
- NTU

---

越来越发现其实台式机才是王道。不过人在江湖，买个台式机行走起来也是不太方便。学校的电脑还算不错，至少对于我这种不太玩高端游戏的人来说是很满足了。上个星期发现电脑机箱响，经常莫名死机，感觉应该是硬盘的问题。果真不出两天 750 GB 硬盘彻底挂了。立马告诉管理员，然后立马换了一块 1 TB 的新硬盘。

一些东西在学校电脑和自己的笔记本之间转来转去比较麻烦，然而我又是最讨厌用U盘的，于是折腾了一下架了个 FTP，通过 NTU 提供的 VPN 可以实现外网访问。

## 安装和设置 Filezilla FTP Server
---

1.  下载 FTP 服务器。这里用最著名的开源 FTP 服务器 Filezilla：[下载地址](http://filezilla-project.org/download.php?type=server)
2.  安装 Filezilla Server
3.  设置用户：Edit -&gt; Users。增加用户名和密码
4.  设置 FTP 目录：Edit -&gt; Users -&gt; Shared folders。同时设置用户相应的访问权限。
5.  设置 Windows Firewall，Windows Firewall 会阻止外网用户访问该机器，得将 Filezilla Server 添加到特例中去：Control Panel -&gt; Windows Firewall -&gt; Allow a program or feature through Windows Firewall，点下面的 Allow another program 按钮，找到 filezilla server.exe 添加到特例中，然后将 Domain 栏下的勾选上。

## 安装运行 NTU VPN
---

机器位于学校的内网，在外网一般是访问不了内网机器的。NTU 的网络还是不是盖的，很早之前就开始提供 VPN 让大家能从外网访问校内资源。以前是利用拨号，后来推出了一个 SSL VPN 的服务，教程在[这里](http://www.ntu.edu.sg/cits/itnetworking/remoteaccess/Pages/quickstartguide.aspx#sslvpn)。感觉相当有爱，这家伙可是翻墙利器哦。

1.  进入 [https://ntuvpn.ntu.edu.sg/](https://ntuvpn.ntu.edu.sg/)
2.  输入 NTU 帐号密码登录
3.  第一次使用的话会提示下载 ActiveX 或者 Java 的插件。甭管是什么下载安装之。
4.  安装好后会弹出一个小窗口提示连接上了 VPN

## 访问 FTP
---

首先要找到校内机器的 IP 地址。NTU 每台电脑都有独立的 IP 地址（这个比较 NB，没有这个条件自己就不可能建 FTP，因为没有权限做端口映射），都是形如 155.x.x.x 的地址。然后用 FTP 软件或者直接在我的电脑中输入 ftp://155.x.x.x，再输入之前设置的用户名和密码就能访问内网的机器了。

今天就折腾到这里。