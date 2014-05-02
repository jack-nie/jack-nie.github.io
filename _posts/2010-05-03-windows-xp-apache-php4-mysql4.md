---
layout: post
title:  "Windows XP 下搭建 Apache + PHP 4 + MySQL 4 环境"
date:   2010-05-03 14:17:59
categories: 
- Notes 
tags:
- Windows
- PHP
- Apache
- MySQL

---

这个年头估计没人写这种旧版本的 Web 服务器搭建教程了。这里只是做个存档，由于我几年前的数据还没转换成新的版本，所以得搭建老的平台进行数据转换。

在开始之前，首先下载到以下组件的相应版本：

*   Apache HTTP Server2.063
*   PHP 4.4.9
*   MySQL 4.026
*   phpMyAdmin 2.0 以上

## Step 1: 安装 Apache HTTP Server
---

1.  双击 Apache 的安装文件 apache_2.0.63-win32-x86-openssl-0.9.7m.msi；
2.  点 Next 按钮；
3.  选择同意 Agreement，点 Next 按钮；
4.  再点 Next 按钮；
5.  在 Server Information 对话框，要填写域名和服务器名等信息。如果是本地安装，一般在 Network Domain 和 Server Name 都填 localhost。下面的 Email 随便填把也没什么用。点 Next；
6.  Setup Type，选 Custom，Next；
7.  安装地址，为了管理方便，统一把程序都安装到 `C:\Webserver` 目录下。这里填 `C:\Webserver`，之后 Apache 会被安装到 `C:\Webserver\Apache2` 目录下；
8.  点 Next，再点 Install 开始安装。
9.  验证 Apache 是否安装成功：打开浏览器在地址栏里输入 `localhost`，如果看到提示安装成功的页面就对了。

## Step 2: 配置 Apache
---

1.  打开 `C:\Webserver\Apache2\conf\httpd.conf` 文件，这个是 Apache 的主要配置文件。
2.  搜索字符串

		C:/Webserver/Apache2/htdocs

	这个定义根目录的位置，文件一共有两个地方，把它们都改为

		C:/Webserver/wwwroot

3.  在 `C:\Webserver` 下手动新建一个 wwwroot 文件夹。

## Step 3: 安装与配置 PHP
---

安装了这个 Apache 才能对 .php 网页进行解析。

1.  解压 PHP 安装包到 `C:\Webserver`下，把 php 文件夹的名字改成 php4。这样所有的 PHP 程序文件都位于 `C:\Webserver\php4` 下。
2.  复制 `php.ini-dist` 文件到 `C:\Windows` 下，将其重命名为 `php.ini`
3.  打开 php.ini，搜索

		;extension=php_mbstring.dll

	可以看到有很多的 extension 前面被分号注释掉了，要启用哪一项就把前面的分号去掉。这里启用 **gd2** 库（图形）和 **mbstring**。

    继续搜索

		extension_dir = "./"

	将其改为

		extension_dir = "C:/Webserver/php4/extensions"

	保存文件退出。

4.  复制文件 `php4ts.dll` 到 `C:\Windows\system32` 文件夹下
5.  打开 `C:\Webserver\Apache2\conf\httpd.conf` 文件，搜索

		#LoadModule ssl_module modules/mod_ssl.so

	在下面一行添加

		LoadModule php4_module C:/Webserver/php4/sapi/php4apache2.dll

	继续搜索

		AddType application/x-gzip .gz .tgz

	下面添加

		Addtype application/x-httpd-php .php

	继续搜索

		DirectoryIndex index.html index.html.var

	修改为

		DirectoryIndex index.html index.html.var index.php

	保存文件并退出。

6.  重新启动 Apache 服务，打开一个命令窗口，依次输入

		net stop apache2
		net start apache2

	然后右键单击右下角的 Apache 的图标，选择 Open Apache Moniter，如果左下角显示了 PHP 的版本号，说明安装成功了。

7.  为了进一部验证，到 C:\Webserver\wwwroot 下建立一个空白文档 index.php，编辑它写入一下 PHP 代码：

		<?php phpinfo(); ?>

	在浏览器里输入 localhost，如果出现 PHP 的配置页面，则证明 PHP 安装成功了。

## Step 4: 安装MySQL数据库
---

1.  运行 MySQL 的安装文件，过程比较简单，中间一部选择安装路径到 `C:\Webserver\mysql`
2.  安装完成后，去 `C:\Webserver\mysql\bin` 文件夹下，启动 **winmysqladmin.exe**。这是会弹出一个窗口，提示输入用户名密码。这个时候其实输入什么都可以继续，比如用户名 root，密码为空。
3.  如果正常的话，右下角任务栏会多出一个红绿灯的图标，并且绿灯亮。