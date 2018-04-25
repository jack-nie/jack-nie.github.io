---
layout: post
title:  "使用sequel需要注意的地方"
date:   "2016-09-16"
keywords: ["ruby", "sequel"]
description: "sequel"
category: "Ruby"
tags: ["Ruby"]
---

使用[sequel](https://github.com/jeremyevans/sequel)有一段时间了，愈发的觉得这是一个优秀的框架。但是作为一个从ActiveRecord资深使用者切换过来的开发人员，还是多少带有一些习惯性思维在里面，因而容易不恰当的使用一些Sequel的特性。这篇文章的主要目的是记录一些需要注意的地方。

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

### 关闭数据库连接

数据库连接，本质上，就是一个长期的 TCP 连接。通过系统的命令行，可以观察连接。
比如在 Linux 使用 ss -anp | grep [端口号]，可以查看指定端口的 TCP 连接数量。Mac 下面的命令是：lsof -i -n -P | grep TCP | grep [端口号]。

对于数据库来说，维护一个应用的 TCP 连接，是需要消耗固定内存，Postgresql 中维护一个连接，耗费的内存是 2 MB。
在数据库中，可以通过 SQL 语句，查询连接的状态，以及上一条执行的 SQL 语句，在指定数据库中执行：SELECT * FROM pg_stat_activity;

对于数据库，维护大量的数据库连接，耗费的内存高。

Sequel可以根据设置的时间，关闭数据库连接。在设置数据库连接后设置：

```
DB.extension(:connection_expiration)
DB.pool.connection_expiration_timeout = 10
```

### Read-Only Slaves/Writable Master

需要注意的是事物和存储过程均不能运行在slave机器上，若需要使用，则需要指定master机器。如果想指定某一条查询使用特定的数据库择可以采用这种方式：

```
DB[:users].server(:default).all
```

数据库使用单个主库单个丛库的配置如下：

```
DB=Sequel.connect('postgres://master_server/database',    :servers=>{:read_only=>{:host=>'slave_server'}})
```
这条配置会让select查询语句运行在slave机器上，其它的查询运行在master机器上。

### Transaction

由于sequel的设计理念，即使是在transaction块中，select语句和非select语句会在不同的shard中执行。要保证transaction块中的所有语句都在同一个数据库中执行则需要显示的指定server。
或者更方便的复写DB.transaction, 主要是利用server_block这个extendion。

```
DB.extensions :server_block
DB.transaction  do
  DB.with_server(:default) do
    -----
  end
end

```

###  参考文献

