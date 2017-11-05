---
layout: post
title:  "Golang中的slice详解及常见坑总结"
date:   "2017-11-05"
keywords: ["slice golang"]
description: "slice详解 "
category: "Go"
tags: ["Go"]
---
{% include JB/setup %}

Slice在golang中属于常见的数据结构，本文将从各个方面对其进行详细的讲解，文末给出一些在开发过程中容易遇到的坑。

### 1. 常见的操作 ###

- **切片**
 
```   
    s := []int{1,2,3,4,5,6}
	l := s[2:5]
	fmt.Println("sl1:", l)
	l = s[:5]
	fmt.Println("sl2:", l)
	l = s[2:]
	fmt.Println("sl3:", l)
    l = s[:2:3]
    fmt.Println("sl3:", l)

```

- **append**


```
   根据[append](https://golang.org/pkg/builtin/#append "append")的官方定义，该方法的作用是用于将元素追加到slice的末尾。如果slice有足够的容量，那么将直接追加, 返回新的slice。如果没有足够的容量，将重新分配一片用于存放数组的内存，返回新的slice。所以官方建议的做法是将返回的slice重新赋值给原来的变量。
     
    slice = append(slice, elem1, elem2)
    slice = append(slice, anotherSlice...)
   
   可以通过如下的程序来验证：

	package main
	
	import "fmt"
	
	func main() {
		s := []int{1}
		fmt.Printf("addr: %p, cap: %d, sliceAddr: %p\n", &s[0], cap(s), &s)
		s = append(s, 2)
		fmt.Printf("addr: %p, cap: %d, sliceAddr: %p \n", &s[0], cap(s), &s)
		s = append(s, 3)
		fmt.Printf("addr: %p, cap: %d, sliceAddr: %p\n", &s[0], cap(s), &s)
		x := append(s, 4)
		fmt.Printf("Saddr: %p, Xaddr: %p, cap: %d, lenS: %d, sliceAddr: %p\n", &s[0], &x[0], cap(s), len(s), &x)
		y := append(s, 5)
		fmt.Printf("Saddr: %p, Yaddr: %p, cap: %d, lenS: %d, sliceAddr: %p\n", &s[0], &y[0], cap(s), len(s), &y)
	}

```
 
   运行得到的结果如下：

 
```    
	addr: 0xc0420401d0, cap: 1, sliceAddr: 0xc042046400
	addr: 0xc042040200, cap: 2, sliceAddr: 0xc042046400 
	addr: 0xc042046460, cap: 4, sliceAddr: 0xc042046400
	Saddr: 0xc042046460, Xaddr: 0xc042046460, cap: 4, lenS: 3, sliceAddr: 0xc042046480
	Saddr: 0xc042046460, Yaddr: 0xc042046460, cap: 4, lenS: 3, sliceAddr: 0xc0420464a0
 
```

   通过运行结果可以得到如下结论：
    
    
   1. 创建初始切片时，容量为1
   2. 追加2之后，因为超出了原有数组的容量，所以扩容为原来的2倍，容量变为2，分配一个新的数组，可以看到地址已经变了。
   3. 追加3之后，因为超出了原有数组的容量，所以扩容为原来的2倍，容量变为4，分配一个新的数组，可以看到地址已经变了。
   4. 追加4之后，因为容量足够，所以直接追加到原有数组的末尾，不重新分配数组，地址不变。
   5. 追加5之后，因为容量足够，所以直接追加到原有数组的末尾，不重新分配数组，地址不变。
   
   这里比较难理解的是4和5，append永远返回新的slice，并不会改变原有切片的值。4,5两步中，s的长度始终为3。
   另外的一个比较困惑的点是sliceAddr的值，变量s所指向的slice的地址始终没有发生变化。实际上&s指向的是slice这个结构体的地址。



### 2. Slice内部实现 ###

    
slice是对一个数组片段的描述，它包含一个指向数组的指针，该数组的长度以及容量。


```
	type Slice struct {
      ptr *Elem
      len int
      cap int
    }

```

对数组或者slice进行切片操作并不会进行复制操作，他会创建一个指向原始数组的新的slice，这就使得操作slice像操作数组下标一样有效率。所以修改slice中的元素，或者对slice进行重新切片，会修改原始切片的值。

扩充slice时不能够超过其容量，如果超过了，将会引发一个panic。要增加slice的容量，必须创建一个新的slice，并将其值复回原来的slice

```
	t := make([]byte, len(s), (cap(s)+1)*2) // +1 in case cap(s) == 0
	for i := range s {
	   t[i] = s[i]
	}
	s = t

```

当然也可以使用copy函数简化操作。


```
	t := make([]byte, len(s), (cap(s)+1)*2)
	copy(t, s)
	s = t

```


### 参考文献

1. [Go Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals)
2. [golang的append()为什么不会影响slice的地址？](https://www.zhihu.com/question/59659980)
3. [Arrays, slices (and strings): The mechanics of 'append'](https://blog.golang.org/slices)