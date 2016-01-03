---
layout: post
title:  "理解JavaScript中的原型继承"
date:   "2016-01-03"
keywords: ["javascript", "prototype"]
description: "javascript prototype"
category: "javascript"
tags: ["javascript"]
---
{% include JB/setup %}
原型是JavaScript的核心概念之一，理解JavaScript的原型就显得尤为重要。在JavaScript中，类的所有对象都从同一个原型对象上继承属性,因此，原型对象是类的核心。

除了null对象之外，每一个JavaScript对象都和另一个对象关联，"另一个"对象就是原型。
根据创建对象的方式的不同，对象的原型也不同。举例来讲，通过对象直接创建的对象拥有同一个原型，可以通过Object.prototype来获得对原型对象的引用。通过关键字new和构造函数调用创建的对象就是构造函数的prototype的值。

这里需要指出的一点就是并不是每一个JavaScript对象都有原型，数量虽然不多，但是确实存在，Object.prototype就是其中的一个。

下面让我们用实际的例子来看一下，原型继承到底是如何工作的

    {% highlight javascript %}
    Code:
      function Parent(name, job) {
        this.name = name;
        this.job  = job;
      } 

      Parent.prototype.getName = function() {
        return this.name;
      }

      Parent.prototype.getJob = function() {
        return this.job;
      }

      var parent = new Parent("Jack", "developer");
      console.log(parent.getName()); //Jack
      console.log(parent.getJob()); //developer

      function Child(name, job) {
        this.name = name;
        this.job  = job;
      }
      Child.prototype = new Parent();

      var child = new Child("Jones", "manager");
      console.log(child.getName());//Jones
      console.log(child.getJob());//manager
    {% endhighlight %}
基本的原型继承是相当简单的，但是这也有一个明显的问题，就是会重复构造函数中的初始化操作,这显然不是我们想看到的。
为了简化操作，我们可以在子类的构造函数上做文章。

    {% highlight javascript %}
    Code:
    function Child(name. job) {
      Parent.apply(this, arguments);
    }
    {% endhighlight %}
###利用create函数

这是来自JavaScript大师Douglas Crockford的方法，具体做法是给Object对象增加一个create方法，创建一个使用原对象作为原型的新对象。


    {% highlight javascript %}
    Code:
    Object.create = function(o) {
      var F = function () {};
      F.prototype = o;
      return new F();
    }
    {% endhighlight %}


原型关系是一种动态的关系，如果我们给一个原型添加新的属性，那么新创建的属性会对所有的基于该原型创建的对象可见。


- [JavaScript权威指南](http://www.amazon.cn/O-Reilly%25252525252525252525E7%25252525252525252525B2%25252525252525252525BE%25252525252525252525E5%2525252525252525252593%2525252525252525252581%25252525252525252525E5%252525252525252525259B%25252525252525252525BE%25252525252525252525E4%25252525252525252525B9%25252525252525252525A6%25252525252525252525E7%25252525252525252525B3%25252525252525252525BB%25252525252525252525E5%2525252525252525252588%2525252525252525252597-JavaScript%25252525252525252525E6%252525252525252525259D%2525252525252525252583%25252525252525252525E5%25252525252525252525A8%2525252525252525252581%25252525252525252525E6%252525252525252525258C%2525252525252525252587%25252525252525252525E5%252525252525252525258D%2525252525252525252597-%25252525252525252525E5%25252525252525252525BC%2525252525252525252597%25252525252525252525E5%2525252525252525252585%25252525252525252525B0%25252525252525252525E7%25252525252525252525BA%25252525252525252525B3%25252525252525252525E6%25252525252525252525A0%25252525252525252525B9/dp/B007VISQ1Y?SubscriptionId=AKIAJMGEVRIO53UGJCYQ&tag=16-28-282__-23&linkCode=sp1&camp=2025&creative=165953&creativeASIN=B007VISQ1Y "JavaScript权威指南(第6版)")
- [JavaScript语言精粹](http://www.amazon.cn/JavaScript%2525252525252525E8%2525252525252525AF%2525252525252525AD%2525252525252525E8%2525252525252525A8%252525252525252580%2525252525252525E7%2525252525252525B2%2525252525252525BE%2525252525252525E7%2525252525252525B2%2525252525252525B9-%2525252525252525E9%252525252525252581%252525252525252593%2525252525252525E6%2525252525252525A0%2525252525252525BC%2525252525252525E6%25252525252525258B%252525252525252589%2525252525252525E6%252525252525252596%2525252525252525AF%2525252525252525E2%252525252525252580%2525252525252525A2%2525252525252525E5%252525252525252585%25252525252525258B%2525252525252525E7%2525252525252525BD%252525252525252597%2525252525252525E5%252525252525252585%25252525252525258B%2525252525252525E7%2525252525252525A6%25252525252525258F%2525252525252525E5%2525252525252525BE%2525252525252525B7/dp/B0097CON2S?SubscriptionId=AKIAJMGEVRIO53UGJCYQ&tag=16-28-282__-23&linkCode=sp1&camp=2025&creative=165953&creativeASIN=B0097CON2S "JavaScript语言精粹(修订版)")
