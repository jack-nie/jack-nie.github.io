---
layout: post
title:  "模板方法模式"
date:   2015-04-25
keywords: ["Ruby","设计模式"]
description: "模板方法模式"
category: "design-pattern"
tags: ["Ruby","设计模式"]
---
{% include JB/setup %}
作为一个软件工程师，拥有一些设计模式的知识总是会受欢迎的。如果你没有听过设计模式，他们基本上是开发者遇到的一些通用问题的可复用的解决方案。设计模式的列表很长，了解全部的设计模式是都非常困难的。好吧，困难或许不是一个很准确的词，但是要掌握全部的设计模式需要很多的实践。让我们来看看设计模式中最简单的一个——模板方法，并且用Ruby来实现它。
### 模板方法模式
模板方法模式定义了一个操作中算法的框架，而将一些步骤延迟到子类中。模板方法模式使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。模板方法是一种基于继承的代码复用技术，他是一种类行为模式。
模板方法模式是结构最简单的行为型设计模式，在其结构中只存在父类与子类之间的继承关系。通过使用模板方法模式，可以将一些复杂流程的实现步骤封装在一系列基本方法中，在抽象父类中提供一个称之为模板方法的方法来定义这些基本方法的执行次序，而通过其子类来覆盖某些步骤，从而使得相同的算法框架可以有不同的执行结果。模板方法模式提供了一个模板方法来定义算法框架，而某些具体步骤的实现可以在其子类中完成。
### 简单既是美
模板方法的优美之处在于它的简单性，但是关于它的定义却很冗长，导致你昏昏欲睡。通过将一系列的过程封装进不同的方法中来实现其行为的可配置性，使得子类可以覆写这些方法。下面是一个简单的例子：
    {% highlight ruby%}
    Ruby Code:
	class MagicCutter
	  def initialize vegetable
	    @vegetable = vegetable
	  end
	
	  def give_me_the_vegetable_cutted_off
	    prepapre_the_blade
	    cut_the_vegetable
	  end
	
	  def prepapre_the_blade
	    raise 'Called abstract method: prepapre_the_blade'
	  end
	
	  def cut_the_vegetable
	    raise 'Called abstract method: cut_the_vegetable'
	  end
	
	end
	{% endhighlight %}
框架方法是`give_me_the_vegetable_cutted_off`，它调用了一些列的方法，可以在子类中覆写这些方法使其拥有不同的行为。子类仅仅定义了那些将要改变的行为，覆写定义在框架类中的方法，但是`give_me_the_vegetable_cutted_off`不会被改变。让我们来看一下子类是如何实现的：
    {% highlight ruby%}
    Ruby Code:
	require_relative 'magic_cutter'
	
	class AubergineCutter < MagicCutter
	
	  def prepapre_the_blade
	    puts "Hey! that's a #{@vegetable}"
	    puts "i need a very sharp knife"
	  end
	
	  def cut_the_vegetable
	    puts "I slice the #{@vegetable}  in long thumbsized pieces, ready to be grilled"
	  end
	
	end
	
	class PotatoCutter < MagicCutter
	
	  def prepapre_the_blade
	    puts "Hey! that's a #{@vegetable}"
	    puts "i take a professional potato cutter for my french fries"
	  end
	
	  def cut_the_vegetable
	    puts "put the #{@vegetable} in the cutter an press it"
	  end
	
	end
    {% endhighlight %}
继承、方法覆写……这些看起来很熟悉，但是结果却令人满意。
    {% highlight ruby%}
    Ruby Code:
	au = AubergineCutter.new("Aubergine")
	po = PotatoCutter.new("Potato")
	au.give_me_the_vegetable_cutted_off
	po.give_me_the_vegetable_cutted_off
	{% endhighlight %}
### 钩子方法
通过观察上面的两个子类，可以发现`puts “Hey! that’s a #{@vegetable}”`被重复了两次，并且从来不会被改变。更进一步，可以发现`PotatoCutter `并没有定义对蔬菜进行剥皮的操作，这正是引入钩子方法的时刻。钩子方法允许在子类覆写框架的实现并且定义一些新的东西，或者继承其默认的实现。让我们通过一个例子来看一下如何实现钩子方法`truly_excitement`和`prepare_the_vegetable`。
	{% highlight ruby%}
	Ruby Code:
	class MagicCutter
	  def initialize vegetable
	    @vegetable = vegetable
	  end
	
	  def give_me_the_vegetable_cutted_off
	    truly_excitement
	    prepapre_the_vegetable
	    prepapre_the_blade
	    cut_the_vegetable
	  end
	
	  def truly_excitement
	    puts "Hey! that's a #{@vegetable}"
	  end
	
	  def prepapre_the_vegetable
	  end
	
	  def prepapre_the_blade
	    raise 'Called abstract method: prepapre_the_blade'
	  end
	
	  def cut_the_vegetable
	    raise 'Called abstract method: cut_the_vegetable'
	  end
	
	end
	
	class PotatoCutter < MagicCutter
	
	  def prepapre_the_vegetable
	    puts "skin the potato"
	  end
	
	  def prepapre_the_blade
	    puts "i take a professional potato cutter for my french fries"
	  end
	
	  def cut_the_vegetable
	    puts "put the potato in the cutter an press it"
	  end
	
	end
	
	c = PotatoCutter.new("Potato")
	c.give_me_the_vegetable_cutted_off
	
	class AubergineCutter < MagicCutter
	
	  def prepapre_the_blade
	    puts "i need a very sharp knife"
	  end
	
	  def cut_the_vegetable
	    puts "I slice the #{@vegetable}  in long thumbsized pieces, ready to be grilled"
	  end
	
	end
	
	c = AubergineCutter.new("Aubergine")
	c.give_me_the_vegetable_cutted_off
	{% endhighlight %}
这些钩子方法的默认实现通常是空的，他们的存在仅仅是为了告诉子类将要发生什么，但是不需要子类覆写不必要的方法。上面的例子中`prepare_vegetable`是一个空的方法，将留给子类来决定如何实现它。

### 一些需要注意的地方
模板方法越简单越好。
要避免写一个充满了方法的模板类，但是没有子类覆写这些方法，这会提高维护成本。
要避免提供过少的钩子，在某些时候，这就迫使子类不得不覆写整个方法。

参考资料：

- [http://reefpoints.dockyard.com/ruby/2013/07/10/design-patterns-template-pattern.html](http://reefpoints.dockyard.com/ruby/2013/07/10/design-patterns-template-pattern.html "Design Patterns: The Template Method Pattern")
- [http://edapx.com/2013/10/27/template-pattern/](http://edapx.com/2013/10/27/template-pattern/ "#1 template pattern")
- [http://tammersaleh.com/posts/the-template-pattern-is-underused/](http://tammersaleh.com/posts/the-template-pattern-is-underused/ "The Template Pattern is underused")
