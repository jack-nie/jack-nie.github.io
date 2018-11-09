---
layout: post
title:  "golang中context包详解"
date:   "2018-04-14"
keywords: ["golang", "context"]
description: "context包详解"
category: "Go"
tags: ["Go"]
---

在开发go web服务器的时候，通常一个request是在特定的goroutine中完成，请求处理程序经常启动额外的goroutine来访问数据库或者RPC等后端服务，处理请求的一系列goroutine通常需要获取终端用户的标识，授权令牌以及确定请求什么时候终止。当请求终止的时候，这一系列的goroutine都应该被通知到并退出，以便系统能够回收资源。

context包就是为了在goroutine之间传递信息用的，能够使得在一系列的处理请求的goroutine中很方便的传递信息。


对服务器的传入请求应创建一个上下文，对服务器的传出调用应接受上下文。它们之间的函数调用链必须传播上下文，可以用使用WithCancel，WithDeadline，WithTimeout或WithValue创建的派生上下文替换它。当上下文被取消时，从它派生的所有上下文也被取消。

WithCancel，WithDeadline和WithTimeout函数采用Context（父级）并返回派生的Context（子级）和CancelFunc。调用CancelFunc将取消子对象及其子对象，删除父对子对象的引用，并停止任何关联的定时器。未能调用CancelFunc会泄漏子项及其子项，直至父项被取消或计时器激发。 go vet工具检查在所有控制流路径上使用CancelFuncs。

使用上下文的程序应该遵循这些规则来保持包之间的接口一致，并使静态分析工具能够检查上下文传播：

不要将上下文存储在结构类型中;相反，将一个Context明确地传递给每个需要它的函数。上下文应该是第一个参数，通常命名为ctx：

即使函数允许，也不要传递零上下文。如果您不确定要使用哪个上下文，请传递context.TODO。

使用上下文值仅适用于传输进程和API的请求范围数据，而不用于将可选参数传递给函数。

相同的上下文可以传递给在不同goroutine中运行的函数;上下文对于多个goroutine同时使用是安全的。

```
//上下文包含截止期限，取消信号和请求范围值
//跨越API边界。它的方法对于多个同时使用是安全的
// goroutines。
type Context interface {
    // 当一个Context被取消或者超时的时候返回一个关闭的通道
    Done() <-chan struct{}

    // 当Done通道关闭后，返回这个context被取消的原因
    Err() error

    // 当设置了取消时间的时候，返回什么时候被取消
    Deadline() (deadline time.Time, ok bool)

    // 返回键值对，如果没有返回nil
    Value(key interface{}) interface{}
}
```

Done方法返回一个通道，该通道作为代表上下文运行的函数的取消信号：当通道关闭时，函数应放弃其工作并返回。 Err方法返回一个错误，指出Context被取消的原因。

由于与完成通道仅接收相同的原因，上下文没有取消方法：接收取消信号的功能通常不是发送信号的功能。特别是，当父操作为子操作启动子程序时，这些子操作不应该能够取消父操作。相反，WithCancel函数（如下所述）提供了一种方法来取消新的上下文值。

上下文对于多个goroutine同时使用是安全的。代码可以将单个Context传递给任意数量的goroutine，并取消该Context来发信号通知所有这些。

Deadline方法允许函数确定他们是否应该开始工作;如果剩下的时间太少，这可能不值得。代码也可以使用最后期限来设置I / O操作的超时时间。

值允许上下文携带请求范围的数据。该数据必须安全，可供多个goroutine同时使用。

### WithCancel

```
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)

```

WithCancel返回一个携带新的Done通道的父亲的副本。当返回的cancel方法被调用或者父级context的Done通道被关闭时，返回的context的Done通道会被关闭。

取消这个context会释放与它相关的资源，因此只要在这个Context中运行的操作完成，代码就应该立即调用cancel。

###  WithDeadline

```
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
```

WithDeadline返回父上下文的副本，并将截止日期调整为不晚于d。如果父母的截止日期早于d，WithDeadline（parent，d）在语义上等同于父母。当截止日期到期，返回的取消功能被调用时，或父上下文的完成通道关闭时，返回的上下文的完成通道关闭，以先发生者为准。

取消这个上下文会释放与它相关的资源，因此只要在这个Context中运行的操作完成，代码就应该立即调用cancel。

### WithTimeout

```
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

WithTimeout返回WithDeadline（parent，time.Now（）。Add（timeout））。

取消这个上下文可以释放与它相关的资源，因此只要在这个Context中运行的操作完成，代码就应该立即调用cancel

### WithValue

```
func WithValue(parent Context, key, val interface{}) Context
```

WithValue返回父键的副本，其中与键关联的值是val。

使用上下文值仅适用于传输进程和API的请求范围数据，而不用于将可选参数传递给函数。

提供的密钥必须具有可比性，不应该是字符串类型或任何其他内置类型，以避免使用上下文的包之间发生冲突。 WithValue的用户应该为键定义他们自己的类型。为了避免在分配给接口时分配{}，上下文键通常具有具体类型struct {}。或者，导出的上下文关键字变量的静态类型应该是指针或接口。
