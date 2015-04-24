---
layout: post
title:  "【翻译】instance_eval和instance_exec之间的区别"
date:   2015-04-22
---

`#instance_eval`和`#instance_exec`之间有一个非常重要的区别。<a href="https://github.com/thoughtbot/factory_girl">Factory Girl</a>是一个展示如何优雅的使用以上两者的很好的例子。

但是首先，在你急不可耐的准备构建令人惊奇的DSL之前，让我们来看一下`#instance_eval`是什么以及有什么作用。

最简单的例子来自于Ruby Docs:

		class KlassWithSecret
		  def initialize
		    @secret = 99
		  end
		end
		k = KlassWithSecret.new
		k.instance_eval { @secret } #=> 99

在提供的block中的`self`的当前值是调用`#instance_eval`的那个对象。所以，假设对象`k`是block的当前上下文；`@secret`是存储在`k`中的变量，`#instance_eval`打开了该对象的通道并且能够获取其所有的内部变量。

FactoryGirl提供的接口简单而直接，下面是一个来自于其"Getting Started"文档的例子。

		FactoryGirl.define do
		  factory :user do
		    first_name "Kristoff"
		    last_name  "Bjorgman"
		    admin false
		  end
		end

在这里，Factory Girl使用`#instance_eval`来执行传递给`factory`的代码块。让我们通过一些富有表现力的代码来看一下FactoryGirl是怎么让这些工作起来的。

		def factory(name, &block)
		  factory = Factory.new(name)
		  factory.instance_eval(&block) if block_given?
		  # ... more code
		end

实际上，这并不是来自于Factory Girl的代码， 但是它大致的说明了发生了什么。当`#factory`被调用时，一个新的`Factory`对象就被创建了，然后代码块就在该对象的上下文中执行。换句话讲，当你看到`first_name`时，就像看到该factory对象在前面，而不是`factory.first_name`。通过使用`#instance_eval`，Factory Girl的使用者不需要指定factory对象，它被隐式的添加。

好了，这一切都很好，但对于`#instance_exec`呢？我很高兴你问到了。

方法`#instance_eval`仅能评估(? evaluate~~怎么翻译好呢)代码块或者字符串，仅此而已。想要传递参数给代码块？你会遇到很大的麻烦。

但是`#instance_exec`不仅能够评估提供的代码块，还允许传递参数。让我们来看一个例子。

FactoryGirl允许通过使用回掉函数来执行一些动作，比如在对象创建之后。

		FactoryGirl.define do
		  factory :user do
		    first_name "Kristoff"
		    last_name "Bjorgman"
		    admin false
		
		    after(:create) do |user, evaluator|
		      create_list(:post, evaluator.posts_count, user: user)
		    end
		  end
		end
在这个例子中，`after(:create)`在对象创建之后执行，代码块接收了两个参数：`user`和`evaluator`。`user`参数是刚刚创建的user对象，`evaluator`参数存储了所有factory创建的所有的值。下面让我们来看一下这些是如何实现的。
		
		def run(instance, evaluator)
		  case block.arity
		  when 1, -1 then syntax_runner.instance_exec(instance, &block)
		  when 2 then syntax_runner.instance_exec(instance, evaluator, &block)
		  else        syntax_runner.instance_exec(&block)
		  end
		end

FactoryGirl会创建一个以传递给after method的参数命名的callback。在该情形中，callback的名字是`：create`，同时还有一个代码块。

我们的例子中使用的代码块有两个参数。`run`方法决定怎样执行来自于代码块的代码。

callback对象存储了代码块，并且Ruby允许我们检查代码块的元数，换句话讲，它允许我们检查参数的数量。

当看`case`语句时，先检查`else`是一个好的主意，这会让你知道在没有匹配到任何代码的时候，`when`的部分会发生什么。接下来我们来看`syntax_runner.instance_exec(&block)`,这可以很容易的被`instance_eval`代替。Ruby会在`syntax_runner`对象的上下文中评估或者执行代码块。

如果代码块的元数大于0，FactoryGirl会为对象提供一个代码块以便让这段代码符合我们的预期。case的第二部分检查代码块的元数是否等于2。

    when 2 then syntax_runner.instance_exec(instance, evaluator, &block)

如果是，`syntax_runner`会接受一个实例对象(在我们的例子中是`user`)和`evaluator`。如果代码块的元数是1或者-1，代码块将仅接受一个`instance`参数。

那么-1代表什么呢？让我们来看一下怎么创建一个callback：
		
		# Two arguments and arity of 2
		after(:create) do |user, evaluator|
		  create_list(:post, evaluator.posts_count, user: user)
		end
		# One argument and arity of 1
		after(:create) do |user|
		  create_group(:people, user: user)
		end
		# Zero arguments and arity of 0
		after(:create) do
		  puts "Yay!"
		end
		# Any arguments and arity of -1
		after(:create) do |*args|
		  puts "The user is #{args.first}"
		end

通过`*args`Ruby并不知道你究竟会传递几个参数，所以Ruby干脆撒手不管，直接传回一个奇怪的数字-1。

这就是理解怎样和合何时使用`instance_exec`的力量，DSL用户会期望这些会有作用，事实上，确实会。

但是等一等，还有更多！如果你想为不同的属性指定同样的值呢？

		FactoryGirl.define do
		  factory :user do
		    first_name "Kristoff"
		    last_name  "Bjorgman"
		
		    password "12345"
		    password_confirmation "12345"
		  end
		end
在上面的例子中，`password`和`password_confirmation`拥有同样的值，这可能不怎么好。如果你改了其中的一个，而忘了了更改另外一个？当他们被绑定在实现中，二者的不一致可能会导致一些意想不到的错误。

我更愿意或许你也是，告诉FactoryGirl使用已经定义好的值。幸运的是，FiactoryGrl允许我们使用Ruby的一个小技巧`#to_proc`,下面展示了如何使用。

		FactoryGirl.define do
		  factory :user do
		    first_name "Kristoff"
		    last_name  "Bjorgman"
		
		    password "12345"
		    password_confirmation &:password
		  end
		end
重要的部分是传递给`password_conformation`的`&：password`值，Ruby看到`&`字符会将其视为代码块调用了`to_proc`方法。为了实现这个功能，FactoryGirl在属性上定义`to_proc`，并且使用`instance_exec`将符号`password`提供给代码块。

		def to_proc
		  block = @block
		
		  -> {
		    value = case block.arity
		            when 1, -1 then instance_exec(self, &block)
		            else instance_exec(&block)
		            end
		    raise SequenceAbuseError if FactoryGirl::Sequence === value
		    value
		  }
		end

那么对于lambda和proc呢？一些评论者在Reddit上提出了一个很重要的问题：关于`#instance_eval`和`#instance_exec`在分别传递lambda和proc的时候是如何表现的。

如果你传递一个lambda并且不提供参数，`#instance_eval`会出错。
	
		object = Object.new
		argless = ->{ puts "foo" }
		object.instance_eval(&argless) #=> ArgumentError: wrong number of arguments (1 for 0)

之所以会出现这个错误，是因为Ruby会将当前对象作为`self`传递给block，解决方案是提供接受参数的lambda。

		args = ->(obj){ puts "foo" }
		object.instance_eval(&args) #=> "foo"
如果使用`#instance_exec`将会有点小变化。

		object.instance_exec(&argless) #=> "foo"
		object.instance_exec(&args) #=> ArgumentError: wrong number of arguments (0 for 1)
		object.instance_exec("some argument", &args) #=> "foo"

因为`proc`在参数校验方面不是那么严格，所以以上两种尝试都不会导致错误。

		p_argless = proc{ puts "foo" }
		object.instance_eval(&p_argless) #=> "foo"
		
		p_args = proc{|obj| puts "foo" }
		object.instance_eval(&p_args) #=> "foo"
		
		object.instance_exec(&p_args) #=> "foo"
		object.instance_exec(&p_argless) #=> "foo"

现在你知道了`#instance_eval`和`#instance_exec`在表现方式上是类似的，如果你需要传递参数，那么请使用`#instance_exec`。

翻译：[http://www.saturnflyer.com/blog/jim/2015/04/22/the-difference-between-instance_eval-and-instance_exec/](http://www.saturnflyer.com/blog/jim/2015/04/22/the-difference-between-instance_eval-and-instance_exec/ "The Difference Between Instance_eval And Instance_exec")

