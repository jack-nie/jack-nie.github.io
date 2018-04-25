---
layout: post
title:  "Ruby 进程fork初探"
date:   "2016-12-31"
keywords: ["Ruby"]
description: "Ruby, fork"
category: "Ruby"
tags: ["Ruby"]
---

元旦放假期间，又捡起CSAPP看了下，主要是看异常处理这一段，感觉之前看的东西忘了不少，所以翻译一段关于Ruby的进程Fork的文章加深理解。(主要基于[Forking Ruby Processes or How to fork Ruby](http://www.tweetegy.com/2012/04/forking-ruby-processes-or-how-to-fork-ruby/)，但是不完全翻译，某些地方进行了扩展)

Forking是一个进程复制的UNIX术语，意味这从父进程复制出一个子进程出来，他们有相同的地址空间，相同的本地变量值，相同的堆，相同的全局变量值以及相同的代码。
在很多情况下，他们共享内存，直到其中的一个进程对内存进行了修改，这被称作CoW（Copy On Write）。因为子进程和父进程都是独立的进程，所以他们都有自己独立的私有地址空间，他们对其中的变量所做的更改都是独立的，不会反应到另一个进程的存储器中。

### 简单例子
我们知道了UNIX术语的含义，那么我们如何在Ruby中实现呢？如何使用Ruby来fork一个进程呢？

```
 puts "This is the first line before the fork (pid #{Process.pid})"
 puts fork
 puts "This is the second line after the fork (pid #{Process.pid})"
```
output:

```
 This the first line before the fork (pid 2284)
 2285
 This is the second line after the fork (pid 2284)

 This is the second line after the fork (pid 2285)
```
让我们来看看发生了什么，首先第一行的输出是非常清晰的，那么第二行又是什么鬼？这是fork函数返回的PID值，当然是子进程的PID。接下来让我们来看一下接下来的两行：一行是父进程执行的结果，另一行是子进程执行的结果。

### Blocks

开发者想执行一个独立的进程通常的做法是传递一个代码块给fork函数，就像下面这样：

```
 puts "You can also put forked code in a block pid: #{Process.pid}"
 fork do
   puts "Hello from fork pid: #{Process.pid}"
 end
 puts "The parent process just skips over it: #{Process.pid}"

```

上面的代码输出三行结果，在我的机子上的运行结果是这样的：

```
you can also put forked code in a block pid: 3465
The parent process just skips over it: 3465
Hello from fork pid: 3466
```

注意代码块内的PID和外面的PID是不一样的，因为代码块内的代码是通过fork函数派生出来的进程执行的。

### 多核CPU测试

让我们来快速的验证一下fork在多核环境下带来的一个主要的好处。让我们写一个ruby程序来呈现这一优势。

```
  def cpu_intensive_process
    puts "Pid: #{Process.pid}"
    x = 0
    10000000.times do |i|
      x = i + x
    end
  end

  fork
  cpu_intensive_process
```
当上面的代码运行时，将会充分的利用两个CPU核，如果机器有多余两个CPU核心，那么只有两个CPU得到充分的利用。然后，额外的fork调用将会创建更多的进程，知道占满你的CPU核心数。

### 内存测试

那么内存分配的情况又是怎么样的呢？很明显ruby1.9.3并没有CoW的特性，也就是会所每一个进程的内存分配将会是一样的。下面的程序能够一定程度上说明这一点。

```
  hash = Hash.new #load up the memory a little

  1000000.times do |i|
    hash[i] = "foo"
  end

  puts "Hash contains #{hash.keys.count} keys"

  def show_memory_usage(whoami)
   pid = Process.pid

   mem = `pmap #{pid}`

   puts "Memory usage for #{whoami} pid: #{pid} is: #{mem.lines.to_a.last}"

   sleep #keep the process alive
 end

 puts "Now lets fork this process and see what memory is allocated to the child"

 puts "Before..."

 if fork
   show_memory_usage("parent")
 else

   puts "After..."

   1000000.times do |i| #change the values in the child memory allocation
     hash[i] = "bar"
   end

   show_memory_usage("child")
 end
```

下面是这个测试的输出：

```

Hash contains 1000000 keys
Now lets fork this process and see what memory is allocated to the child
Before…
After…
Memory usage for parent pid: 10291 is: total 62592K
Memory usage for child pid: 10293 is: total 73164K
```

### 孤儿进程

最后让我们来测试一下孤儿进程并且看一下它们的表现，运行以下小程序看一下：

```
 fork do
   5.times do
     sleep 1
     puts "I'm an orphan!"
   end
 end
 abort "Parent process died..."
```

运行的结果像下面这样：

```
Parent process died...
😈  : ~/work I'm an orphan!
I'm an orphan!
I'm an orphan!
I'm an orphan!
I'm an orphan!

```
这里又发生了什么？父进程运行完毕所以在终端输出。但是我被5个子进程(孤儿进程)打断了，因为他们顺序的隔一段时间就执行代码！

### 参考文献

- [Forking Ruby Processes or How to fork Ruby](http://www.tweetegy.com/2012/04/forking-ruby-processes-or-how-to-fork-ruby/)
