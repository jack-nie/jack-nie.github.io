---
layout: post
title:  "Golang中的router"
date:   "2018-03-30"
keywords: ["golang", "router", "ServeMux"]
description: "golang中net/http的路由分析"
category: "Go"
tags: ["Go"]
---
{% include JB/setup %}

一个web应用程序要接受来及外部的url请求，一个重要的工作是对url进行解析，并将这次请求转给对应的逻辑代码进行处理，这里就是路由机制大展身手的地方了。

golang中有一个ServeMux类型，该类型同样实现了ServeHttp方法，因而可以直接作为参数传入http.LisenAndServe方法。iServeMux类型是HTTP请求的多路转接器。它会将每一个接收的请求的URL与一个注册模式的列表进行匹配，并调用和URL最匹配的模式的处理器。

下面来看一下具体的应用：

```
package main

import (
	"io"
	"net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, "Hello, world!\n")
}

func echoHandler(w http.ResponseWriter, r *http.Request) {
	io.WriteString(w, r.URL.Path)
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/hello", helloHandler)
	mux.HandleFunc("/", echoHandler)

	http.ListenAndServe(":8080", mux)
}
```

首先通过http.NewServeMux()创建并返回一个新的*ServeMux,然后相应的路由和handler都注册到它上面。
路由的匹配规则如下：


1. 匹配根路径或者以根路径开始的子树，如'/'和'/images/', 注意"/images/"后面的"/",这代表一条子路径，可以匹配任何以"/images/"开始的路径，如果没有"/"则代表叶子，是一个固定的路径。
2.较长的路径匹配的优先级会高于较短的路径，如果同时注册了两个路由"/images/"和"/images/avatar",请求的url是"http://localhost:8080/images/avatar/",那么会优先匹配后一条路由而不管这两条路由注册的先后顺序。
3.任何路径中包含"."或".."元素的请求重定向到等价的没有这两种元素的URL。
