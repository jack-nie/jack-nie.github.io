---
layout: post
title:  "Golang中的handle和handleFunc"
date:   "2018-03-31"
keywords: ["golang", "handle", "handleFunc", "net/http"]
description: "golang中handle和handleFunc的分析"
category: "Go"
tags: ["Go"]
---
{% include JB/setup %}

在golang的标准库文档中有这么一段示例：

```
http.Handle("/foo", fooHandler)

http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})

log.Fatal(http.ListenAndServe(":8080", nil))
```

handle和handleFunc都接收两个参数，第一个都是将要访问的路由，是一个string类型，不同的是第二个参数，前者接受一个实现了Handlerinterface的type，后者是一个handler的方法。
之所以要有handle是为了当逻辑比较复杂时可以在请求的过程中加入一些状态，一个实现了Handler类型的type可以很容易的做到这一点。
当然对于简单的场景，使用handleFunc开发效率更高一些。

在server.go的源码中可以看到handlerFunc的具体实现：

```
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

首先自定义一个类型为func的type，添加一个ServeHttp的方法，并在方法体中调用自身，这实际上是一个wrapper，主要是为了方便。
