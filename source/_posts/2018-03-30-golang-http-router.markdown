---
layout: post
title:  "Golang中的router"
date:   "2018-03-30"
keywords: ["golang", "router", "ServeMux"]
description: "golang中net/http的路由分析"
category: "Go"
tags: ["Go"]
---

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
2. 较长的路径匹配的优先级会高于较短的路径，如果同时注册了两个路由"/images/"和"/images/avatar",请求的url是"http://localhost:8080/images/avatar/",那么会优先匹配后一条路由而不管这两条路由注册的先后顺序。
3. 任何路径中包含"."或".."元素的请求重定向到等价的没有这两种元素的URL。

go自带的路由实现了一些基本的功能，但是并不完善：

1. 没法处理query paramter如"/user/:id"
2. 没法限定请求的http方法， 如"/user", 用get，post，put，delete等都可以匹配到
#### ServeMux源码

```
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	hosts bool // whether any patterns contain hostnames
}

type muxEntry struct {
	h       Handler
	pattern string
}

```

首先定义ServeMux类型，是一个结构体，包括一个读写锁，一个路由注册器，和一个标示路由是否携带主机名的hosts bool变量。

```
// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

  // 边界情况处理
	if pattern == "" {
		panic("http: invalid pattern")
	}
	if handler == nil {
		panic("http: nil handler")
	}
  // 如果对应的路由已经注册，那么将触发panic
	if _, exist := mux.m[pattern]; exist {
		panic("http: multiple registrations for " + pattern)
	}

  // 如果还没有任何路由注册过，则创建一个map
	if mux.m == nil {
		mux.m = make(map[string]muxEntry)
	}
  // 将路由写入map
	mux.m[pattern] = muxEntry{h: handler, pattern: pattern}

  // 如果路由不以"/"开头，则说明有主机名
	if pattern[0] != '/' {
		mux.hosts = true
	}
}

// HandleFunc封装Handle,处理方式和"net/http" 一致。
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	mux.Handle(pattern, HandlerFunc(handler))
}
```

```
// ServeHTTP将请求转发给最匹配的handler处理
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
	if r.RequestURI == "*" {
		if r.ProtoAtLeast(1, 1) {
			w.Header().Set("Connection", "close")
		}
		w.WriteHeader(StatusBadRequest)
		return
	}
	h, _ := mux.Handler(r)
	h.ServeHTTP(w, r)
}
```

```
// Handler 返回根据路由匹配的Handler,
// 它永远返回一个非空的Handler，如果请求的路由是非标准化的，那么将会对其进行转换。
// 如果路由带有端口号，则在匹配的时候忽略。
//
// 如果是connect请求，则不会对host和path做处理。
// 如果没有匹配的，则返回""
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {

	// CONNECT requests are not canonicalized.
	if r.Method == "CONNECT" {
		// If r.URL.Path is /tree and its handler is not registered,
		// the /tree -> /tree/ redirect applies to CONNECT requests
		// but the path canonicalization does not.
		if u, ok := mux.redirectToPathSlash(r.URL.Host, r.URL.Path, r.URL); ok {
			return RedirectHandler(u.String(), StatusMovedPermanently), u.Path
		}

		return mux.handler(r.Host, r.URL.Path)
	}

  // 在交给mux.hanlder处理之前，先删除port，清理path
	host := stripHostPort(r.Host)
	path := cleanPath(r.URL.Path)

  // 如果"/tree"没有注册，则返回一个带有3XX code的Handler，交给"/tree/"
	if u, ok := mux.redirectToPathSlash(host, path, r.URL); ok {
		return RedirectHandler(u.String(), StatusMovedPermanently), u.Path
	}

  // 如果处理的后的路径和原始路径不一致，交给RedirectHandler处理
	if path != r.URL.Path {
		_, pattern = mux.handler(host, path)
		url := *r.URL
		url.Path = path
		return RedirectHandler(url.String(), StatusMovedPermanently), pattern
	}

  // 否则由mux.handler处理
	return mux.handler(host, r.URL.Path)
}
```

```
// handler是Handler的主要处理逻辑，对path做标准化处理。
func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
	mux.mu.RLock()
	defer mux.mu.RUnlock()

  // 定义了hosts的路由有更高的优先级
	if mux.hosts {
		h, pattern = mux.match(host + path)
	}
  // 如果没有匹配到，则交给后续处理
	if h == nil {
		h, pattern = mux.match(path)
	}
	if h == nil {
		h, pattern = NotFoundHandler(), ""
	}
	return
}
```

```
// Return the canonical path for p, eliminating . and .. elements.
func cleanPath(p string) string {
	if p == "" {
		return "/"
	}
	if p[0] != '/' {
		p = "/" + p
	}
	np := path.Clean(p)
	// path.Clean removes trailing slash except for root;
	// put the trailing slash back if necessary.
	if p[len(p)-1] == '/' && np != "/" {
		np += "/"
	}
	return np
}

// 删除port
func stripHostPort(h string) string {
	// If no port on host, return unchanged
	if strings.IndexByte(h, ':') == -1 {
		return h
	}
	host, _, err := net.SplitHostPort(h)
	if err != nil {
		return h // on error, return unchanged
	}
	return host
}

// 路由越精确优先级越高
func (mux *ServeMux) match(path string) (h Handler, pattern string) {
	// Check for exact match first.
	v, ok := mux.m[path]
	if ok {
		return v.h, v.pattern
	}

  // 找最长的路径
	var n = 0
	for k, v := range mux.m {
		if !pathMatch(k, path) {
			continue
		}
		if h == nil || len(k) > n {
			n = len(k)
			h = v.h
			pattern = v.pattern
		}
	}
	return
}
```

