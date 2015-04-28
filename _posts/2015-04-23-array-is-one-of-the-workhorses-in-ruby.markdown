---
layout: post
title:  "Ruby中的Array详解"
date:   2015-04-23
keywords: ["Ruby","array"]
description: "Ruby中的Array详解"
category: "Ruby"
tags: ["Ruby"]
---
{% include JB/setup %}
在这篇文章中，我将深入探讨`Arrays`。数组中有很多方法我们每天都用，但是也有一部分方法我们甚至不知道他们的存在。
###基础
关于数组有一个非常无聊的定义，但是我们必须记住：数组的下标是从0开始的。一个负数下标表明从数组的最后开始计数--也就是说-1代表数组的最后一个数，-2代表倒数第2个数，以此类推。

那么问题来了,我们要怎样去创建一个数组呢？
        {% highlight ruby%}
 Ruby code:
		array = Array.new 
		 => []
        
        {% endhighlight %}
或者
        {% highlight ruby%}
 Ruby code:
		array = []  
		 => []
        {% endhighlight %}
数组可以包含各种类型的对象，整数、字符串、浮点数等-数组并不关心类型。
        {% highlight ruby%}
 Ruby code:
		array = [1, 2, "Super Mario", 2.4, "Batman", 100]  
		 => [1, 2, "Super Mario", 2.4, "Batman", 100]
		 {% endhighlight %}
或者

        {% highlight ruby%}
 Ruby code:		
        Array.new(3)  
		 => [nil, nil, nil] 
		 {% endhighlight %}
或者

        {% highlight ruby%}
 Ruby code:		
        Array.new(3, true)  
		 => [true, true, true]
		 {% endhighlight %}
就像我们所熟知的，Ruby是一个完全的面向对象的语言，所以arrays也表现为对象。
###探索的越深就越有趣
让我们从用的最多的迭代方法`#each`开始。这是一个我们过去见到过很多次并且在将来也会遇到很多次的方法。`Array`和`Range`类都有`#each`方法，通常`#each`会遍历每一个对象，并将其作为参数传入到block中，返回值为原始的集合。
        {% highlight ruby%}
 Ruby code:
		[1, 3, 4, 1, 2].each {|n| puts "#{n}! "}
		1!  
		3!  
		4!  
		1!  
		2!  
		 => [1, 3, 4, 1, 2]
		 {% endhighlight %}
还有一个情况是如果想知道数组元素的位置，可以用`#each_with_index`。
        {% highlight ruby%}
 Ruby code:
	    %w{Ruby is cool and provides us 
	    with another syntax arrays}.each_with_index {|item, index| puts "position
	    #{index} for #{item}"}
	    position 0 for Ruby  
	    position 1 for is  
	    position 2 for cool  
	    position 3 for and  
	    position 4 for provides  
	    position 5 for us  
	    position 6 for with  
	    position 7 for another  
	    position 8 for syntax  
	    position 9 for for  
	    position 10 for arrays  
	     => ["Ruby", "is", "cool", "and", "provides", 
	         "us", "with", "another", "syntax", "for", 
	         "arrays"]
	         {% endhighlight %}
假设我们要区分偶数与基数，解决这个问题有许多种方法，但是`#select`是相对比较简洁的方法。
        {% highlight ruby%}
 Ruby code:
		%w{1 2 3 4 5 6 7 8 9 10 11 112 123 1432}.select { |num| num%2==0 }
		 => [] 
		 {% endhighlight %}
但是结果并不是像我们预期的那样，为什么呢？答案是非常简单的，让我们先来看一看`%w{1 2 3 4 5 6 7 8 9 10 11 112 123 1432}`返回的是什么。
        {% highlight ruby%}
 Ruby code:
		%w{1 2 3 4 5 6 7 8 9 10 11 112 123 1432}
		 => ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11", "112", "123", "1432"] 
		 {% endhighlight %}
问题很明显，我们不能对字符串应用除法运算，实际上我们不能对字符串做任何数学运算。
        {% highlight ruby%}
 Ruby code:
		%q => Single quoted string
		%Q or % => Double quoted string
		%w or %W => Array of tokens
		%r => Regular Expression
		%x => Shell Command
		%i => Array of Symbols
		{% endhighlight %}
为了避免这种情况让我们回到`[...]`,这样就不会再次踩坑。
        {% highlight ruby%}
 Ruby code:
		[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 112, 123, 1432].select { |num| num%2==0 }
		 => [2, 4, 6, 8, 10, 112, 1432]
		 {% endhighlight %}

**没有数据操作我们将变成什么?**举例来说，我们有很多用户，然后需要根据用户名来自动生成邮件地址，`#map`在这里将充当救世主的角色。
        {% highlight ruby%}
 Ruby code:
		users = ["Dalton Brown", "Ralph Braun", "Keara Mueller", "Suzanne Schuppe", "Trisha Batz"]  
		users.map { |user| user.downcase.tr(" ", "_") + "@your_site.com" }
		{% endhighlight %}
得到的结果是：
        {% highlight ruby%}
 Ruby code:
	    => ["dalton_brown@your_site.com",  "ralph_braun@your_site.com", 
	      "keara_mueller@your_site.com", "suzanne_schuppe@your_site.com", 
	      "trisha_batz@your_site.com"]
	      {% endhighlight %}
###鲜为人知的方法###
请记住`#uniq`和`#uniq`这两个方法之间的区别。`#uniq`返回一个去掉了重复元素的新数组，但是`#uniq！`会直接从原数组中删除重复的元素。
        {% highlight ruby%}
 Ruby code:
		example_array = ["x", "z", "z", "z", "a", "b", "b"] 
		> example_array.uniq
		 => ["x", "z", "a", "b"] 
		> example_array
		 => ["x", "z", "z", "z", "a", "b", "b"]
		 {% endhighlight %}
和
         {% highlight ruby%}
  Ruby code:
		> example_array.uniq!
		 => ["x", "z", "a", "b"] 
		> example_array
		 => ["x", "z", "a", "b"]
		 {% endhighlight %} 
你想要对数组中的元素进行分组么？那么`#group将`很适合你。
         {% highlight ruby%}
  Ruby code:
		users.group_by{|name| name.length}  
		 => {12=>["Dalton Brown"], 11=>["Ralph Braun", "Trisha Batz"], 13=>["Keara Mueller"], 15=>["Suzanne Schuppe"]} 
		 {% endhighlight %}
还有其它的一些非常有用的方法如`|`和 `&`。
        {% highlight ruby%}
 Ruby code:
		milan_colors = %w{red black}
		liverpool_colors = %w{white red}
		milan_colors | liverpool_colors
		=> ["red", "black", "white"] 
		milan_colors & liverpool_colors
		=> ["red"]
		{% endhighlight %} 
当然还不能忘了范围方法，我是这么称呼的~~
        {% highlight ruby%}
 Ruby code:
		2.1.2 :001 > a = [1, 2, 3, 4, 5]
		=> [1, 2, 3, 4, 5] 
		2.1.2 :002 > a[0..-3]
		=> [1, 2, 3] 
		2.1.2 :003 > a[0...-3]
		=> [1, 2]
		{% endhighlight %}

参考资料：</br>
[http://blog.diatomenterprises.com/arrays-are-the-workhorse-of-ruby/](http://blog.diatomenterprises.com/arrays-are-the-workhorse-of-ruby/)        