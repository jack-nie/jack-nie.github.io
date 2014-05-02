---
layout: post
title:  "Windows 下自动连接 NTU VPN（不用每次输入用户名密码）"
date:   2012-04-14 21:17:59
categories: 
- Notes 
tags:
- VPN
- NTU
- Windows

---

在 Windows 下连接 NTU 的 VPN 一般使用 Juniper Network 的 Network Connection 程序。每次连接都要手动输入用户名密码，非常不方便。今天研究了一下通过命令行连接 NTU VPN。通过命令行可以实现在桌面创建一个快捷方式，双击图标直接连接 NTU VPN。

假设已经安装好了 NTU 官方提供的[SSL VPN 程序](http://www.ntu.edu.sg/cits/itnetworking/remoteaccess/Pages/quickstartguide.aspx#winvpn) (Juniper Networks - Network Connection)。

这里其实是使用一个叫 nclauncher 的命令程序。该文件位于 Network Connection 程序的安装目录下。Network Connection 程序的默认安装目录通常是 `%ProgramFiles%\Juniper Networks`。 该命令的用法可以通过输入 `nclauncher -help` 获得。在桌面创建快捷连接的步骤如下：

1. 打开记事本，输入以下内容	

		"%ProgramFiles%\Juniper Networks\Network Connect 7.3.0\nclauncher" -url https://ntuvpn.ntu.edu.sg/ -u [用户名] -p [密码] -r [域]"
		EXIT
其中 `[用户名]` 和 `[密码]` 分别是 NTU 分配的用户名和密码，[域] 可填写 **Student**, **Assoc** 或者 **Staff**，注意大小写不能错，第一个字母必须是大写。

    `%ProgramFiles%` 是程序默认的安装路径。如果操作系统是 64 位 Windows，则得将 `%ProgramFiles%` 改为 `%ProgramFiles(x86)%`。
    
2. 保存该文件为 .bat 或者 .cmd 文件到本地磁盘，如桌面；
3. 若要连接 VPN，双击该文件即可。