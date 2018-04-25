---
layout: post
title:  "理解Rails中的try方法"
date:   "2015-11-21"
keywords: ["rails","try"]
description: "understanding try method"
category: "Rails"
tags: ["Rails"]
---

随着`ruby-2.3.0-preview1`的发布，ruby语言原生的支持安全运算子。那么之前为了解决`nil`问题创建出来的诸如`NullObject`、`try`是不是就没有用了呢？答案是否定的。

在`Rails`中，经常会有很多的关联关系，这样就会用到链式方法的调用，其中一个不可忽视的问题就是对`nil`问题的处理。我们来看下面的一个例子,假设我们有一个product，我们要查询该product所属公司的名称，为了防止`nil`的错误，可能会采用如下写法：

    product.try(:company).try(:name)
    
那么使用安全运算符的方式`product&.company&.name`。

我们来看一下其它的例子：

      Ruby Code:
      require "rails/all"
      class Name
        def say
          p "hello"
        end

        def talk word
          p word
        end
      end

分别采用安全运算子和`try`方法进行调用的例子：

      Ruby Code:
      name = Name.new
      name.try(:say) #"hello"
      name.try(:talk, "hello") #"hello"
      name.try(:cry) #nil
      name&.say #"hello"
      name&.cry #NoMethodError: undefined method 'cry' for #<Name:0x000000018ada08>
      nil&.cry #nil
      #带参数的方法没法调用

通过以上例子可以发现，当调用者不为`nil`，而又不能响应被调用的方法时，会抛出`NoMethodError`的异常。

那么为什么`try`就能够处理这种情形呢？

  Ruby Code:
  class Object
    def try(*a, &b)
      try!(*a, &b) if a.empty? || respond_to?(a.first)
    end
    
    def try!(*a, &b)
      if a.empty? && block_given?
        if b.arity.zero?
          instance_eval(&b)
        else
          yield self
        end
      else
        public_send(*a, &b)
      end
    end
  end

阅读源码可以发现，Rails扩展了`Object`对象，提供了`try`方法，可以接受多个参数和一个代码块。
参数为空或者调用者能够响应第一个参数时，就交给`try!`方法处理。

1. `try`方法首先判断参数为空和给定了代码块的情况
  * 如果代码块不带参数，则直接执行代码块
  * 如果代码块带参数则把调用者作为参数传递给代码块
2. 否则直接交给`public_send`方法处理,把第一个参数作为被调用的方法，其余的参数作为被调用方法的参数

`Rails`同时扩展了`NilClass`,调用`try`方法直接返回`nil`。

