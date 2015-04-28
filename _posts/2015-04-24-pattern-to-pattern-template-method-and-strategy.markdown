---
layout: post
title:  "【翻译】模板方法模式和策略模式的比较"
date:   2015-04-24
keywords: ["Ruby","设计模式"]
description: "模板方法模式和策略模式的比较"
category: "design-pattern"
tags: ["Ruby","设计模式"]
---
{% include JB/setup %}
最近我写了一片关于<a href="http://rubylogs.com/template-method-pattern-in-ruby/">模板方法模式</a>以及怎样在Ruby中实现的文章。有读者在文章的评论中指出模板方法模式实际上是策略模式。在经过仔细的考虑该怎么回答这个问题在之后，我决定写一篇文章来比较这两个模式。

所以，在这里我将展示我对于设计模式的理解，让我们来看一下这两个模式之间的共同点以及二者之间的关键区别。

###模板方法模式
关于这个模式需要记住的最重要的一点是：样板方法写在父类中，而特定的算法则在子类中实现(通常是覆盖)。也就是说，所有的子类都会共享父类的功能(继承)，但父类仅仅是一个框架类。另一方面来讲，子类只能复写特定于其领域的方法。
###策略模式
策略模式的核心是将逻辑封装进对象中，使的在这些对象可以互换并将对象作为算法实现的一部分，也就是说算法的行为可以通过策略在运行时改变。
###实现策略模式
让我们来通过下面的小例子看一下如何使用策略模式，首先创建一个简单的`Invoice`类：
    {% highlight ruby %}
    Ruby Code:
	class Invoice
	  attr_accessor :formatter
	 
	  def initialize amount, buyer, seller
	    @amount = amount
	    @buyer = buyer
	    @seller = seller
	    @formatter = JSONFormatter.new
	  end
	 
	  def generate
	    @formatter.format! @amount, @buyer, @seller
	  end
	 
	end
	{% endhighlight %}
就像你所看到的，发票包含数量，买家的名字，卖家的名字，在它的构造方法中，一个格式器被创建。格式器是一个拥有`format！`方法的类的实例，它将amount，buyer，seller作为参数。当该方法被调用，将会创建一个特定格式的`Invoice`对象。策略类的一个非常重要的方面是上下文类期望通过策略类来实现，在我们的例子中是`format！`方法。举例来讲，在Java它将通过接口来实现，它强迫实现类实现某些特定的方法。但是因为Ruby是一门动态语言，它没有接口之类的东西，这一点我们必须提前考虑到。

话虽这么说，还是让我们来看一看格式器类，它们都是非常简单的。
    {% highlight ruby %}
    Ruby Code:
	class JSONFormatter
	  def format! amount, buyer_name, seller_name
	    %Q{
	      {
	        "buyer" => "#{buyer_name}",
	        "seller" => "#{seller_name}",
	        "amount" => "#{amount}"
	      }
	    }
	  end
	end
    {% endhighlight %}
类`JSONFormatter`以JSON格式创建发票，说句题外话，我在这里用了百分比符号，通过这种形式使得代码拥有更好的可读性。
    {% highlight ruby %}
    Ruby Code:
	class XMLFormatter
	  def format! amount, buyer_name, seller_name
	    %Q{
	     <invoice>
	       <buyer>#{buyer_name}</buyer>
	       <seller>#{seller_name}</seller>
	       <amount>#{amount}</amount>
	     </invoice>
	   }
	  end
	end
	{% endhighlight %}
类`XMLFormatter`以`XML`格式创建发票。
    {% highlight ruby %}
    Ruby Code:
	class YAMLFormatter
	  def format! amount, buyer_name, seller_name
	    %Q{
	      ---
	      invoice:
	        buyer: #{buyer_name}
	        seller: #{seller_name}
	        amount: #{amount}
	    }
	  end
	end
	{% endhighlight %}
写在最后但并非不重要，类`YAMLFormatter`以`YAML`格式创建发票。策略模式的优美指出存在于上下文对象（`Invoice`对象）和策略(格式器)中。如你所见，当我们首先创建一个`Invoice`对象时，我们能够创建一个`JSON`格式的发票，这非常的酷。更酷的是你可以在运行时改变`Invoice`对象的行为，也就是说如果你改变了格式器对象，那么`Invoice`对象的行为也将改变。更抽象的将，上下文对象会随着赋予的每个不同的策略改变。那么怎样将其应用到我们的代码中呢？
    {% highlight ruby %}
    Ruby Code:
	invoice = Invoice.new 100, "John", "Jane"
	puts invoice.generate
	{% endhighlight %}
我们第一次输出Invoice，采用JSONFormatter，当我们我们将得到一个JSON格式的发票。
    {% highlight ruby %}
    Ruby Code:
	{
	  "buyer" => "John",
	  "cashier" => "Jane",
	  "amount" => "100"
	}
    {% endhighlight %}
现在，如果我们对同一个`Invoice`对象采用不同的格式器……
	{% highlight ruby %}
	Ruby Code:
	invoice.formatter = XMLFormatter.new
	puts invoice.generate
	{% endhighlight %}
我们将得到同样的发票，不过是`XML`格式的。
    {% highlight ruby %}
    Ruby Code:
	<invoice>
	  <buyer>John</buyer>
	  <cashier>Jane</cashier>
	  <amount>100</amount>
	</invoice>
	{% endhighlight %}
如你所见，我们改变了格式器，`Invoice`对象的行为也改变了。

###策略模式和模板方法模式
那么这二者之间的共同点和关键的不同之处是什么呢？我们你们其中的一些人已经很清楚了，另外一些人可能会很困惑（就像我最初一样）。二者都是将领域特定算法封装进对象中。最关键的区别在于策略模式通过策略在运行时改变上下文对象的行为，模板方法模式通过使用一个算法的框架实现，并且通过在框架的子类中覆写方法来改变其行为。

原文：

- [http://rubylogs.com/pattern-to-pattern-template-method-and-strategy/](http://rubylogs.com/pattern-to-pattern-template-method-and-strategy/ "Pattern to pattern: Template Method & Strategy")