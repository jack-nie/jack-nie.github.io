---
layout: post
title:  "理解JavaScript中的变量作用域及声明提前"
date:   2015-11-07
keywords: ["javascript","variable scope", "hoisting"]
description: "understanding javascript"
category: "JavaScript"
tags: ["JavaScript"]
---
{% include JB/setup %}
   在编程语言中，作用域控制着变量及参数的可见性与生命周期,能够减少名称冲突以及提供必要的自动内存管理。如果想要更好的理解及运用JavaScript,那么理解其作用域就显得很有必要，本文将深入探讨这一主题。
   变量的作用域定义了在该变量的上下文中对于该变量的可见性,JavaScript中的变量有全局变量和局部变量之分。

### 局部变量

   大多数语言都有块级作用域，在一个代码块中在所定义的代码块之外是不可见的，在代码块中定义的变量在代码块执行结束之后就会被释放掉。但是实际上JavaScript并不支持块级作用域，这常常会造成混淆。对于JavaScript来说，它拥有函数级的作用域，函数内部的变量只在其内部及内部定义的函数可见。
   {% highlight javascript%}
   JavaScript Code:
   var greet = "Hello world!";
   function sayHi() {
     var greet = "Hello inner!";
     console.log(greet);//"Hello inner!"
   }

   console.log(greet);//"Hello world!"
   {% endhighlight %}

### #JavaScript没有块级作用域

  让我们来看下面的一个例子：

   {% highlight javascript%}
   JavaScript Code:
     var greet = "Hello world!"
     if (greet) {
       greet = "hello inner!"
       console.log(greet);//"Hello inner"
     }
     console.log(greet)://"Hello inner"
   {% endhighlight %}
   我们可以看到代码块外部定义的变量的值已经被改变。

### #全局变量

  所有在函数外部定义的变量都是全局变量，对整个应用可见，如果在函数体内定义变量，但是没有使用`var`关键字，那么它也将成为全局变量。

   {% highlight javascript%}
   JavaScript Code:
     var greetOuter = "Hello world!";
     greetAnother = "Hello another!"
     function sayHi() {
       greet = "Hello inner!"
       console.log(greetOuter);//"Hello world!"
       console.log(greetAnother);//"Hello another!"
       console.log(greet);//"Hello inner!"
     }
     console.log(greet);//"Hello inner!"
   {% endhighlight %}

### #`setTimeout`中的变量实在全局作用域中执行的

   需要注意的是`setTimeout`中的所有函数都是在全局作用域中执行的。

   {% highlight javascript%}
   JavaScript Code:
   // setTimeout中的`this`对象指向全局变量`window`
   var highValue = 200;
   var constantVal = 2;
     var myObj = {
     highValue: 20,
     constantVal: 5,
     calculateIt: function () {
       setTimeout (function  () {
         console.log(this.constantVal * this.highValue);
       }, 2000);
     }
   }
   myObj.calculateIt(); // 400
   {% endhighlight %}

### 声明提前

  JavaScript的函数作用域是指在函数体内声明的所有变量在函数体内始终可见，也就是说，在函数体内变量在声明之前已经可以使用，JavaScript的这个特性被称为声明提前。JavaScript中的所有变量声明都被提升至函数体的顶部，但是不包括赋值。声明提前的这一步操作是在JavaScript引擎预编译时进行的，是在代码开始执行之前。

    {% highlight javascript%}
    JavaScript Code:
    var greet = "Hello world!";
    function sayHi() {
      console.log(greet);//"undefined"
      var greet = "Hello inner";
      console.log(greet);//"Hello inner"
    }
    {% endhighlight %}

   由于声明提前的存在，`greet`的声明被提升到了函数体的顶部，在函数体内的局部变量拥有更高的优先级，因此覆盖掉了同名的全局变量。声明提前只是提升了变量的声明，但是赋值操作并没有提前，还停留在原来的位置，所以函数体内的第一行语句输出为`undefined`。

   另外，需要指出的一点是，函数声明会比变量声明拥有更高的优先级，在提升的过程中会覆盖掉同名的变量声明，但是函数表达式则没有这一特性。

   由于声明提前的存在以及JavaScript没有块级作用域，所以推荐的做法是将变量的声明放在函数体的顶部，可以真实的反应变量的作用域。

参考文献:

- [JavaScript Variable Scope and Hoisting Explained](http://javascriptissexy.com/javascript-variable-scope-and-hoisting-explained/ "JavaScript Variable Scope and Hoisting Explained")
- [JavaScript权威指南](http://www.amazon.cn/O-Reilly%25252525252525252525E7%25252525252525252525B2%25252525252525252525BE%25252525252525252525E5%2525252525252525252593%2525252525252525252581%25252525252525252525E5%252525252525252525259B%25252525252525252525BE%25252525252525252525E4%25252525252525252525B9%25252525252525252525A6%25252525252525252525E7%25252525252525252525B3%25252525252525252525BB%25252525252525252525E5%2525252525252525252588%2525252525252525252597-JavaScript%25252525252525252525E6%252525252525252525259D%2525252525252525252583%25252525252525252525E5%25252525252525252525A8%2525252525252525252581%25252525252525252525E6%252525252525252525258C%2525252525252525252587%25252525252525252525E5%252525252525252525258D%2525252525252525252597-%25252525252525252525E5%25252525252525252525BC%2525252525252525252597%25252525252525252525E5%2525252525252525252585%25252525252525252525B0%25252525252525252525E7%25252525252525252525BA%25252525252525252525B3%25252525252525252525E6%25252525252525252525A0%25252525252525252525B9/dp/B007VISQ1Y?SubscriptionId=AKIAJMGEVRIO53UGJCYQ&tag=16-28-282__-23&linkCode=sp1&camp=2025&creative=165953&creativeASIN=B007VISQ1Y "JavaScript权威指南(第6版)")
- [JavaScript语言精粹](http://www.amazon.cn/JavaScript%2525252525252525E8%2525252525252525AF%2525252525252525AD%2525252525252525E8%2525252525252525A8%252525252525252580%2525252525252525E7%2525252525252525B2%2525252525252525BE%2525252525252525E7%2525252525252525B2%2525252525252525B9-%2525252525252525E9%252525252525252581%252525252525252593%2525252525252525E6%2525252525252525A0%2525252525252525BC%2525252525252525E6%25252525252525258B%252525252525252589%2525252525252525E6%252525252525252596%2525252525252525AF%2525252525252525E2%252525252525252580%2525252525252525A2%2525252525252525E5%252525252525252585%25252525252525258B%2525252525252525E7%2525252525252525BD%252525252525252597%2525252525252525E5%252525252525252585%25252525252525258B%2525252525252525E7%2525252525252525A6%25252525252525258F%2525252525252525E5%2525252525252525BE%2525252525252525B7/dp/B0097CON2S?SubscriptionId=AKIAJMGEVRIO53UGJCYQ&tag=16-28-282__-23&linkCode=sp1&camp=2025&creative=165953&creativeASIN=B0097CON2S "JavaScript语言精粹(修订版)")
