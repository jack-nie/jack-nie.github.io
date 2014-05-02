---
layout: post
title:  "Mac OS X 下修改文件属性：创建时间、修改时间"
date:   2011-09-12 14:17:59
categories: 
- Notes 
tags:
- OS X

---

## 格式
---

*   YYYY 四位数年
*   MM 两位数月
*   DD 两位数日
*   hh 两位数小时
*   mm 两位数分钟

## 修改文件的“修改时间”
---

打开 Terminal，首先输入以下内容，不要回车：
	
	touch -mt YYYYMMDDhhmm

其中 YYYYMMDDhhmm 要替换成期望的时间，比如 201112310101。
打开 Finder，进入需修改的文件所在的文件夹，把改文件拖到 Terminal 窗口，这时文件的路径自动加到了刚才输入的内容的后面，例如

	touch -mt 201112310101 /Volumes/MacHD/Pictures/somefile.jpg

当然你可以自己手动输入文件路径。回车即完成修改。

## 修改文件的“创建时间”
---

步骤和上面修改“修改时间”基本相同，只是把参数 -mt 改成 -t，例如

	touch -t 201112310101 /Volumes/MacHD/Pictures/somefile.jpg

还有批量修改文件日期的方法可以[参考文章链接](http://danilo.ariadoss.com/howto-change-date-modified-date-created-mac/)。