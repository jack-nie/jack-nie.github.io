---
layout: post
title:  "Ruby中的GIL初探"
date:   2015-05-05
keywords: ["ruby"]
description: "Ruby GIL"
category: "Ruby"
tags: ["Ruby"]
---
许多脚本语言都都使用GIL来简化其解释器的内部设计，那么GIL到底是什么呢？全局解释锁（GIL），有时候也叫做全局VM锁（GVL）。每一个Ruby线程需要在运行之前请求GIL。Ruby的GIL是个虚拟机层面的互斥机制即一个线程只有在活动状态下才能执行Ruby代码。这个很有必要，因为C扩展以及很多Ruby组件并不是线程安全的（包括集合和哈希表！）持有GIL,我们便能确保正在执行的Ruby代码是同时刻唯一的运行线程；这就避免了并发问题。

Ruby的线程是 Native 线程，线程的调度是 OS 来实现的。但是由于 GIL 的存在，同一时间只有获取了这个锁的线程在跑， 也限制了有GIL的语言利用多核的能力。有 GIL 的限制并不代表你不能进行一些异步（通常意义上的异步）的操作。因为 GIL 不会被某个线程一直持有，所以其他的线程都有运行的机会，只是利用不上多核的能力 。就像以前，我们用单核 CPU 跑多线程程序类似。举个例子来说，如果你有8个线程运行在一台8核的机器上，在一个时间段内只有一个线程和一个cpu核是活动的，GIL主要是为了防止因竞态条件的发生而导致的数据完整性问题。GIL的关键所在就是确保两个线程不会同时查询和修改内存。两个线程可以同时查看内存，但如果过程中一个内存修改了内存另一个线程就需要等待下个操作数。这对于原子性、一致性的事物操作很必要：操作数获取的内存视图应该是一致的否则就会出现回滚。

参考资料：

- [Ruby 的 GIL 和事务性内存](http://www.oschina.net/translate/rubys-gil-and-transactional-memory "Ruby 的 GIL 和事务性内存")
- [Ruby 2.0 新功能演示](https://ruby-china.org/topics/6494 "Ruby 2.0 新功能演示")
- [Eliminating Global Interpreter Locks in Ruby](http://researcher.watson.ibm.com/researcher/files/jp-ODAIRA/PPoPP2014_RubyGILHTM.pdf "Eliminating Global Interpreter Locks in Ruby")

深入阅读：

- [As GIL in Ruby worked. Part 1](http://sysmagazine.com/posts/189320/ "As GIL in Ruby worked. Part 1")
- [As GIL in Ruby worked. Part 2](http://sysmagazine.com/posts/189486/ "As GIL in Ruby worked. Part 2")
- [Nobody understands the GIL](http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil "Nobody understands the GIL")
- [Nobody understands the GIL Part 2: Implementation](http://www.jstorimer.com/blogs/workingwithcode/8100871-nobody-understands-the-gil-part-2-implementation "Nobody understands the GIL Part 2: Implementation")
