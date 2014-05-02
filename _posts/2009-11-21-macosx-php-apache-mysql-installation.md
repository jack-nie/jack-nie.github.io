---
layout: post
title:  "Mac OS X 中架构 PHP + Apache + MySQL 环境"
date:   2009-11-21 14:17:59
categories: 
- Notes 
tags:
- OS X
- PHP
- Apache
- MySQL

---

最近又用回了 Mac OS X。以前也没怎么在 Mac 的环境下维护网站。比较简单的是去下载一个 [XAMPP](http://www.apachefriends.org/en/xampp-macosx.html) (Apache + MySQL + PHP + Perl)。这次我使用 Mac 自带的 Apache 和 PHP，主要参考[这里](http://huajun.w18.net/2009/03/macphp.html)的一篇文章。

## Step 1: 启用 PHP
---

1.  启动终端程序：应用程序 -&gt; 实用工具 -&gt; 终端；
2.  输入以下命令：

	 	sudo su

3.  输入密码后回车；注意如果系统没有设置过密码，可能这里不能继续下去，所以要先去系统偏好设置 -> 账户里设置一个密码。
4.  使用以下命令打开 httpd.conf 文件：

		vi /etc/apache2/httpd.conf

5.  搜索以下内容：

		#LoadModule php5_module libexec/apache2/libphp5.so

6.  去掉前面的注释符 #，保存退出。（:wq）

## Step 2: 启用 Apache
---

Mac OS X 自带 Apache，在系统偏好设置 -> 共享中打开 Web 共享即可。
![system-settings](http://dannyli.net/beta/wp-content/uploads/2009/11/system-settings.png "system-settings")

![sharing](http://dannyli.net/beta/wp-content/uploads/2009/11/sharing.png "sharing")

用浏览器打开本地网站测试一下 Apache 是否工作正常。例如在站点根目录下建立一个 phpinfo.php，里面写上以下内容：
	
		<?php phpinfo(); ?>

然后从浏览器里通过“您的个人网站地址”（上图所示）来测试一下 PHP 工作情况。

## Step 3: 安装 MySQL
---

这部分内容比较多，参见[这里](http://blog.kaishao.idv.tw/?p=905)。

1.  主要步骤就是首先去官网下载安装文件，建议下载 dmg 格式的安装文件，然后进行安装。
2.  打开终端，输入：

		cd /usr/local/mysql
		sudo chown -R mysql data/

	输入密码，然后再输入：

		sudo echo
		sudo ./bin/mysqld_safe &amp;
3.  测试一下 MySQL：

		/usr/local/mysql/bin/mysql test

	如果出现下面的内容就说明安装好了：

	> Welcome to the MySQL monitor. Commands end with ; or \g.Your MySQL connection id is 1 to server version 5.0.45Type 'help;' or '\h' for help. Type '\c' to clear the buffer.mysql&gt;
	
	输入 quit 退出 MySQL 提示符；

4.  设置 MySQL root 密码，输入：
  
		/usr/local/mysql/bin/mysqladmin -u root password {密码}

5.  创建数据库，键入：

		/usr/local/mysql/bin/mysql -u root -p

	输入密码后，再输入 SQL 语句建立数据库：

		CREATE DATABASE {数据库名};

6.  编辑 php.ini 文件。

## 解决 MySQL 不能启动问题
---

第一次装完后重启计算机，发现进入 phpmyadmin 登录会出现错误提示：

> \#2002 - 服务器没有响应 (或者本地 MySQL 服务器的套接字没有正确配置)
	
找了很多解决方法都不成功，最后发现需要创建并修改 php.ini：
	
	sudo cp /etc/php.ini.default /etc/php.ini

找到一行包括：

	mysql.default_socket

将其修改为：

	mysql.default_socket = /tmp/mysql.sock

然后重启 Apache：

	sudo apachectl restart

2010-5-13 Update: 重启计算机后可能出现 MySQL 没有启动的状况，这是因为安装 MySQL 的时候没有安装 MySQLStartItem，位于 MySQL 的安装包里。安装好这个，再执行：

	sudo /Library/StartupItems/MySQLCOM/MySQLCOM start

参见MySQL文档：[http://dev.mysql.com/doc/refman/5.1/zh/installing.html#mac-os-x-installation](http://dev.mysql.com/doc/refman/5.1/zh/installing.html#mac-os-x-installation)