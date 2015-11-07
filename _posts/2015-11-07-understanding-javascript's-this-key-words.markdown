---
layout: post
title:  "理解JavaScript中的this关键字"
date:   2015-11-07
keywords: ["javascript","this"]
description: "understanding javascript"
category: "JavaScript"
tags: ["JavaScript"]
---
{% include JB/setup %}
   JavaScript函数调用中关于对象的引用一直是一个令人头疼的问题，造成这种困扰的很大一部分的原因是`this`关键字造成的。本文将深入探讨`this`关键字的各种用法，以及在不同情景中的表现。
   `this`是一个关键字，不是变量，也不是属性名，JavaScript的语法不允许给`this`赋值，它没有作用域的限制。

###普通的函数调用

  在JavaScript中，每一个函数调用都会包含一个`this`值。对于普通的函数调用`this`指向全局对象，在严格模式下,`this`则为`undefined`。
  以函数形式调用的函数通常不用指定`this`关键字，不过可以使用`this`来判断当前是否处于严格模式中。

    {% highlight javascript %}
    var strict = (function() { return !this; }());
    {% endhighlight %}

  普通函数调用的例子：

    {% highlight javascript %}
    JavaScript Code:
    function sayHi(name) {
      console.log(this + " says hello " + name);
    }
    sayHi("Jack");//[object Window] says hello Jack
    sayHi.call("Jones", "Jack");//Jones says hello Jack
    window.sayHi("Jack");//[object Window] says hello Jack
    sayHi.call(window,"Jack");//[object Window] says hello Jack
    {% endhighlight %}

###嵌套的函数调用

  嵌套的函数不会从调用它的函数中继承`this`,如果嵌套函数作为方法调用，那么`this`指向调用它的对象。如果嵌套函数作为函数调用，那么`this`
  的值是全局对象或者`undefined`,这取决与处于何种模式下（严格模式、非严格模式）。如果想访问外部函数的`this`值,则需要将`this`的值保存到一个变量中。

    {% highlight javascript %}
    JavaScript Code:
    var person = {
      getName: function() {
        var that = this;
        console.log(this === person);//true
        f();
        function f() {
          console.log(this === person);//false
          console.log(that === person);//true
        }
      }
    };
    {% endhighlight %}

###成员函数调用
  
  一种常见的情形是成员函数的使用，通过该对象来调用。
   {% highlight javascript %}
   JavaScript Code:
    var person = {
      name: "Jack";
      sayHi: function(thing) {
        console.log(this.name + " says hello " + thing);
      }
    };
    person.sayHi("world!");//Jack says hello world!
    {% endhighlight %}

函数`sayHi`如何同`person`对象关联并不影响`this`的指向，我们来看一个动态关联的例子：

    {% highlight javascript %}
    JavaScript Code:
    var person = { name: "Jack"};
    function sayHi(thing) {
        console.log(this.name + " says hello " + thing);
    }
    person.sayHi = sayHi;
    person.sayHi("world!");//Jack says hello world!
    sayHi("world!");//[object window] says hello world!
    {% endhighlight %}

###间接调用

我们可以通过`call`、`apply`来间接的调用函数，这两个都允许显示的指定调用所需的`this`值，也就是说任何函数都可以作为任何对象的方法来调用，即使这个函数不是某对象的方法。这两个方法的不同之处在于`call`使用自有的实参列表作为函数的实参，`apply`接受一个数组。

    {% highlight javascript %}
    JavaScript Code:
    function sayHi(name) {
      console.log(this + " says hello " + name);
    }
    sayHi.call("Jones", "Jack");//Jones says hello Jack
    sayHi.call(window,"Jack");//[object Window] says hello Jack
    sayHi.apply("Jones", ["Jack"]);//Jones says hello Jack
    {% endhighlight %}

有时候我们希望能够引用一个有着持久化`this`值的函数，那么我们可以使用`bind`函数。


    {% highlight javascript %}
    JavaScript Code:
    function bind(func, obj) {
      if (func.bind) {
        return func.bind(obj);
      } else {
        return function() {
          func.apply(obj, arguments);
        };
      }
    }
    var bindSayHi = bind(person.sayHi, person);
    bindSayHi("world!");//Jack says hello world!
    {% endhighlight %}

###jQuery中的this

    {% highlight javascript %}
    JavaScript Code:
    $("button").click(function(event) {
      console.log($(this).prop("name"));
    });
    {% endhighlight %}

`$(this)`等同于`jQuery`中的`this`关键字，它被用在一个匿名函数内部，该函数在`button`的`click`方法中执行。`jQuery`将`$(this)`绑定到调用`click`方法的对象上，所以`$(this)`指向`$("button")`对象，即使`$(this)`被用在匿名函数内部，外部`this`对其不可见。

参考文献:

- [Understand JavaScript’s “this” With Clarity, and Master It](http://javascriptissexy.com/understand-javascripts-this-with-clarity-and-master-it/ "Understand JavaScript’s “this” With Clarity, and Master It")
- [Understanding JavaScript Function Invocation and "this"](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/ "Understanding JavaScript Function Invocation and 'this'")
- [JavaScript权威指南](http://www.amazon.cn/O-Reilly%25252525252525252525E7%25252525252525252525B2%25252525252525252525BE%25252525252525252525E5%2525252525252525252593%2525252525252525252581%25252525252525252525E5%252525252525252525259B%25252525252525252525BE%25252525252525252525E4%25252525252525252525B9%25252525252525252525A6%25252525252525252525E7%25252525252525252525B3%25252525252525252525BB%25252525252525252525E5%2525252525252525252588%2525252525252525252597-JavaScript%25252525252525252525E6%252525252525252525259D%2525252525252525252583%25252525252525252525E5%25252525252525252525A8%2525252525252525252581%25252525252525252525E6%252525252525252525258C%2525252525252525252587%25252525252525252525E5%252525252525252525258D%2525252525252525252597-%25252525252525252525E5%25252525252525252525BC%2525252525252525252597%25252525252525252525E5%2525252525252525252585%25252525252525252525B0%25252525252525252525E7%25252525252525252525BA%25252525252525252525B3%25252525252525252525E6%25252525252525252525A0%25252525252525252525B9/dp/B007VISQ1Y?SubscriptionId=AKIAJMGEVRIO53UGJCYQ&tag=16-28-282__-23&linkCode=sp1&camp=2025&creative=165953&creativeASIN=B007VISQ1Y "JavaScript权威指南(第6版)")
