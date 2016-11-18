---
layout: post
title:  "DNS预读取"
date:   "2016-11-18"
keywords: ["http"]
description: "http"
category: "HTTP"
tags: ["HTTP"]
---
{% include JB/setup %}
用户访问一个网页时，经常会遇到有很多域名需要解析的情况，而解析这些域名是需要时间的。
根据网络情况的好坏耗费的时间从数ms到数s不等。DNS预读取技术的出现就是为了解决这种问题的，
它使得用户再点击一个链接之前就能在服务端将域名解析出来。这项技术利用了计算机的常规DNS解析机制，
不需要额外的连接。对于需要加载的图片很多或者页面组件很多的网站，DNS预读取技术能够显著的减少页面加载的
时间。

###  浏览器端预读取控制

通常情况下，针对DNS预读取不需要进行任何动作，但是有时候用户需要关闭DNS预读取，那么可以进行如下操作;

Firefox

```
network.dns.disablePrefetch = true
```
默认情况下，启用了https的网站文档中嵌入的链接是不会进行DNS预读取的，如果想要对其进行预读取，则需要进行下面的操作:

```
network.dns.disablePrefetchFromHTTPS = false
```

### 服务端预读取控制

服务端可以通过设置`x-dns-prefetch-control`头来控制预读取的开关，也可以在页面中嵌入如下代码：

```
<meta http-equiv="x-dns-prefetch-control" content="off">

```

还可以通过link的`rel`属性来精确的控制某一特定域名的DNS预读取：

```
<link rel="dns-prefetch" href="http://www.example.com">
```

### 参考文献

- [dns-prefetching](http://dev.chromium.org/developers/design-documents/dns-prefetching)
- [Controlling DNS prefetching](https://developer.mozilla.org/zh-CN/docs/Controlling_DNS_prefetching)
