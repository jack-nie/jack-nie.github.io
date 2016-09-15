---
layout: post
title:  "字符串与指针"
date:   "2016-05-08"
keywords: ["c", "pointer", "string"]
description: "string, pointer"
category: "Learning C"
tags: ["C"]
---
{% include JB/setup %}
字符串常量是一个字符数组，在字符串的内部表示中，字符数组以空字符`\0`结尾。所以程序可以通过检查空字符找到字符数组的结尾，因此字符串常量存储空间比双引号内的字符数大1。
字符串常量和字符数组的区别仅仅在于最后有没有`\0`。当我们访问字符串时，实际上是通过字符指针进行访问的。
    {% highlight c %}
    Code:
    #include <iostream>
    using namespace std;
    int
    main()
    {
      char c[6] = {'h', 'e', 'l', 'l', 'o', '\0'}
      char *pc = c;
      cout<<c<<endl;
      cout<<pc<<endl;
      return 0;
    }
    {% endhighlight %}

这里的`cout<<c<<endl; cout<<pc<<endl`实际上是打印出整个字符串数组，而不是该字符串数组的地址，原因是`cout`对字符串的输出做了特殊的处理，如果我们要打印出该字符串数组的首地址，应该采用`cout<<static_cast<void*>(c)<<endl; cout<<static_cast<void*>(pc)<<endl`。

#### 字符串指针举例;

    {% highlight c %}
    Code:
    #include <iostream>
    using namespace std;
    int
    main()
    {
      char buffer[10] = "ABC";
      char *pc;
      pc = "hello";
      cout<<pc<<endl;
      pc++;
      cout<<pc<<endl;
      cout<<*pc<<endl;
      pc = buffer;
      cout<<pc;
      return 0
    }
    {% endhighlight %}

我们可以直接把字符串赋给指针变量`pc="hello"`，但是不能把字符串赋值给字符数组，除了在初始化的时候。在这里，因为字符串是一个常量，所以我们只能读取该字符串的值，但是不能改变它的值。在程序运行时，所有的常量都是放在一块特殊的内存区域中，这块区域是不允许修改的。在进行`pc++`后，指针指向下一个位置，因此`pc++;cout<<pc<<endl`打印出`ello`, `cout<<*pc<<endl`打印出`e`。
