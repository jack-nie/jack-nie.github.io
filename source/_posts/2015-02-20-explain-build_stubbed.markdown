---
layout: post
title:  "【翻译】build_stubbed详解--Factory Girl"
date:   2015-02-20
keywords: ["Rspec", "Rails", "测试"]
description: "build_stubbed详解"
category: "Test"
tags: ["Rspec", "Rails", "测试"]
---
build_stubbed是在FactoryGirl中引入的一个方法，build_stubbed和build都是用来创建实例并分配属性。区别是build_stubbed使得对象看起来像是被持久化了，实际上并没有。它采用build_stubbed策略来创建关联，并且提供了少量的同数据库进行交互的方法，如果调用他们，这些方法会被触发。通过这种策略，可以加快测试的速度，同时减少对数据库的依赖。

但是如果创建的对象有关联的对象的话，被关联的对象是会被持久化到数据中的。为了避免这一情况的
发生可以将不必要的的关联设置为nil。
    Ruby Code:
	assoiation :store
	assoiation :area

	FactoryGirl.build_stubbed(:store, company: fg_company, area: nil)
通过这种方式就不会额外的创建被关联的数据了。如果创建时有很多依赖，但是你忘记了将其中的一个
或者几个关联设置为nil，同样会往数据库中写入很多数据。这里给出解决方案：
1. 在FactoryGirl的定义中手动的删除assoiation。
2. 将所有的实例变量移入before(:all)代码块中。
    Ruby Code:
    before(:all) do
      @company = FactoryGirl.build_stubbed(:company)
      @store = FactoryGirl.build_stubbed(:store, company: company)
    end
或者采用下面的方案：
    Ruby Code:
	factory :post do
	  title # sequence
	  author # sequence, so the example is shorter
	  # stuff
	  after(:build) {|p| p.comments ||= create_list(:comment, 5, post: p) }
	  after(:stub) {|p| p.comments ||= build_stubbed_list(:comment, 5, post: p}
	end

	create(:post) # with auto comments
	create(:post, comments: some_other_comments) # with comments for a specific test
	build_stubbed(:post) # with auto comments and no DB
	build_stubbed(:post, comments: []) # a post with no comments, still not touching the DB
这种用法使得before(:all)仅仅被用作提供事物功能的一个骨架。这就导致了测试状态更加的依赖测试的顺序，这是应该避免的。当然，也可以通过在before/after :all 代码块中加入清除数据的逻辑来解决。

参考资料：

- [http://blog.spoolz.com/2012/07/09/small-revelation-factorygirl-build_stubbed-associations-and-let/](http://blog.spoolz.com/2012/07/09/small-revelation-factorygirl-build_stubbed-associations-and-let/ "small-revelation-factorygirl-build_stubbed-associations-and-let")
- [https://robots.thoughtbot.com/use-factory-girls-build-stubbed-for-a-faster-test](https://robots.thoughtbot.com/use-factory-girls-build-stubbed-for-a-faster-test "use-factory-girls-build-stubbed-for-a-faster-test")
