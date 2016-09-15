---
layout: post
title:  "【翻译】Ruby全局解释锁"
date:   2015-05-10
keywords: ["Ruby"]
description: "Ruby全局解释锁"
category: "Ruby"
tags: ["Ruby"]
---
{% include JB/setup %}

本文中的所有代码于2012年2月使用Ruby 1.9.3执行，Ruby MRI指Ruby的C实现。

Ruby MRI全局解释锁（GIL）经常被人误解。程序员听到"Global"或者"lock"会假设编写的并行代码不会以任何的并行方式执行。但是我希望这GIL最终将被移除(jRuby and Rubinius Hydra)。GIL并不是所有的都是糟糕，Ruby MRI的线程状态只能改善。

我之所以说不那么糟糕是因为GIL仅仅应用于Ruby操作，这意味着仅仅应用于Ruby所做的工作。这是有意义的，因为它的存在是为了保护的Ruby虚拟机的完整性。

但是除了Ruby操作，还有其他的什么工作呢？在一个典型的Web应用中，当你查询数据库、磁盘或者Memcached，Ruby并没有参与其中，这意味着你并没有调用GIL。让我们来看看这个简单的例子。

### I/O Operations

	{% highlight ruby %}
	Ruby code：
	require 'benchmark'
	require 'mysql2'
	
	x = Mysql2::Client.new
	y = Mysql2::Client.new
	
	Benchmark.bm do |b|
	  b.report('w/o') do
	    x.query("SELECT SLEEP(1)")
	    y.query("SELECT SLEEP(1)")
	  end
	
	  b.report('with') do
	    a = Thread.new{ x.query("SELECT SLEEP(1)") }
	    b = Thread.new{ y.query("SELECT SLEEP(1)") }
	    a.join
	    b.join
	  end
	end
	
	       user     system      total        real
	w/o  0.000000   0.000000   0.000000 (  2.002874)
	with  0.000000   0.000000   0.000000 (  1.001658)
    {% endhighlight %}

在这个例子中，Ruby将SQL查询传递给一个等待数据库响应的C程序。因为GIL并没有进行任何工作，所以在这段时间内，Ruby释放了GIL。查询数据库是一个I/O操作，GIL并不影响I/O操作。这意味着Ruby解释器在同一时间内能够执行多个线程。为了理解这些区别，让我们来看下一个例子。

### Ruby Operations
    
    {% highlight ruby %}
	Ruby code：
	require 'benchmark'
	
	Benchmark.bm do |x|
	  x.report('w/o') do
	    10_000_000.times{ 2+2 }
	  end
	
	  x.report('with') do
	    a = Thread.new{ 5_000_000.times{ 2+2 } }
	    b = Thread.new{ 5_000_000.times{ 2+2 } }
	    a.join
	    b.join
	  end
	end
	
	       user     system      total        real
	w/o  2.300000   0.000000   2.300000 (  2.312023)
	with  2.330000   0.000000   2.330000 (  2.338784)
    {% endhighlight %}

在这个例子中，Ruby需要执行运算并且修改数据，因为GIL被激活，所以多线程并没有带来任何好处。实际上，尝试将计算并行化将会产生两个线程的开销。

### 总结

Ruby GIL仅仅应用于非I/O操作，当移除GIL时，最后一个例子将能够充分的利用原生线程，Ruby的线程并不是一个不存在的状态。Ruby的线程是能够工作的，仅仅是还没有达到其他语言的级别。

希望本文解释清楚了GIL和线程，在这一点上，事件驱动的Ruby，尽管存在GIL，线程和纤程仍旧为Ruby提供了丰富的并发功能。我相信在将来这将会得到改善-特别是线程。Github Gist中的代码同时包含了使用Ruby的sleep方法的另外一个例子。

原文：

- [The Ruby Global Interpreter Lock](http://ablogaboutcode.com/2012/02/06/the-ruby-global-interpreter-lock/ "The Ruby Global Interpreter Lock")
