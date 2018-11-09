---
layout: post
title:  "观察者模式"
date:   2015-04-26
keywords: ["Ruby","设计模式"]
description: "观察者模式"
category: "design-pattern"
tags: ["Ruby","设计模式"]
---
观察者模式（有时又被称为发布/订阅模式）是软件设计模式的一种。在此种模式中，一个目标对象管理所有相依于它的观察者对象，并且在它本身的状态改变时主动发出通知。这通常透过呼叫各观察者所提供的方法来实现。此种模式通常被用来实作事件处理系统。在Ruby标准库中，观察者模式得到了很好的支持，开箱即用，但是为了更好的说明解释观察者模式，我们首先看一个不利用Ruby的Observer机制的例子。
### Observable
首先为被观察对象创建一个`Observable`模块，该模块的职能是在其状态发生改变时通知一个或多个观察者。
	Ruby Code:
	module Observable

	  attr_accessor :state

	  def attachObserver(o)
	    @observers ||= []
	    @observers << o if !@observers.include?(o)
	  end

	  def removeObserver(o)
	    @observers.delete(o)
	  end

	  def state=(s)
	    @state = s
	    @observers.each do |o|
	      o.update(self) # more on update() later!
	    end
	  end

	end
方法`attachObserver`和`removeObserver`用来管理观察者，我们不想拥有相同的观察者，所以只添加那些没有观察我们定义的主体的观察者对象。
    Ruby Code:
	def state=(s)
	  @state = s
	  @observers.each do |o|
	    o.update(self) # more on update() later!
	  end
	end
当被观察对象的状态发生了改变，就需要通知其观察了该对象的观察者。首先，我们覆写了有方法`attr_accssor`默认创建的修改器，在这个例子中是`state=`。在给state赋一个新的值后，调用所有观察者的`update`方法。
### Observer
下面是Observer模块的实现：
    Ruby Code:
	module Observer

	  def update(o)
	    raise 'Implement this!'
	  end

	end
在该模块中，将`update`的具体功能留给包含它的类来实现，通过这种方式，`Observer`模块的行为就像是一个需要被其包含者实现的抽象类。
### 蚁群
作为一个简单的例子，首先定义我们的主体，也就是被观察者--蚂蚁女王。
    Ruby Code:
	class QueenAnt
	  include Observable
	end
接下来定义一个观察者：
    Ruby Code:
	class WorkerAnt
	  include Observer

	  def update(o)
	    p "I am working hard. Queen has changed state to #{o.state}!"
	  end
	end
类WorkerAnt定义了在蚂蚁女王的状态发生改变时的特殊行为。
    Ruby Code:
	queen = QueenAnt.new

	worker1 = WorkerAnt.new
	worker2 = WorkerAnt.new
	worker3 = WorkerAnt.new

	queen.attachObserver(worker1)
	queen.attachObserver(worker2)

	queen.state = 'sleeping'
	# I am working hard. Queen has changed state to sleeping!"
	# I am working hard. Queen has changed state to sleeping!"
当蚂蚁女王的状态发生了改变，实例`worker1`、`worker2`会收到通知，并针对性的做出响应，实例`worker3`并没有并加入观察者列中，所以它不会受到通知。

### Obserever机制
通过Ruby的Observer机制来实现观察者模式是很便捷的，只需要将`observer`模块混入被观察者对象就可以。
    Ruby Code:
	# require the observer lib file
	require “observer”
	class Notifier
	end

	class EmailNotifier < Notifier
	  def update(bank_account)
	    if bank_account.balance <= 10
	      puts “Sending email to: ‘#{bank_account.owner}’ informing \n
                him with his account balance: #{bank_account.balance}$.”
	      # send the email mechanism
	    end
	  end
	end

	class SMSNotifier < Notifier
	  def update(bank_account)
	    if bank_account.balance <= 0.5
	      puts “Sending SMS to: ‘#{bank_account.owner}’ informing \n
               him with his account balance: #{bank_account.balance}$.”
	      # send sms mechanism
	    end
	  end
	end

	class BankAccount
	  # include the observable module
	  include Observable
	  attr_reader :owner,:balance
	  def initialize(owner,amount)
	    @owner,@balance = owner,amount
	    # adding list of observes to the account
	    add_observer EmailNotifier.new
	    add_observer SMSNotifier.new
	  end

	  # withdraw operation
	  def withdraw(amount)
	    # do whatever you need
	    @balance -=amount if (@balance - amount) > 0
	    # now here comes our logic
	    # issue that the account has changed
	    changed
	    # notify the observers
	    notify_observers self
	  end
	end
	account = BankAccount.new “Jim Oslen”, 100
	account.withdraw 99.5
	#=>Sending email to: ‘Jim Oslen’ informing him with
    #his account balance: 0.5$.
	#=>Sending SMS to: ‘Jim Oslen’ informing him with his
    #account balance: 0.5$.
利用Ruby的Observer机制我们要做下面的工作：

- 引入`observer`库并将其包含在被观察者类中。
- 改变对象的状态并通知观察者。
- 让所有需要的观察者和被观察者关联。
- 每个观察者必须实现`update`方法。

### 总结
观察者模式和<a href="https://jack-nie.github.io/blog/pattern-to-pattern-template-method-and-strategy/">策略模式</a>之间有一些相似之处,二者都是用一个对象（观察者模式的被观察者对象和策略模式的上下文对象）调用另一个对象（观察者模式的观察者和策略模式的策略）。二者的区别在于目的和适用情形的不同，观察者模式通过观察主题的不同状态来做出响应，策略模式则依赖于具体的策略来实现。

参考资料：

- [http://blog.khd.me/ruby/observer-and-singleton-design-patterns-in-ruby/](http://blog.khd.me/ruby/observer-and-singleton-design-patterns-in-ruby/ "OBSERVER AND SINGLETON DESIGN PATTERNS IN RUBY")
- [http://www.runtime-era.com/2012/12/basic-observer-design-pattern-in-ruby.html](http://www.runtime-era.com/2012/12/basic-observer-design-pattern-in-ruby.html "Quick Review: Basic Observer Design Pattern in Ruby")
- [http://reefpoints.dockyard.com/2013/08/20/design-patterns-observer-pattern.html](http://reefpoints.dockyard.com/2013/08/20/design-patterns-observer-pattern.html "Design Patterns: The Observer Pattern")
