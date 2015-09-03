---
layout: post
title:  "【翻译】Ruby’s case statement – advanced techniques"
date:   2015-08-02
keywords: ["Ruby"]
description: "Ruby’s case statement – advanced techniques"
category: "Ruby"
tags: ["Ruby"]
---
{% include JB/setup %}
没有什么比case语句更加的简单和无聊了。这是沿袭自C语言，通常被用来代替一系列的if语句。实际上，Ruby中的case语句比你想象的要更加的丰富和更加的复杂，让我们来看一下下面的这个例子。

    {% highlight ruby %}
    Ruby Code:
    case "Hi there"
    when String
      pute "case statements match class"
    end
    # outputs: "case statements match class"
    {% endhighlight %}
这个例子表明case语句不仅匹配一个条目的值，并且还能匹配其class。这可能是因为Ruby内部使用了===操作符。

###快速浏览===操作符

当你用Ruby写下x===y，你就是在询问y是属于x所代表的组吗？这是一个非常通用的语句，它的定义会依据你所处理的组别而改变。   

    {% highlight ruby %}
    Ruby Code:
    # Here, the Class.===(item) method is called, which returns true if item is an instance of the class 
 
    String === "hello" # true
    String === 1 # false
    {% endhighlight %}

字符串，正则表达式以及区间都定义了各自的===方法，表现上大抵和你的预期相符合，你甚至可以在你自己的类中增加一个===方法。现在我们懂得了这些，我们可以变出各种各样的戏法。

###case语句中的区间匹配

你之所以能够在case语句中使用区间，是因为range === n 返回的是range.include?(n)。我为什么这肯定呢？这是因为[文档](http://ruby-doc.org/core-2.2.0/Range.html#method-i-3D-3D-3D "文档")中已经写明了。

    {% highlight ruby %}
    Ruby Code:
    case 5
    when (1..10)
      puts "case statements match inclusion in a range"
    end
     
    # outputs "case statements match inclusion in a range"
    {% endhighlight %}

###case语句中的正则表达式匹配

在case语句中使用正则也是可能的，因为/regexp/ === "string"返回true，当且仅当字符串匹配该正则表达式时。正则表达式的[文档](http://ruby-doc.org/core-2.2.0/Regexp.html#method-i-3D-3D-3D "文档")对其做出了解释。
    {% highlight ruby %}
    Ruby Code:
    case "FOOBAR"
    when /BAR$/
      puts "they can match regular expressions!"
    end
     
    # outputs "they can match regular expressions!"
    {% endhighlight %}

###匹配proc和lambda

这是非常奇怪的一类，当你使用Proc#===(item)，就和Proc#call(item)一样。这是定义的[文档](http://ruby-doc.org/core-2.2.0/Proc.html#method-i-3D-3D-3D "文档")，这个文档的意思是你可以通过动态匹配在case语句中使用proc和lambda。
    {% highlight ruby %}
    Ruby Code:
    case 40
    when -> (n) { n.to_s == "40" }
      puts "lambdas!"
    end
     
    # outputs "lambdas"
    {% endhighlight %}

###编写你自己的匹配类

就像我上文提到的那样，在你自己的类中增加一个自定义的case行为就和你定义一个===方法一样简单。使用这项技术能够讲一系列复杂的条件逻辑拆分成多个较小的类。
    {% highlight ruby %}
    Ruby Code:
    class Success
      def self.===(item)
        item.status >= 200 && item.status < 300
      end
    end
     
    class Empty
      def self.===(item)
        item.response_size == 0
      end
    end
     
    case http_response
    when Empty
      puts "response was empty"
    when Success
      puts "response was a success"
    end
    {% endhighlight %}


###Extra
另外还可以在when语句中增加多个可以匹配的值。

    {% highlight ruby  %}
    Ruby code:
    print "Enter your grade: "
    grade = gets.chomp
    case grade
    when "A", "B"
      puts 'You pretty smart!'
    when "C", "D"
      puts 'You pretty dumb!!'
    else
      puts "You can't even use a computer!"
    end
    {% endhighlight %}
  
原文：

- [Ruby’s case statement – advanced techniques](http://blog.honeybadger.io/rubys-case-statement-advanced-techniques "Ruby’s case statement – advanced techniques")

