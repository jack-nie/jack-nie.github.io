---
layout: post
title:  "理解JavaScript中的闭包"
date:   2015-11-07
keywords: ["javascript","closures"]
description: "understanding javascript"
category: "JavaScript"
tags: ["JavaScript"]
---
{% include JB/setup %}
   通俗的讲闭包就是在函数内部定义函数，内部的函数可以访问其外部函数的作用域。函数的变量可以被隐藏与作用域链之内，看起来像是变量被函数包裹了起来，因此被称作闭包。
   从技术的角度讲，所有的JavaScript函数都是闭包，它们都是对象，它们都关联到作用域链。
   理解闭包，需要首先了解JavaScript的词法作用域规则，函数的执行依赖变量作用域，这个作用域是在函数定义时绑定的，而不是在函数调用时绑定的。JavaScript函数对象的内部状态不仅包含函数的代码逻辑，还包含了当前的作用域链。
   我们来看一下下面的在这个例子：

     {% highlight javascript%}
     JavaScript Code:
       var scope = "global scope"
       function checkScope() {
         var scope = "local scope";
         function f() {return scope;}
         return f;
       }
       checkScope()();
     {% endhighlight%}

运行代码可以发现,输出值为"local scope",这意味着闭包可以访问外部函数的变量，即使外部函数已经执行完毕。

闭包存储的是对外部变量的引用，而不是真实的值。当闭包被调用之前就改变了外部函数的变量的值的时候，情况会变得很有趣。通过这一特性，我们可以在JavaScript实现私有变量。

    {% highlight javascript %}
    JavaScript Code:
    function car() {
      var status = "driving";
      return {
        getStatus: function() {
          return status;
        },

        setStatus: function(newStatus) {
          status = newStatus;
        }
      }
    }

    var audi = car();
    audi.getStatus();
    audi.setStatus("stop");
    audi.getStatus();
    {% endhighlight %}

通过分析可以发现，内部函数访问的外部函数的实际变量，而不是拷贝。为了深刻的理解这一特性，我们来看一下下面的例子：

    {% highlight javascript %}
    JavaScript Code:
    var handlers = function(nodes) {
      var i;
      for (i = 0; i < nodes.length; i += 1) {
        nodes[i].onClick = function(e) {
          console.log(i);
        };
      }
    };
    {% endhighlight %}

这段函数的本意是当用户点击某一节点的时候，打印出该节点的序号，但是事实是每次都返回节点的数目的值。这是因为事件处理器绑定了变量`i`本身，而不是在函数构造时的`i`的值。

修复这段代码的方式是循环外部构造一个函数，然后在循环体内调用它。

    {% highlight javascript %}
    JavaScript Code:
    var handlers = function(nodes) {
      var helper = function(i) {
        return function(e) {
          console.log(i);
        };
      };

      var i;
      for (i = 0, i < nodes.length, i += 1) {
        nodes[i].onClick = helper(i);
      }
    };
    {% endhighlight %}

这样，我们就能得到正确的结果了，当然还有一种改进的方式就是在循环体内构造一个立即调用的函数。
但是在循环中构造函数会带来不必要的计算，还会引起混淆，所以不建议这么做。



参考文献:

- [Understand JavaScript Closures With Ease](http://javascriptissexy.com/understand-javascript-closures-with-ease/ "Understand JavaScript Closures With Ease")
- [JavaScript权威指南](http://www.amazon.cn/O-Reilly%25252525252525252525E7%25252525252525252525B2%25252525252525252525BE%25252525252525252525E5%2525252525252525252593%2525252525252525252581%25252525252525252525E5%252525252525252525259B%25252525252525252525BE%25252525252525252525E4%25252525252525252525B9%25252525252525252525A6%25252525252525252525E7%25252525252525252525B3%25252525252525252525BB%25252525252525252525E5%2525252525252525252588%2525252525252525252597-JavaScript%25252525252525252525E6%252525252525252525259D%2525252525252525252583%25252525252525252525E5%25252525252525252525A8%2525252525252525252581%25252525252525252525E6%252525252525252525258C%2525252525252525252587%25252525252525252525E5%252525252525252525258D%2525252525252525252597-%25252525252525252525E5%25252525252525252525BC%2525252525252525252597%25252525252525252525E5%2525252525252525252585%25252525252525252525B0%25252525252525252525E7%25252525252525252525BA%25252525252525252525B3%25252525252525252525E6%25252525252525252525A0%25252525252525252525B9/dp/B007VISQ1Y?SubscriptionId=AKIAJMGEVRIO53UGJCYQ&tag=16-28-282__-23&linkCode=sp1&camp=2025&creative=165953&creativeASIN=B007VISQ1Y "JavaScript权威指南(第6版)")
- [JavaScript语言精粹](http://www.amazon.cn/JavaScript%2525252525252525E8%2525252525252525AF%2525252525252525AD%2525252525252525E8%2525252525252525A8%252525252525252580%2525252525252525E7%2525252525252525B2%2525252525252525BE%2525252525252525E7%2525252525252525B2%2525252525252525B9-%2525252525252525E9%252525252525252581%252525252525252593%2525252525252525E6%2525252525252525A0%2525252525252525BC%2525252525252525E6%25252525252525258B%252525252525252589%2525252525252525E6%252525252525252596%2525252525252525AF%2525252525252525E2%252525252525252580%2525252525252525A2%2525252525252525E5%252525252525252585%25252525252525258B%2525252525252525E7%2525252525252525BD%252525252525252597%2525252525252525E5%252525252525252585%25252525252525258B%2525252525252525E7%2525252525252525A6%25252525252525258F%2525252525252525E5%2525252525252525BE%2525252525252525B7/dp/B0097CON2S?SubscriptionId=AKIAJMGEVRIO53UGJCYQ&tag=16-28-282__-23&linkCode=sp1&camp=2025&creative=165953&creativeASIN=B0097CON2S "JavaScript语言精粹(修订版)")
