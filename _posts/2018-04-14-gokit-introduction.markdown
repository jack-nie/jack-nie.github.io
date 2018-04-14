---
layout: post
title:  "go-kit简介"
date:   "2018-04-14"
keywords: ["golang", "go-kit"]
description: "go-kit简介"
category: "Go"
tags: ["Go"]
---
{% include JB/setup %}

### gokit是什么

Gokit是一系列的工具的集合，能够帮助你快速的构建健壮的，可靠的，可维护的微服务。它提供了一系列构建微服务的成熟的模式和惯用法，背后有着一群经验丰富的开发者支持，并且已经在生产环境中被广泛的使用。

### gokit的架构

gokit不同于传统的MVC的框架，它只是一系列工具的组合，他有着自己的层次结构，主要有三层，分别是transport，endpoint和service层。

### transport层

这是一个抽象的层级，对应真实世界中的http/grpc/thrift等，通过gokit你可以在同一个微服务中同时支持http和grpc。

### endpoint层

endpoint层对应于controller中的action，主要是实现安全逻辑的地方，如果你要同时支持http和grpc，那么你将需要创建两个方法同时路由到同一个endpoint。

### service层

service是具体的业务逻辑实现的层级，在这里，你应该使用接口，并且通过实现这些接口来构建具体的业务逻辑。一个service通常聚合了多个endpoints，在service层，你应该使用clean architecture或者六边形模型，也就是说你的service层不需要知道enpoint以及transport层的具体实现，也不需要关心具体的http头部或者grpc的错误状态码。

### middleware

middleware实现了装饰器模式，通过middleware你可以包装你的service或者endpoint，通常你需要构建一个middleware链来实现如日志，rate limit，负载均衡和分布式追踪。

### 缺点

1. 太过复杂 如果你要添加一个API那么你将需要做如下的工作。

    1. 声明一个interface，并定义相关的方法
    2. 实现这个interface
    3. endpoint工厂方法
    4. transport方法
    5. request encoder，request decoder， response encoder response decoder
    6. 把endpoint添加到server
    7. 把endpoint添加到client
2.  难以理解 各种分层，每一层都有特定的用处。
