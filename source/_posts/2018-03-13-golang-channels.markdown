---
layout: post
title:  "Golang中channels详解"
date:   "2018-03-13"
keywords: ["golang", "channels", "详解"]
description: "golang channels详解"
category: "Go"
tags: ["Go"]
---

### 什么是channel

GO语言并发编程模型参考了了CSP理论，也就是所谓的通过传递消息来共享内容，而不是共享内存。你可以把channel想像成一根水管，发送者将消息从水管的一端发进去，接受者从水管的另一端取出来。
channel是有类型的，一端定义了channel的类型，那么发送者将只能发送特定类型的数据到该channel。
例如定义`chan T`，那么该chan将只能接受类型为T的数据。

### 发送和接收

channel的发送和读取的语法如下：

```
data := <- a //从channel中读取数据
a <- data //将数据发送到channel
```

需要注意的是，声明一个channel变量并不能直接使用，因为该变量的值是nil，无论是向nil的channel发送数据还是从nil的channel接收数据都将引发一个panic。

channel的读取和发送都是阻塞的，除非有另外的goroutine向channel的一端发送数据，或者从channel中读取数据。channel的这种特性能够帮助开发者不显示的使用锁来完成同步操作。

### bufferd channel VS unbuffers channel

channel可以有两种初始化的方式，分别是初始化成带缓冲的和不带缓冲的。下面将介绍这两种方式的差异。

```
var a chan int
var b chan int
a = make(chan int) // 不带缓冲的chan
b = make(chan int, 3) //带缓冲的chan
```

对于带缓冲的channel，只要channel未满，我们可以一直往里面写数据，但是无缓冲的channel则不行，下面的一段程序可以验证上面的结论。

```
package main

import "fmt"

func main() {
  a := make(chan int)
  b := make(chan int, 3)
  b <- 1
  b <- 2
  b <- 3
  fmt.Println(<-b)
  fmt.Println(<-b)
  fmt.Println(<-b)
  go func() {
    fmt.Println(<-a)
  }
  a <- 1
}
```

### channel with select

可以通过select模式来实现多个chan的收发操作，还可以定义一个timer来实现超时的逻辑处理。

```
select {
case x := <- a:
  //do something
case y, ok := <- b:
  //do something
case c <- z:
  //do something
case <- time.After(....):
  //do something
default:
  //do something
}
```

### for..range chan 和 close

channel也可以像slice一样通过for range来遍历，只要没关闭channel，for语句将一直阻塞。

```
for c, ok := range b {
  ....
  //do something
}
```

上面代码中的ok可以用来标示channel是否已经关闭。我们偶尔也会见到下面的用法。

```
for range b {
  ....
  //do something
}
```

这里只要b不关闭，并且有数据发送到b，就可以一直执行for loop中的逻辑。
当一个带缓冲的channel被关闭后，我们仍然可以接收已经发送到该channel的数据直到完全取出所有的数据 ，
取完所有的数据之后再进行读取，将会得到声明的channel的类型的零值。
关闭一个不带缓冲的channel后，也可以从该channel读取数据，读取的是声明的channel的类型的零值。
无论是带缓冲的还是不带缓冲的channel，一旦关闭之后，就不能再向其发送数据了。
我们可以通过：

```
if c, ok := <-b; ok {
  //do something
}

```
来判断channel是否关闭。

### directional channel VS undirectional channel

通常我们使用时不会声明一个带方向的chan，因为只能接收或者只能发送的channel并没有实际的用处。
带方向的chan一般是用在方法的签名中，更多的时候是做一个约定，在代码review的时候可以更容易的看出该chan是用来读取还是用来接收。
