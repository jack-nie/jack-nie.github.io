---
layout: post
title:  "理解Rails中的delegate模块"
date:   2015-10-29
keywords: ["rails","delegate"]
description: "rails delegate"
category: "Ruby"
tags: ["Ruby","Rails"]
---
Rails中有一个非常炫酷的`associations`特性能够帮助我们很方便的创建链式方法，它看起来像是这样子的：

    Ruby code:
    product.provider.name
    provider.address.city
    company.building.city

但是，这破坏了["得墨忒耳"](http://jack-nie.github.io/best-practices/understanding-the-law-of-demeter.html "得墨忒耳")法则，我们更希望通过如下方式进行调用：

    Ruby code:
    product.provider_name
    provider.address_city #or provider.city
    company.city

为了实现这样的调用方式，通常我们需要在对应的`model`中定义一些方法：
　　
    Ruby code:
    class Product < ActiveRecord::Base
      belongs_to :provider

      def provider_name
        provider.name
      end

      # More methods to access the provider's attributes
    end

    class Provider < ActiveRecord::Base
      has_many :products

      # Has a name attribute
    end

这是一个很好的解决方案，但是当我们有许多的属性时，就要定义更多的方法，这会增加代码的体积，也会更容易的引入bug。更进一步的我们可以引入ruby的动态特性。

    Ruby code:
    %w[name age address].each do |attr|
      define_method "provider_#{attr}" do
        provider.send("#{attr}")
      end
    end

很酷的解决方案，但是仍然不够简洁，我们可以利用Rails给我们提供的轮子`Delegate`提供更加优雅的解决方案。

    Ruby code:
    class Product < ActiveRecord::Base
      belongs_to :provider

      delegate :name, to: :provider, prefix: true
    end

    class Provider < ActiveRecord::Base
      has_many :products

      # Has a name attribute
    end

这样，我们就可以实现前文所述的调用方式了，关于`Delegate`,必须以hash结尾。
若定义了`prefix: true`,则需要在调用时加上委托对象前缀，当然也可以自定义前缀。同时也提供了一个`allow_nil`的选项，如果定义为`true`,则被委托的对象不存在时，不会抛出异常。除此之外，还提供对实例变量、类变量、常量甚至类本身的代理。需要指出的是如果对实例变量委托，那么即使定义了`allow_nil: true`，也会抛出异常。需要了解更多，在[文档](http://apidock.com/rails/Module/delegate "文档")中有更加详细的说明。

参考文献:

- [Ruby on Rails Delegate : Don’t break the Law of Demeter !](http://samurails.com/tutorial/rails-delegate-dont-break-the-law-of-demeter "Ruby on Rails Delegate : Don’t break the Law of Demeter !")
- [delegate ](http://apidock.com/rails/Module/delegate "delegate")
- [Understanding Ruby and Rails: Delegate](http://simonecarletti.com/blog/2009/12/inside-ruby-on-rails-delegate/ "Understanding Ruby and Rails: Delegate")
- [delegate源码 ](https://github.com/rails/rails/blob/8ba491acc31bf08cf63a83ea0a3c314c52cd020f/activesupport/lib/active_support/core_ext/module/delegation.rb#L104 "delegate")
