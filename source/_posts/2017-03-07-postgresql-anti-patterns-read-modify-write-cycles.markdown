---
layout: post
title:  "【翻译】PostgreSQL anti-patterns: read-modify-write cycles"
date:   "2017-03-07"
keywords: ["postgresql anti-patterns"]
description: "postgresql anti-patterns"
category: "postgresql"
tags: ["postgresql"]
---

如果你对基于sql的应用编程不是很熟悉的话，那么我强烈建议你先去读这一篇文章。

它让我想起了另一个sql编程的反模式: read-modify-write cycles。在本篇文章中，我将要解释这个常见的开发错误是什么，怎么识别以及怎么修复。

想象一下你现在需要查找一个用户的账户，如果账户余额减掉100还不是负数的话就将其保存。

下面是一种很常见的写法：

```
SELECT balance FROM accounts WHERE user_id = 1;
-- in the application, subtract 100 from balance if it's above
-- 100; and, where ? is the new balance:
UPDATE accounts SET balance = ? WHERE user_id =1;

```

对于开发者来讲代码似乎运行的很好。然而，这段代码错的很离谱，只要不同的session操作同时操作同一个用户的账户就会出现故障。

想象一下两个并发的session,每一个都从用户的账户余额中减掉100,从初始余额300开始。

| Session 1                            |   Session 2                             |
|:-------------------------------------|:----------------------------------------|
| SELECT balance FROM accounts         |                                         |
| WHERE user_id = 1; (returns 300)     |                                         |
|                                      | SELECT balance FROM accounts WHERE      |
|                                      | user_id = 1; (also returns 300)         |
| UPDATE balance SET balance = 200     |                                         |
| WHERE user_id = 1; (300 – 100 = 200) |                                         |
|                                      | UPDATE balance SET balance = 200 WHERE  |
|                                      | user_id = 1; (300 – 100 = 200)          |
|                                      |                                         |

现在余额变成了200，但是你从有300余额的账户中扣掉了200，所以有100消失了。
大部分的测试和开发是在单独的session和单独的服务器上开发的，所以除非你做非常严格的测试，否则这样的错误知道生产环境上线之前是不会被发现的，而且也非常的难调试。知道这些对你进行防御型编程是非常重要的。

### 事务不能阻止么？

我经常在stackoverflow上面看到有人问"难道事务不能阻止吗？"。不幸的是，事务虽然很伟大，但是并不是你进行简单的并发编程的装饰。唯一能够让你忽视并发问题的操作就是在事务开始之前锁住每一个你将要操作的表（你甚至需要以同样的顺序加锁以避免死锁的发生）。

加了事务之后，执行的过程和上面是一样的：

|Session1                                                              |Session 2                                                              |
|:---------------------------------------------------------------------|:----------------------------------------------------------------------|
|BEGIN;                                                                | BEGIN;                                                                |
|SELECT balance FROM accounts WHERE user_id = 1; (returns 300)         |                                                                       |
|                                                                      | SELECT balance FROM ac  counts WHERE user_id = 1; (also returns 300)  |
|UPDATE balance SET balance = 200 WHERE user_id = 1; (300 – 100 = 200) |                                                                       |
|                                                                      | UPDATE balance SET balance =  200 WHERE user_id = 1; (300 – 100 = 200)|
|COMMIT;                                                               | COMMIT;                                                               |
|                                                                      |                                                                       |

### 解决方案

幸运的是PostgreSQL有很多的工具还可以帮助你，还有一些应用程序层面的解决方案。
一些比较流行的解决方案如下:

* 使用calculated update来避免read-modify-write
* 使用行级锁`select ... for update`
* 使用序列化的事务
* 乐观并发控制，也就是使用乐观锁

### 避免read-modify-write

最好的方式就是使用sql来解决，彻底解决read-modify-write-cycle。

|Session1                                                                            | Session 2                                                                         |
|:-----------------------------------------------------------------------------------|:----------------------------------------------------------------------------------|
|UPDATE accounts SET balance = balance - 100   WHERE user_id = 1; (sets balance=200) |                                                                                   |
|                                                                                    | UPDATE accounts SET balance =  balance - 100 WHERE user_id = 1; (sets balance=100)|

上面的代码在并发的事务中也是可以工作的，因为第一个获取行锁之后，第二个就开始等待，知道第一个完成提交或者回滚。 transaction isolation文档的READ COMMITED部分有详细的解释。

这个解决方案只在不是很复杂的情形下有效，当在复杂的业务逻辑下，比如需要基于当前的账户余额来判断是否需要执行。这个操作在可用的时候是最简单和快速的。

需要注意的是非默认的SERIALIZABLE隔离也会阻止该错误，但是处理方式不同。让我们来看一下关于SERIALIZABLE的讨论。

### 行级锁

修复已经存在缺陷的应用的最简单的方式就是加上行级锁。

使用` SELECT balance FROM accounts WHERE user_id = 1 FOR UPDATE`来代替`SELECT balance FROM accounts WHERE user_id = 1`。这使用了行级锁。所有其它的试图更新该行锁或者使用`select .. for update` 或者 `select ... for share`的操作都将等待前一个事务提交或者回滚。

上面的例子中，在PostgreSQL的默认事务隔离级别中，第二条select语句直到第一条语句执行update并且commit之后才会返回。然后第一个事务将会继续，但是select将会返回200而不是300,所以会产生正确的数据。



|Session1                                                                       |      Session 2                                                                                           |
|:------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------|
|BEGIN;                                                                         |      BEGIN;                                                                                              |
|SELECT balance FROM accounts WHERE user_id = 1 FOR UPDATE; (returns 300)       |                                                                                                          |
|                                                                               |      SELECT balance FROM accounts WHERE user_id  = 1 FOR UPDATE; (gets stuck and waits for transaction 1)|
|UPDATE balance SET balance = 200 WHERE user_id = 1; (300 – 100 = 200)          |                                                                                                          |
|COMMIT;                                                                        |                                                                                                          |
|                                                                               |      (second transaction’s SELECT returns 200)                                                           |
|                                                                               |      UPDATE balance SET balance = 100 WHERE user_id = 1; (200 – 100 = 100)                               |
|                                                                               |      COMMIT                                                                                              |

PostgreSQL官方文档中关于[explicit lock]("https://www.postgresql.org/docs/current/static/explicit-locking.html")的部分有详细的解释。

需要注意的是上面的方法只有在read-modify-write在一个事务中才有用，因为锁只存在于事务的生命周期中。

### SERIALIZABLE事务

如果read-modify-write总是存在于单独的事务中，并且你使用的PostgreSQL版本大于9.1，那么你可以使用SERIALIZABLE事务来代替显式的`select ... for update`。

在这种情况下，所有的`select`和`update`语句将会正常的执行。第一个事务将会COMMIT，然后当你COMMIT第二个事务的时候，将会退出并且抛出一个`serialization error`。你将需要从头开始执行失败的事务。

|Session1                                                                |  Session 2                                                                                                                              |
|:-----------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------|
|BEGIN ISOLATION LEVEL SERIALIZABLE;                                     |  BEGIN ISOLATION LEVEL SERIALIZABLE;                                                                                                    |
|SELECT balance FROM accounts WHERE user_id = 1; (returns 300)           |                                                                                                                                         |
|                                                                        |  SELECT balance FROM accounts WHERE user_id = 1; (also returns  300)                                                                    |
|UPDATE accounts SET balance = 200 WHERE user_id = 1; (300 – 100 = 200)  |                                                                                                                                         |
|                                                                        |  UPDATE accounts SET balance = 200 WHERE user_id = 1; (gets stuck on session1’s   lock and doesn’t proceed)                             |
|COMMIT – succeeds, setting balance=200                                  |                                                                                                                                         |
|                                                                        |  (UPDATE continues, but sees that the row has been changed and aborts with a could not serialize access due to concurrent update error) |
|                                                                        |  COMMIT converted into forced ROLLBACK, leaving balance unchanged                                                                       |

SERIALIZABLE隔离级别在一个大的事务退出或者有冲突的时候将会强制应用重复很多工作。这对于在尝试使用行级锁可能会造成死锁的复杂情况下，将会很有用。

PostgreSQL官方文档中关于并发控制和事务隔离级别的描述是关于SERIALIZABLE隔离级别的最好资源。如果你还不习惯于思考并发问题，那么你将需要多读几遍，并且尝试一下文档中给出的例子。

### 乐观并发控制

[乐观并发控制]("http://en.wikipedia.org/wiki/Optimistic_concurrency_control")通常是在应用端来实现的处理并发的功能，比如hibernate一类的ORM工具。

在这个模式下，所有的表都会一个版本号或者最近更新的事件戳，并且所有的`update`语句都有一个额外的where语句来保证自该行被读之后就没有被更新。应用程序将会检查是否有被`update`影响到的行，如果没有，将会被视为一个错误并且放弃事务。

在这个demo中，需要添加一个新的列：

```
ALTER TABLE accounts ADD COLUMN version integer NOT NULL DEFAULT 1;
```

然后这个例子就变成了：

|Session1                                                                                                                                       |Session 2                                                                                                                       |
|:----------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|
|BEGIN;                                                                                                                                         |BEGIN;                                                                                                                          |
|SELECT balance, version FRO   M accounts WHERE user_id = 1; (returns 1, 300)                                                                   |                                                                                                                                |
|                                                                                                                                               |SELECT version, bala  nce FROM accounts WHERE user_id = 1; (also returns 1, 300)                                                |
|COMMIT;                                                                                                                                        |COMMIT;                                                                                                                         |
|BEGIN;                                                                                                                                         |BEGIN;                                                                                                                          |
|UPDATE accounts SET balance = 200, version = 2 WHERE  user_id = 1 AND version = 1;  (300 – 100 = 200. Succeeds, reporting 1 row changed.)      |                                                                                                                                |
|                                                                                                                                               |UPDATE accounts SET balance = 200, version = 2 WHERE user_id = 1 AND ve rsion = 1; (300 – 100 = 200). Blocks on session 1’s lock|
|COMMIT;                                                                                                                                        |                                                                                                                                |
|                                                                                                                                               |(UPDATE returns, matching zero rows because it sees version=2 in the WHERE clause)                                              |
|                                                                                                                                               |ROLLBACK; because of error detected                                                                                             |

因为实现这个功能非常的繁琐，所以乐观并发控制通常是通过ORM或者查询构造器来使用。
和SERIALIZABLE隔离级别不同的是，它在自动提交模式或者语句在不同的事务中也是有效的。基于这个原因，这通常在给用户很长思考时间的web应用或者用户在交易中途就退出。因为它不需要可能造成性能问题的长时事务。

如果你使用出发起来出发乐观并发控制规则为，乐观并发控制可以和传统的基于锁的实现协同使用。否则，它通常和其它的显示并发控制方式单独使用。

### 该选哪一个？

最合适的选择取决于你在干什么，你的确定的需求，书写retry循环来处理失败的事务的难度等。

没有一个适用于所有人的答案，如果有，那么将会只有一种方案，而不是多种。

但是有一个一定是错误的方式，那就是忽略并发问题，并且期待数据库能够正确的处理。

### 参考文献

- [PostgreSQL anti-patterns]("https://blog.2ndquadrant.com/postgresql-anti-patterns-read-modify-write-cycles/")
