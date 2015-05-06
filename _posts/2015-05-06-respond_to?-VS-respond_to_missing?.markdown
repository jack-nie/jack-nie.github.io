---
layout: post
title:  "respond_to? VS respond_to_missing?"
date:   2015-05-06
keywords: ["Ruby"]
description: "respond_to? VS respond_to_missing?"
category: "Ruby"
tags: ["Ruby"]
---
{% include JB/setup %}

通过`method_missing?`方法可以动态的向类中添加不存在的方法，这是ruby种的一个很酷的特性！但是如果使用不当，也会带来一些麻烦。一个常见的错误就是忘记覆写`respond_to?`方法。下面让我们来看一个例子：
    {% highlight ruby %}
    Ruby Code:
	require 'ostruct'
	
	class Order
	  def user
	    @_user ||= OpenStruct.new(name: 'Mike', age: 28, occupation: 'slacker')
	  end
	
	  def method_missing(method_name, *arguments, &block)
	    if method_name.to_s =~ /user_(.*)/
	      user.send($1, *arguments, &block)
	    else
	      super
	    end
	  end
	
	end
    
	order = Order.new
	p order.user_name
	p order.respond_to? :user_name
	p order.method(:user_name)
    {% endhighlight %}
接下来，让我们看一下运行的结果：

	#=> "Mike"
	#=> validate.rb:73:in `method': undefined method `user_name' for class
        `Order' (NameError)
	#=> false
	
通过上面的结果可以看到`respond_to?`是不认识新方法的,而`method`来得到新方法引发了一个错误，接下来在类中增加一个`respond_to?`方法。
    {% highlight ruby %}
    Ruby Code:
	def respond_to?(method_name, include_private = false)
	  method_name.to_s.start_with?('user_') || super
	end
    {% endhighlight %}
再看一下结果。
    
    #=> "Mike"
	#=> validate.rb:73:in `method': undefined method `user_name' for class
        `Order' (NameError)
	#=> true

现在`respond_to?`返回`true`，但是仍然无法通过`method`方法来得到新方法。最后，让我们将`respond_to?`替换为`respond_to_missing?`,再看一下结果。
    
	#=> "Mike"
	#=> true
	#=> #<Method: Order#user_name>
现在，是我们期望的结果了。

###总结

- 通过`method_missing?`机制添加新的方法时总是应该覆写`respond_to_missing?`方法，可以帮助Ruby解释器更好的理解新方法的存在。

参考资料：

- [Always Define respond_to_missing? When Overriding method_missing](https://robots.thoughtbot.com/always-define-respond-to-missing-when-overriding "Always Define respond_to_missing? When Overriding method_missing")