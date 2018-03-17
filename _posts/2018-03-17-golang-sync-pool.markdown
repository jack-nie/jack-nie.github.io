---
layout: post
title:  "Golang中sync.Pool详解"
date:   "2018-03-17"
keywords: ["golang", "sync.Pool", "对象重用", "临时对象池"]
description: "golang sync.Pool详解"
category: "Go"
tags: ["Go"]
---
{% include JB/setup %}


我们通常用golang来构建高并发场景下的应用，但是由于golang内建的GC机制会影响应用的性能，为了减少GC，golang提供了对象重用的机制，也就是sync.Pool对象池。
sync.Pool是可伸缩的，并发安全的。其大小仅受限于内存的大小，可以被看作是一个存放可重用对象的值的容器。
设计的目的是存放已经分配的但是暂时不用的对象，在需要用到的时候直接从pool中取。

任何存放区其中的值可以在任何时候被删除而不通知，在高负载下可以动态的扩容，在不活跃时对象池会收缩。

sync.Pool首先声明了两个结构体

```
// Local per-P Pool appendix.
type poolLocalInternal struct {
	private interface{}   // Can be used only by the respective P.
	shared  []interface{} // Can be used by any P.
	Mutex                 // Protects shared.
}

type poolLocal struct {
	poolLocalInternal

	// Prevents false sharing on widespread platforms with
	// 128 mod (cache line size) = 0 .
	pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte
}
```

为了使得在多个goroutine中高效的使用goroutine，sync.Pool为每个P(对应CPU)都分配一个本地池，当执行Get或者Put操作的时候，会先将goroutine和某个P的子池关联，再对该子池进行操作。
每个P的子池分为私有对象和共享列表对象，私有对象只能被特定的P访问，共享列表对象可以被任何P访问。因为同一时刻一个P只能执行一个goroutine，所以无需加锁，但是对共享列表对象进行操作时，因为可能有多个goroutine同时操作，所以需要加锁。

值得注意的是poolLocal结构体中有个pad成员，目的是为了防止false sharing。cache使用中常见的一个问题是false sharing。当不同的线程同时读写同一cache line上不同数据时就可能发生false sharing。false sharing会导致多核处理器上严重的系统性能下降。具体的可以参考[伪共享(False Sharing)](http://ifeve.com/falsesharing/ "http://ifeve.com/falsesharing/")。

类型sync.Pool有两个公开的方法，一个是Get，一个是Put, 我们先来看一下Put的源码。

```
// Put adds x to the pool.
func (p *Pool) Put(x interface{}) {
	if x == nil {
		return
	}
	if race.Enabled {
		if fastrand()%4 == 0 {
			// Randomly drop x on floor.
			return
		}
		race.ReleaseMerge(poolRaceAddr(x))
		race.Disable()
	}
	l := p.pin()
	if l.private == nil {
		l.private = x
		x = nil
	}
	runtime_procUnpin()
	if x != nil {
		l.Lock()
		l.shared = append(l.shared, x)
		l.Unlock()
	}
	if race.Enabled {
		race.Enable()
	}
}

```


1. 如果放入的值为空，直接return.
2. 检查当前goroutine的是否设置对象池私有值，如果没有则将x赋值给其私有成员，并将x设置为nil。
3. 如果当前goroutine私有值已经被设置，那么将该值追加到共享列表。

```
func (p *Pool) Get() interface{} {
	if race.Enabled {
		race.Disable()
	}
	l := p.pin()
	x := l.private
	l.private = nil
	runtime_procUnpin()
	if x == nil {
		l.Lock()
		last := len(l.shared) - 1
		if last >= 0 {
			x = l.shared[last]
			l.shared = l.shared[:last]
		}
		l.Unlock()
		if x == nil {
			x = p.getSlow()
		}
	}
	if race.Enabled {
		race.Enable()
		if x != nil {
			race.Acquire(poolRaceAddr(x))
		}
	}
	if x == nil && p.New != nil {
		x = p.New()
	}
	return x
}
```

1. 尝试从本地P对应的那个本地池中获取一个对象值, 并从本地池冲删除该值。
2. 如果获取失败，那么从共享池中获取, 并从共享队列中删除该值。
3. 如果获取失败，那么从其他P的共享池中偷一个过来，并删除共享池中的该值(p.getSlow())。
4. 如果仍然失败，那么直接通过New()分配一个返回值，注意这个分配的值不会被放入池中。New()返回用户注册的New函数的值，如果用户未注册New，那么返回nil。

最后我们来看一下init函数。

```
func init() {
    runtime_registerPoolCleanup(poolCleanup)
}
```

可以看到在init的时候注册了一个PoolCleanup函数，他会清除掉sync.Pool中的所有的缓存的对象，这个注册函数会在每次GC的时候运行，所以sync.Pool中的值只在两次GC中间的时段有效。

通过以上的解读，我们可以看到，Get方法并不会对获取到的对象值做任何的保证，因为放入本地池中的值有可能会在任何时候被删除，但是不通知调用者。放入共享池中的值有可能被其他的goroutine偷走。
所以对象池比较适合用来存储一些临时切状态无关的数据，但是不适合用来存储数据库连接的实例，因为存入对象池重的值有可能会在垃圾回收时被删除掉，这违反了数据库连接池建立的初衷。

参考资料：

- [go语言的官方包sync.Pool的实现原理和适用场景](http://blog.csdn.net/yongjian_lian/article/details/42058893 "go语言的官方包sync.Pool的实现原理和适用场景")
- [sync.Pool源码](https://golang.org/src/sync/pool.go "sync.Pool源码")
