---
layout: post
title:  "使用sequel需要注意的地方"
date:   "2016-09-16"
keywords: ["ruby", "sequel"]
description: "sequel"
category: "Ruby"
tags: ["Ruby"]
---
{% include JB/setup %}

使用[sequel](https://github.com/jeremyevans/sequel)有小半年的时间了，愈发的觉得这是一个优秀的框架。但是作为一个从ActiveRecord资深使用者切换过来的开发人员，还是多少带有一些习惯性思维在里面，因而容易不恰当的使用一些Sequel的特性。这片文章的主要目的是记录一些需要注意的地方。

#### 调用first时，采用不同的方法返回的结果的类型并不完全一致

```
Post.where(:title => 'Hello world').first.class #=> Post
Post.eager(:author).first.class #=> Post
Post.eager_graph(:author).first.class #=> Hash
```

#### 没有pluck方法，可用select_map

```
Post.select_map([:id, :title])
```

#### 没有attribute_names方法

```
album.keys.map{|x| x.to_s}.sort
```

#### 打印sql语句

```
items.where{price * 2 < 50}.sql
```

###  参考文献

