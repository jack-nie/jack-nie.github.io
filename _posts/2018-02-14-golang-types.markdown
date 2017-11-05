---
layout: post
title:  "Golang中类型系统详解"
date:   "2018-02-14"
keywords: ["golang", "name type", "uname type"]
description: "golang 类型系统详解 "
category: "Go"
tags: ["Go"]
---
{% include JB/setup %}

最近看到一篇关于指导go语言新手怎么学习的[文章](https://blog.rubylearning.com/best-practices-for-a-new-go-developer-8660384302fc "文章")，里面讲到了named type, unnamed type，恰好这一块有点印象，但是记忆已经不是太清晰了，所以本文总结一下，加深理解和记忆。

### 1.Named Type VS Unnamed Type ###

golang提供了一些基础类型(pre-delared)如boolean, numeric, string。基础类型可以组合成组合类型，这些类型有数组，切片，指针，结构体，map以及channel。

通过关键字type定义的类型叫做named type，前文描述的组合类型叫做unamed type。值得提出的一点就是基础类型也是named type。

可以为named type添加方法集，但是为unamed type添加方法集将会造成编译不通过。新定义的类型不会继承named type或者unamed type的方法集。

### 2.底层类型 ###

named type的底层类型是基础类型或者组合类型，基础类型和组合类型的底层类型是它们自己。

### 3.赋值 ###

go是静态类型的，每个变量都有一个静态类型，在编译时，一个变量只有一种类型，不能被隐式的转换。
如果named type的底层类型是基础类型，那么通过named type声明的变量和基础类型声明的变量之间不可以赋值，即使这两者的底层类型是一致的。
unamed type和组合类型之间是可以相互赋值的。
