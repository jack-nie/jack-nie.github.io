---
layout: post
title:  "Postgres's database VS schema"
date:   "2017-01-14"
keywords: ["Postgresql"]
description: "postgresql, database, schema"
category: "Postgresql"
tags: ["Postgresql"]
---
{% include JB/setup %}

一个PostgreSQL数据库集簇中包含一个或更多命名的数据库。用户和用户组被整个集簇共享，但没有其他数据在数据库之间共享。
任何给定客户端连接只能访问在连接中指定的数据库中的数据。一个集簇的用户并不必拥有访问集簇中每一个数据库的权限。
用户名的共享意味着不可能在同一个集簇中出现重名的不同用户，例如两个数据库中都有叫joe的用户。但系统可以被配置为只允许joe访问某些数据库。

实际上可以认为一个模式包含一组表，而一个数据库包含一组模式。

### 为什么需要模式

和数据库不同，模式并不是被严格的隔离，如果用户被赋予足够的权限，他就能够访问数据中所有模式内的对象。

首先，模式允许多个用户使用一个数据库并且不会被干扰。其次，模式可以将数据库对象组织成逻辑组以便更容易管理。
最后，第三方应用的对象可以放在独立的模式中，这样它们就不会与其他对象的名称发生冲突。

一个例子,你有一个web应用来收集用户的信息并且需要将这些信息存储到数据库中，这些数据被称作交易数据。
还有一些数据使用来做展示用的，这类数据被称作配置性数据。所以我们需要创建两个模式一个用来存储交易数据，
一个用来存储配置数据，通过这种配置开发者就无法获取交易数据，从而保证数据的安全。

如果需要创建一个SAAS的应用，那么可以针对每一个用户创建一个模式。但是在用户数过多（>50）的情况下，可能会造成性能问题。


### 参考文献

- [PostgreSQL 9.5.3 中文手册](http://www.postgres.cn/docs/9.5/ddl-schemas.html)
- [What is the key difference between a schema and a database with respect to MySQL, Oracle and other databases?](https://www.quora.com/What-is-the-key-difference-between-a-schema-and-a-database-with-respect-to-MySQL-Oracle-and-other-databases)
