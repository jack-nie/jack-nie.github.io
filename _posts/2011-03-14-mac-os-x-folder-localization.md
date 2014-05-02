---
layout: post
title:  "Mac OS X 文件夹本地化方法（汉化）"
date:   2011-03-24 14:17:59
categories: 
- Notes 
tags:
- OS X

---

经常看到 OS X 中的一些文件夹名称会随系统语言的更改而自动改变。有时候你想要某个英文的文件夹名称显示成对应的中文，但是又不想改文件夹名（可能由于里面安装了一些程序），是否能用类似的文件夹**本地化 (Localization) **方式来解决？

有这么一个例子：如果使用默认路径安装了 Xcode，其程序文件会放在根目录下名为的 Developer 文件夹内。如果系统语言使用中文，你会发现在根目录下除了这个 Developer 文件夹，其他的的文件夹都是中文名称，如系统、应用程序、用户等。说明除了这几个文件夹使用了文件夹本地化。现在我们想把 Developer 的名字改成“开发者”，如何做到的呢？步骤如下：

## Step 1: 添加字符串至本地化配置文件
---

1.  打开 Finder，进入 /System/Library/CoreService/SystemFolderLocalizations/ (/系统/资源库/CoreService/SystemFolderLocalizations/)。
2.  进入 zh_CN.lproj 文件夹，看到下面有一个文件叫 SystemFolderLocalizations.strings。字符串就存放在这个文件里。
3.  修改文件权限。右键单击 SystemFolderLocalizations.strings，选“显示简介”，在“共享与权限”中添加自己的用户名并设置权限为读与写。
4.  用文本编辑软件打开该文件，添加以下一行：
 
	"Developer" = "开发者";

5.  保存退出

## Step 2: 添加 .localized 文件
---

如果一个文件夹要使用本地化，它的下面必须有一个名为 .localized 的文件隐藏文件。

1.  设置 Finder 中显示隐藏文件，参看[这里](http://dannyli.net/139/changing-macosx-default-settings-in-terminal#3 "更改 Mac OS X 隐藏的默认设置的代码收集")。
2.  复制任意一个文件夹中的 .localized 文件到 /Developer
3.  重启 Finder：按快捷键 Command + Option + Esc，调出“强制退出应用程序”窗口，结束 Finder 进程。
再重新打开 Finder 看文件夹的名字已经变为刚才设置的“开发者”了。

事实上还有另一种文件夹本地化的方法，主要用于应用程序的文件夹。可以参看下面这篇文章：[http://www.chinamac.com/2009/1012/49609.html](http://www.chinamac.com/2009/1012/49609.html)