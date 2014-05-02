---
layout: post
title:  "IEEE 论文 LaTeX 模板 – LaTeX2e 类文件"
date:   2010-07-13 14:17:59
categories: 
- Notes 
tags:
- IEEE
- LaTeX
- paper

---

研究人员如果把大量时间用来写论文上显然是不本末倒置。诚然论文对各位学者们十分重要，但其实说到底这些只是“虚荣”。

写论文的很大一部分时间是用在排版上。特别是使用 LaTeX 这样的排版方式，大多数软件不是所见即所得的编辑工具，排版要下极大的功夫。其实现在的各种 LaTeX 排版工具都有“宏命令”，如果我们能有这样一个 IEEE 的“模板”，只关心文章内容而不是文章的界面和结构，这样定能事倍功半。

[这里](http://mocha-java.uccs.edu/ieee/ieeeftp/ieeecls.pdf)有一个 pdf 模板，打开看看和平时看的 paper 格式基本一模一样。这个 pdf 是用一个 TeX 模板生成的。

### LaTeX2e files for formatting IEEE papers

类文件 ieee.cls is well documented in the above paper. 这里提供这个文件以及大量的例子。

*   [ieee.cls](http://mocha-java.uccs.edu/ieee/ieeeftp/ieee.cls) 类文件
*   [ieeecls.pdf](http://mocha-java.uccs.edu/ieee/ieeeftp/ieeecls.pdf) IEEE 论文：Specification of Common IEEE Styles，可以看成模板生成的 pdf 示例文件。
*   [ieeecls.tex](http://mocha-java.uccs.edu/ieee/ieeeftp/ieeecls.tex) 以上文章 LaTeX2e 源文件。这个文件是很好的学习 LaTeX 材料，对照文章和源代码分析，效果很好。
*   [ieeeskel.tex](http://mocha-java.uccs.edu/ieee/ieeeftp/ieeeskel.tex) IEEE 论文的简单框架。可以用这个开始写一篇新文章。
*   [ieeefig.sty](http://mocha-java.uccs.edu/ieee/ieeeftp/ieeefig.sty) 图片格式。
下载以上 .cls 和 .tex 文件到同一目录下，用任何 LaTeX 编辑器打开即可。

参考：[http://mocha-java.uccs.edu/ieee/](http://mocha-java.uccs.edu/ieee/)