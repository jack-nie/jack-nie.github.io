---
layout: post
title:  "C语言static揭秘"
date:   "2017-01-07"
keywords: ["c", "static", "syntax"]
description: "static, syntax"
category: "Learning C"
tags: ["C"]
---

在C语言中，`static`是一个保留字，它被用来控制生命周期和可见性,它可以被分为static全局变量，static局部变量和static函数。

### static 全局变量

static对于全局变量来讲会自动加上，例如

```
int a;
static int b;
void main() {

}
```
以上a和b有同一个storage class。

### static 局部变量

在局部变量中，static修饰的变量被存储静态分配的内存中，而静态分配的内存通常在编译时就被保存在数据段，动态分配的内存通常被保存在一个临时的栈中。static局部变量在编译时就被初始化。

```
#include <stdio.h>

void test() {
  int a = 1;
  static int b = 1;
  a += 1;
  b += 1;
  printf("a = %d, b = %d\n", a, b);
}

void main() {
  for (int i = 0; i < 5; i++)
  {
    test();
  }
}

```

输出

```
a = 2, b = 2
a = 2, b = 3
a = 2, b = 4
a = 2, b = 5
a = 2, b = 6
```
static局部变量的初始值为0，并且在整个生命周期内只会被初始化一次。

### static 函数

static应用与函数的作用是控制改函数的可见性，一旦被static修饰，就不能被外部文件引用。

### 总结

* static全局变量: 被static关键字修饰的全局变量只在当前文件下有效。
* static局部变量: 被static关键字修饰的局部变量是静态分配的，它再程序的整个生命周期中保持自己的内存地址，也就是说每次调用都会保持static修饰的局部变量的值。
* static函数: 被static修饰的函数只在当前文件下有效，对其它的文件不可见。
