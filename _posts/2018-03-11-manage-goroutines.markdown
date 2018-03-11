---
layout: post
title:  "Golang中goroutine的管理 "
date:   "2018-03-11"
keywords: ["golang", "goroutine", "management"]
description: "goroutines管理"
category: "Go"
tags: ["Go"]
---
{% include JB/setup %}

最近在研究nsq的源码，首先看了下官方的一些文档，看到了一些对于goroutine管理方面的技巧，这里记录一下，备查。

使用go关键字能够很容易的创建一个goroutine，但是对于goroutine的管理和清理却并不那么容易。goroutine带来的死锁问题也是不可回避的，这通常是由于顺序处理不当造成的。
goroutine也能造成内存泄漏，这在长时间运行的系统中会造成很大的影响。

### sync.WaitGroup

waitGroup可以用来记录goroutine的数量，并且提供一种方式来等待所有的goroutine退出, nsq包装了waitGroup，方便后续的调用。

```
type WaitGroupWrapper struct {
	sync.WaitGroup
}

func (w *WaitGroupWrapper) Wrap(cb func()) {
	w.Add(1)
	go func() {
		cb()
		w.Done()
	}()
}

// can be used as follows:
wg := WaitGroupWrapper{}
wg.Wrap(func() { n.idPump() })
...
wg.Wait()
```

### Exit Signal

一种最简单的方式就是给每个goroutine传递一个chan，当关闭这个chan的时候，所有的goroutine都能够收到通知，这样就能知道什么时候该退出了。

```
func work() {
    exitChan := make(chan int)
    go task1(exitChan)
    go task2(exitChan)
    time.Sleep(5 * time.Second)
    close(exitChan)
}
func task1(exitChan chan int) {
    <-exitChan
    log.Printf("task1 exiting")
}

func task2(exitChan chan int) {
    <-exitChan
    log.Printf("task2 exiting")
}
```

### 退出时的同步

实现一个可靠的，无死锁的，所有传递中的消息的退出顺序是相当困难的, 下面是一些TIPS：

* 理想的情况是负责发送数据到 go-chan 的 goroutine 也应负责关闭它。
* 如果 message 不能丢失，确保相关的 go-chan 被清空（尤其是无缓冲的！），以保证发送者可以继续发送数据。
* 另外，如果消息是不重要的，发送给一个单一的 go-chan 应转换为一个 select 附加一个退出信号（如上所述），以避免阻塞。
* 一般的顺序应该是

    * 停止接受新的连接
    * 发送退出信号给 child goroutines
    * 利用 WaitGroup 的 wait 等待 goroutine 退出
    * 恢复缓冲数据
    * 刷新所有东西到硬盘
