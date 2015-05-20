---
layout: post
title:  "Preload, includes, eagerload and joins"
date:   2015-05-20
keywords: ["Ruby"]
description: "Preload, includes, eagerload and joins"
category: "Ruby"
tags: ["Ruby"]
---
{% include JB/setup %}
Rails提供了四种不同的方式来加载关联数据，本文将详细解析这四种方式的异同。

###Preload

方法`preload`通过两条sql查询来加载关联数据。
  {% highlight ruby %}
  Ruby Code:
  User.preload(:posts).to_a
  
  # =>
  SELECT "users".* FROM "users"
  SELECT "posts".* FROM "posts"  WHERE "posts"."user_id" IN (1)
  {% endhighlight %}
上面的查询是方法`includes`默认的加载数据的方式。因为`preload`总是产生两条sql查询，所以我们不能够在`where`条件中使用`posts`表，下面的查询将导致一个错误。
  {% highlight ruby %}
  Ruby Code:
  User.preload(:posts).where("posts.desc='ruby is awesome'")
  
  # =>
  SQLite3::SQLException: no such column: posts.desc:
  SELECT "users".* FROM "users"  WHERE (posts.desc='ruby is awesome')

当然，我们可以通过如下方式来使用`where`条件。
  {% highlight ruby %}
  Ruby Code:
  User.preload(:posts).where("users.name='Neeraj'")
  
  # =>
  SELECT "users".* FROM "users"  WHERE (users.name='Neeraj')
  SELECT "posts".* FROM "posts"  WHERE "posts"."user_id" IN (3)
  {% endhighlight %}
###Includes

方法`includes`通过分离的sql查询加载关联数据时和`preload`的表现形式是一样的。但是`includes`比`preload`更加的智能，通过上面的例子可以看出通过`User.preload(:posts).where("posts.desc='ruby is awesome'")`查询语句使用`preload`产生了一个错误。下面来看一下使用`includes`的情况。
  {% highlight ruby %}
  Ruby Code:
  User.includes(:posts).where('posts.desc = "ruby is awesome"').to_a
  
  # =>
  SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, "posts"."id" AS t1_r0,
         "posts"."title" AS t1_r1,
         "posts"."user_id" AS t1_r2, "posts"."desc" AS t1_r3
  FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
  WHERE (posts.desc = "ruby is awesome")
  {% endhighlight %}
可以看到，`includes`通过`LEFT OUTER JOIN`将两条sql查询转换成一条单独的sql语句查询。默认情况下，`includes`将产生两条单独的查询。假定在某些情况下，你需要强制使`includes`只产生一条单独的查询语句，可以通过`references`来达到目标。
  {% highlight ruby %}
  Ruby Code:
  User.includes(:posts).references(:posts).to_a
  
  # =>
  SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, "posts"."id" AS t1_r0,
         "posts"."title" AS t1_r1,
         "posts"."user_id" AS t1_r2, "posts"."desc" AS t1_r3
  FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
  {% endhighlight %}
上面的例子中，只产生了一条sql查询。
###Eager load

立即加载使用`LEFT OUTER JOIN`产生一条sql语句来加载所有的关联数据。
  {% highlight ruby %}
  Ruby Code:
  User.eager_load(:posts).to_a
  
  # =>
  SELECT "users"."id" AS t0_r0, "users"."name" AS t0_r1, "posts"."id" AS t1_r0,
         "posts"."title" AS t1_r1, "posts"."user_id" AS t1_r2, "posts"."desc" AS t1_r3
  FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
  {% endhighlight %}
当在`where`或者`order`子句中使用了来自表`posts`中的属性时，includes会强制只使用一条sql语句。

###Joins

方法`joins`使用`inner join`来加载关联数据。
  {% highlight ruby %}
  Ruby Code:
  User.joins(:posts)
  
  # =>
  SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id"
  {% endhighlight %}
上面的案例中没有选择任何posts数据，上面的查询也有可能产生重复的结果，为了验证，让我们来创建一些示例数据。
  {% highlight ruby %}
  Ruby Code:
  def self.setup
    User.delete_all
    Post.delete_all
  
    u = User.create name: 'Neeraj'
    u.posts.create! title: 'ruby', desc: 'ruby is awesome'
    u.posts.create! title: 'rails', desc: 'rails is awesome'
    u.posts.create! title: 'JavaScript', desc: 'JavaScript is awesome'
  
    u = User.create name: 'Neil'
    u.posts.create! title: 'JavaScript', desc: 'Javascript is awesome'
  
    u = User.create name: 'Trisha'
  end
  {% endhighlight %}
利用上面的示例数据，如果执行`User.joins(:posts)`将会得到如下的结果。
  {% highlight ruby %}
  Ruby Code:
  #<User id: 9, name: "Neeraj">
  #<User id: 9, name: "Neeraj">
  #<User id: 9, name: "Neeraj">
  #<User id: 10, name: "Neil">
  {% endhighlight %}
可以通过使用`distinct`来消除重复的结果。
    {% highlight ruby %}
    Ruby Code:
    User.joins(:posts).select('distinct users.*').to_a
    {% endhighlight %}
如果想利用来自`posts`表中的属性，我们需要选择他们。
  {% highlight ruby %}
  Ruby Code:
  records = User.joins(:posts).select('distinct users.*, posts.title as posts_title').to_a
  records.each do |user|
    puts user.name
    puts user.posts_title
  end
  {% endhighlight %}
需要指出的一点是使用`joins`意味着如果你使用`user.posts`将会产生另外的一条SQL查询。

原文：

- [Preload, Eagerload, Includes and Joins](http://blog.bigbinary.com/2013/07/01/preload-vs-eager-load-vs-joins-vs-includes.html "Preload, Eagerload, Includes and Joins")

扩展阅读：


- [3 ways to do eager loading (preloading) in Rails 3 & 4](http://blog.arkency.com/2013/12/rails4-preloading/ "3 ways to do eager loading (preloading) in Rails 3 & 4")
