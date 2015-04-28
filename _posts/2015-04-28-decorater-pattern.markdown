---
layout: post
title:  "【翻译】装饰器模式"
date:   2015-04-28
keywords: ["设计模式", "Rails", "Ruby"]
description: "装饰器模式"
category: "设计模式"
tags: ["设计模式"]
---
{% include JB/setup %}
装饰器是一种设计模式，它的目的正如Gang of Four所描述的：
> 装饰器模式，是面向对象编程领域中，一种动态地往一个类中添加新的行为的设计模式。就功能而言，装饰器模式相比生成子类更为灵活，这样可以给某个对象而不是整个类添加一些功能。

通过使用修饰模式，可以在运行时扩充一个类的功能。原理是：增加一个修饰类包裹原来的类，包裹的方式一般是通过在将原来的对象作为修饰类的构造函数的参数。装饰类实现新的功能，但是，在不需要用到新功能的地方，它可以直接调用原来的类中的方法。修饰类必须和原来的类有相同的接口。修饰模式是类继承的另外一种选择。类继承在编译时候增加行为，而装饰模式是在运行时增加行为。
###使用装饰器模式来代替继承
一个经常被用来说明装饰器模式的例子是“coffee with milk and sugar” ，下面是利用类继承来实现这个例子的代码。
    {% highlight ruby %}
    Ruby Code:
	class CoffeeWithSugar < Coffee
	  def cost
	    super + 0.2
	  end
	end
	
	class CoffeeWithMilkAndSugar < Coffee
	  def cost
	    super + 0.4 + 0.2
	  end
	end
    {% endhighlight %}
利用继承有如下缺点：

- 做出的选择是固化的。
- 客户端无法控制怎样和什么时候装饰一个组件。
- 紧耦合。
- 改变父类的接口意味着所有的子类也必须要改变。

在Ruby中，include一个模块在某种意义上来说也是继承关系。
	{% highlight ruby %}
	Ruby Code:
	module Milk
	  def cost_of_milk
	    0.4 if milk?
	  end
	end
	
	class Coffee
	  include Milk
	  include Sugar
	
	  def cost
	    2 + cost_of_milk + cost_of_sugar
	  end
	end
    {% endhighlight %}
###装饰器是如何工作的
用 Gang of Four的话来说，一个装饰器就是一个用来包裹一个组件的对象，它还有如下功能：

- 符合组件的接口，所以它的存在对客户端是透明的。
- 将请求转发（代理）给该组件。
- 在请求转发之前或者之后执行一些额外的操作。

这种方法比类继承更加的灵活，因为你可以有多种组合混合和匹配职责因为它的透明性，你可以递归的嵌套装饰器，所以它允许添加无限数量的职责。

###Ruby中的替代实现
我通过研究在Ruby中发现了4个常用的实现：

- Module + Extend + Super 装饰器
- Plain Old Ruby Object 装饰器
- Class + Method Missing 装饰器
- SimpleDelegator + Super + Getobj 装饰器

可能还有其他的实现，但是这4个是最常见的。

###Module + Extend + Super装饰器
在Design Patterns in Ruby一书中描述了此种实现，我认为这是值得提倡的。它由`module`、`super`和`extend`组成。

为了保持一致性，让我们继续回到“coffee with milk and sugar”这个例子，它被实现成这样：
    {% highlight ruby %}
    Ruby Code:
	class Coffee
	  def cost
	    2
	  end
	end
	
	module Milk
	  def cost
	    super + 0.4
	  end
	end
	
	module Sugar
	  def cost
	    super + 0.2
	  end
	end
	
	coffee = Coffee.new
	coffee.extend(Milk)
	coffee.extend(Sugar)
	coffee.cost   # 2.6
	{% endhighlight %}
此种实现的好处是：

- 它通过所有的装饰器代理。
- 它具有所有的原始接口，因为它是原始的对象。

此种实现的缺点是：

- 不能对同一个对象多次使用同样的装饰器。
- 很难分辨出是哪一个装饰器增加的功能。

###PORO装饰器
我建议在Ruby程序中从此种形式的装饰器开始，包括Rails应用，直到这种形式失败了才寻求其他的方式。
    {% highlight ruby %}
    Ruby Code:
	class Coffee
	  def cost
	    2
	  end
	
	  def origin
	    "Colombia"
	  end
	end
	
	class Milk
	  def initialize(component)
	    @component = component
	  end
	
	  def cost
	    @component.cost + 0.4
	  end
	end
	
	coffee = Coffee.new
	Sugar.new(Milk.new(coffee)).cost  # 2.6
	Sugar.new(Sugar.new(coffee)).cost # 2.4
	Sugar.new(Milk.new(coffee)).class # Sugar
	Milk.new(coffee).origin           # NoMethodError
	{% endhighlight %}
该实现的优点如下：

- 可以通过Ruby的实例化无限的包装。
- 通过所有的装饰器代理。
- 可以多次对一个组件使用同样的装饰器。

该实现的缺点如下：

- 不能以透明的方式使用组件的原始接口，这个缺点也意味着按照Gang of
Four的定义，它并不是一个真的装饰器。我主张仍然称它为装饰器，因为它看起来并且它的行为绝对像是一个装饰器。

这是一个相当有粘性的观点，然而，还是让我们详细的讨论Gang of Four的“透明接口”吧。

###我们在乎“透明接口”的需求吗
假定我们关心装饰器的透明接口是因为成本，如果是这样，我们不需要同时只支持原始方法。因此，PORO装饰器满足我们的实际需求。通过将接口的范围重新定义为该对象的整个接口的一个子集，我们就符合Gang of Four的定义，这算是一种作弊手段么？我说不，请考虑在Ruby 1.9.3中有多少方法是在`Obeject`上：
    {% highlight ruby %}
    Ruby Code:
	> Object.new.methods.size
	=> 56
	{% endhighlight %}
Rails中的`Object`中的方法是Ruby种方法的两倍：
    {% highlight ruby %}
    Ruby Code:
	> Object.new.methods.size
	=> 118
	{% endhighlight %}
一个`ActiveRecord`对象上甚至有更多的方法：
    {% highlight ruby %}
    Ruby Code:
	> User.new.methods.size
	=> 366
	{% endhighlight %}
我们并不在每一个对象上使用上百种中方法，尤其是在Rails View的典型应用中。

然而，如果我们在Rails应用中使用PORO装饰器来装饰`ActiveRecord`对象，我们可能通过接近300个方法减少了接口。这可能是，也可能不是一个问题，取决于我们在该应用中如何使用该对象。在实际应用中，当使用TDD开发一个新功能，我并没有发现这是一个问题，这就是我为什么要说从这个装饰器开始。如果这不是一个问题，那么很好，你的测试套件将会告诉你。你可能会决定增加一个或者两个方法来做非常清晰的代理。
    {% highlight ruby %}
    Ruby Code:
	def comments
	  @component.comments
	end
	
	def any?
	  @component.any?
	end
    {% endhighlight %}
然而，你可能会觉得这很冗长或者重复，所以让我们在晚些时候来假设我们很关心透明接口。

这经常通过Ruby的`method_missing`来完成，或者通过Ruby的`delegate`库，如 `Delegator`、 `SimpleDelegator`、 `DelegateClass`、`Forwardable`。

###“Method Missing”装饰器
下面的例子通过`method_missing`来实现一个Ruby装饰器：
    {% highlight ruby %}
    Ruby Code:
	module Decorator
	  def initialize(component)
	    @component = component
	  end
	
	  def method_missing(meth, *args)
	    if @component.respond_to?(meth)
	      @component.send(meth, *args)
	    else
	      super
	    end
	  end
	
	  def respond_to?(meth)
	    @component.respond_to?(meth)
	  end
	end
	
	class Coffee
	  def cost
	    2
	  end
	
	  def origin
	    "Colombia"
	  end
	end
	
	class Milk
	  include Decorator
	
	  def cost
	    @component.cost + 0.4
	  end
	end
	
	coffee = Coffee.new
	Sugar.new(Milk.new(coffee)).cost   # 2.6
	Sugar.new(Sugar.new(coffee)).cost  # 2.4
	Sugar.new(Milk.new(coffee)).origin # Colombia
	Sugar.new(Milk.new(coffee)).class  # Sugar
	{% endhighlight %}
此种实现的优点如下：

- 可以通过Ruby实例化无限的包装。
- 通过所有的装饰器代理。
- 可以对同一个组件多次使用同一个装饰器。
- 透明的利用组件的原始接口。

此种实现的缺点如下：

- 使用了`method_missing`。
- 被装饰的对象的类是该装饰器。

###我们关心类是装饰器吗
我们不应该，这是Ruby，鸭子类型的大陆。

然而，Ruby在多态关系，form_for以及其他的地方的反射机制(`object.class.name`)，当我尝试将Rails view帮助方法转换为装饰器的时候，测试执行时，这在ActiveRecord的错误形式中确实是一个问题。
> 我刚刚发现了一个问题，Rails在检查一个对象的`to_patial_path`的属性之前，会调用`to_model`方法，所以当你准备使用装饰器来设置`to_partial_path`,你必须覆写`to_model`方法来阻止它返回没有被装饰的对象。—— Andrew Meyer

在一些情境中，这暴露了一个更深的问题。通过对model代码进行重构，我能够使用PORO装饰器并且使整个代码库变得更加干净。在另外的情况下，我仅仅认为使类表现为组件的类是一项耗时少的工作，所以我继续下去。

###“SimpleDelegator + Super + Getobj” 装饰器
所以，在Rails中的一个让步是使用这种实现，我把它作为最后的解决方案。
    {% highlight ruby %}
    Ruby Code:
	class Coffee
	  def cost
	    2
	  end
	
	  def origin
	    "Colombia"
	  end
	end
	
	require 'delegate'
	
	class Decorator < SimpleDelegator
	  def class
	    __getobj__.class
	  end
	end
	
	class Milk < Decorator
	  def cost
	    super + 0.4
	  end
	end
	
	coffee = Coffee.new
	Sugar.new(Milk.new(coffee)).cost   # 2.6
	Sugar.new(Sugar.new(coffee)).cost  # 2.4
	Milk.new(coffee).origin            # Colombia
	Sugar.new(Milk.new(coffee)).class  # Coffee
    {% endhighlight %}
此种实现的优点如下：

- 可以通过Ruby实例化无限的包装。
- 通过所有的装饰器代理。
- 可以对同一个组件多次使用同一个装饰器。
- 透明的利用组件的原始接口。
- 类既是组件。

此种实现的缺点如下：

-它重新定义了类。

我真的不不是很清楚什么样的问题会导致使用`method_missing`或者重新定义一个类，但我想他们会在一个耗时的调试过程中得到清晰地展现。

###行动计划

我认为我现在理解不同的Ruby实现的好处和坏处，并且有一个如何使用它们的计划：

- 以PORO开始。
- 假如我需要透明接口或者被装饰的对象的类会在Rails中引发问题，使用`SimpleDelegator`

原文：</br>
[https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in "Evaluating Alternative Decorator Implementations In Ruby")</br>
深入阅读：</br>
[http://codebrahma.com/design/patterns/2014/04/28/ruby-decorators.html](http://codebrahma.com/design/patterns/2014/04/28/ruby-decorators.html "ruby-decorators")</br>
[https://sourcemaking.com/design_patterns/decorator](https://sourcemaking.com/design_patterns/decorator "decorator")</br>
[https://robots.thoughtbot.com/tidy-views-and-beyond-with-decorators](https://robots.thoughtbot.com/tidy-views-and-beyond-with-decorators "tidy-views-and-beyond-with-decorators")
