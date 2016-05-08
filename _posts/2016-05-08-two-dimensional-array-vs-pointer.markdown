---
layout: post
title:  "指向二维数组的指针"
date:   "2016-05-08"
keywords: ["c", "pointer"]
description: "array, two dimensional, pointer"
category: "Learning C"
tags: ["C"]
---
{% include JB/setup %}
首先我们通过一道例题来认识一下指向二维数组的指针。
    {% highlight c %}
    Code:
    #include <iostream>
    using namespace std;
    int
    main()
    {
      int a[3][4] = {1,3,5,7,9,11,13,15,17,19,21,23};
      int *p;
      for(p=&a[0][0]; p<&a[0][0]+12;p++)
      {
        cout<<p<<" "<<*p<<endl;
      }
      return 0;
    }
    {% endhighlight %}

我们把二维数组存储在内存里面的时候，实际上是把二维数组拉成了一条直线存储在内存里面的。数组里面的每个元素都是连续的存储在内存中的，二维数组也一样，所以我们通过对指针进行＋＋运算的时候，指针会指向下一个元素。

#### 输入i,j;输出a[i][j];

    {% highlight c %}
    Code:
    #include <iostream>
    using namespace std;
    int
    main()
    {
      int a[3][4] = {1,3,5,7,9,11,13,15,17,19,21,23};
      int (*p)[4],i,j;
      p = a;
      cin>>i>>j;
      cout<<setw(4)<<*(*(p+i)+j);
      return 0;
    }
    {% endhighlight %}

首先定义一个二维数组a[3][4]，我们从p=a开始，a相当于指向a[3][4]的第一个元素的指针；所谓的第一个元素是指一个包涵4个int型元素的一维数组；所以a相当于一个包涵4个int型元素的一维数组的地址；如果要定义p的话，p的基类型就是包涵4个int型类型的一维数组。所以我们要定义一个包涵4个int型类型的一维数组的指针变量`int (*p)[4]`。

#### 利用指针变量引用多维数组中的数组

* `*(*(p+i)+j)`
  p指向一个包涵4个int型元素的一维数组，`p+i`是第`i＋1`个包涵4个int型元素的一维数组的地址，所以`p+i`等价于`&a[i]`。那么很明显的`*(p+i)`等价于`a[i]`，`*(p+i)+j`等价于`a[i]+j`，因为`a[i]+j`等价于`&a[i][j]`，所以`*(*(p+i)+j)`等价于`a[i][j]`。

####  指针变量的`++`操作

当我们对一个指向整型的指针变量进行`++`操作的时候，指针变量的值会增加4，也就是指针变量的基类型的大小。
