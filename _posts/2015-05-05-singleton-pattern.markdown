---
layout: post
title:  "单例模式"
date:   2015-05-05
keywords: ["Ruby","设计模式"]
description: "singleton"
category: "design-pattern"
tags: ["Ruby","设计模式"]
---
{% include JB/setup %}

单例模式的设计思想很直接，只允许有一个类的实例存在。这对于一个给定的类只允许有一个实例化对象的应用非常有用，比如日志功能，数据库访问等。

关于单例模式更详细的解释如下。

以下摘自必应网典：
> 单例模式，也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类必须保证只有一个实例存在。在它的核心结构中只包含一个被称为单例类的特殊类。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为，比如在某个服务器程序中，该服务器的配置信息存放在一个文件中，这些配置数据由一个单例对象统一读取，然后服务进程中的其他对象再通过这个单例对象获取这些配置信息。通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源。这种方式简化了在复杂环境下的配置管理。实现单例模式的思路是：一个类能返回对象一个引用(永远是同一个)和一个获得该实例的方法（必须是静态方法，通常使用getInstance这个名称）；当我们调用这个方法时，如果类持有的引用不为空就返回这个引用，如果类保持的引用为空就创建该类的实例并将实例的引用赋予该类保持的引用；同时我们还将该类的构造函数定义为私有方法，这样其他处的代码就无法通过调用该类的构造函数来实例化该类的对象，只有通过该类提供的静态方法来得到该类的唯一实例。单例模式在多线程的应用场合下必须小心使用。解决这个问题的办法是为指示类是否已经实例化的变量提供一个互斥锁(虽然这样会降低效率)。

Ruby的标准库实现了singleton模块，可以用来实现单例模式。

### 场景
假设我们要为我们的博客应用构建一个类，该类用来保存博客应用的配置数据，该配置的实例只允许有一个。当然，我们可以通过创建一个module来模仿单例，但是我们必须要保证它不能被被复制。

### 实现
首先通过`require`和`include`引入`singleton`模块。
    
    {% highlight ruby %}
    Ruby Code:
    require 'singleton'

    class BlogConfig
      include Singleton
    end
    {% endhighlight %}

如果通过new方法来实例化该类，将会得到一个NoMethodError的异常，构造器被设置成私有的以防止其他的实例被创建。为了得到该类的实例，需要通过`instance`方法，当首次调用该方法时，该类的一个实例就被创建，再次调用，会返回已经创建好的实例。

    {% highlight ruby %}
    Ruby Code:
    first, second = BlogConfig.instance, BlogConfig.instance
    first == second
    #=> true
    {% endhighlight %}
### 自己造轮子
我们可以模仿Singleton模块自己创造一个。关键步骤在于将要include该模块的类的new方法设置为私有的，防止不小心调用了new方法。

    {% highlight ruby %}
    Ruby Code:
    module Singleton
      def self.included(base)
        base.instance_variable_set("@instance", base.new)
        base.extend(self)
        base.private_class_method :new
      end
    
      def instance
        instance_variable_get("@instance")
      end
    end
    
    class Test
      include Singleton
      def say(val)
        puts val
      end
    end
    
    
    a = Test.instance 
    b = Test.instance
    puts a.object_id
    puts b.object_id
    #=> 23304264
    #=> 23304264
    {% endhighlight %}

### 单例的缺点

- 单例通常提供对一些服务的全局访问。
- 单例允许你限制对象的数量。
- 单例提升了类之间的耦合程度。
- 单例在整个程序的运行期间都保存了状态。

参考资料：

- [Design Patterns in Ruby: Observer, Singleton](http://www.sitepoint.com/design-patterns-in-ruby-observer-singleton/ "Design Patterns in Ruby: Observer, Singleton")
- [Ruby Singleton Pattern](http://dalibornasevic.com/posts/9-ruby-singleton-pattern-again "Ruby Singleton Pattern")
- [Why Singletons are Evil](http://blogs.msdn.com/b/scottdensmore/archive/2004/05/25/140827.aspx "Why Singletons are Evil")

扩展阅读：

- [Ruby and the singleton pattern don't get along](https://practicingruby.com/articles/ruby-and-the-singleton-pattern-dont-get-along "Ruby and the singleton pattern don't get along")
- [Implementing and Testing the Singleton Pattern in Ruby](http://blog.8thlight.com/josh-cheek/2012/10/20/implementing-and-testing-the-singleton-pattern-in-ruby.html "Implementing and Testing the Singleton Pattern in Ruby")
