---
layout: post
title:  "【翻译】利用Nginx加速Rails下载"
date:   2015-10-24
keywords: ["nginx","rails"]
description: "Accelerated Rails Downloads with NGINX"
category: "ruby"
tags: ["Rails"]
---
{% include JB/setup %}
Rails有一个选项可以用来启用`X-Accel-Redirect`，但是这并不是全部。为了能够使其工作，还需要对Nginx进行一些配置，这稍微有点繁琐。下面说一下我是怎么做的。

###问题：高效的传送文件

扩展Rails应用全部都是有关与减少请求数，使得等待的时间越短越好。将耗费时间的请求使用异步的方式在后台运行，分发静态资源请求到CDN,使用缓存降低响应时间。

但是有一类情景需要耗费比较长的时间，但是并不适合与一上的任意一种情况，那就是通过Rails应用传递大文件。你可能需要解决这个问题，因为文件需要加密(例如Rails session),或者是动态生成的。在这些情景下，你不能提前将其上传到CDN,异步传送可能会造成不好的用户体验。

那么如何才能传送大文件的过程中不阻塞Rails应用呢？

###Nginx和`SEND_FILE`是如何工作的？

如果你正在使用Nginx,那么解决方案就是使用`X-Accel-Redirect`,它是这样工作的。

1. Rails控制器调用`send_file`方法，准备一个将要被下载的文件。对于一个生成好的文件，通常会被放在`<rails root>/tmp`文件夹下。
2. Rack检测`X-Accel-Mapping`头部是否存在于请求中。如果存在，它就通过这种映射将`send_file`路径转换成Nginx能够理解的URI。
3. Rack发送一个包含`X-Accel-Redirect`头的空响应给Nginx，这个头部告诉Nginx"请加载这个URI，并将其作为响应"。从Rails应用的角度来讲，在这一点上响应已经完成了，并且能够继续接受其他的请求。
4. Nginx完成当前请求并处理一段时间，内部通过`X-Accel-Redirect`头部提供的URI进行重定向。
5. Nginx查询配置文件(例如location指令)来找到URI指向的文件。假定该文件存在，它将会高效的将文件传送给客户端。

如你所见，一个简单的`sned_file`只有在一系列的步骤都设置正确才会成功，下面是让其正确工作的步骤。

###设置Rails

首先，Rails需要被告知使用`X-Accel-Redirect`特性。否则Rails自身将会使用一个更加基础也更加慢的方式来处理I/O流。

为了启用Nginx加速，反注释掉`config/environments/production.rb`中的代码：

    {% highlight ruby%}
    config.action_dispatch.x_sendfile_header = "X-Accel-Redirect" # for NGINX
    {% endhighlight %}
    
###设置Nginx路径来伺服文件

如果不做配置，Nginx并不能伺服文件系统中的任何文件，我们需要设置`document root`和一个URI来访问到将要传送的文件。为了防止和Rails的路由产生冲突，我选择使用`__send_file_accel`作为URI。

`document root`需要仔细选择以让Rails应用能够轻松的访问到。我的项目使用了 Capistrano, Rails应用被部署在`/home/deployer/apps/my_app/releases`,所以很自然的root路径为：

    {% highlight ruby %}
    code:
    # Allow NGINX to serve any file in /home/deployer/apps/my_app/releases
    # via a special internal-only location.
    location /__send_file_accel {
        internal;
        alias /home/deployer/apps/my_app/releases;
    }
    {% endhighlight %}

###发送`X-Accel-Mapping`头

假设我们有以下的文件将要传送：


    {% highlight ruby %}
    /home/deployer/apps/my_app/releases/20151003173639/tmp/file.zip
    {% endhighlight %}
   
这意味者Nginx将会收到以下头部：

    {% highlight ruby %}
    X-Accel-Redirect: /__send_file_accel/20151003173639/tmp/file.zip
    {% endhighlight %}

Rails(更精确地将Rack)需要直到怎么进行转换，下面是通过Nginx发送给Rails的正确头部(比如将其放在Unicorn的反向代理文件中):

    {% highlight ruby %}
    code:
    proxy_set_header X-Sendfile-Type X-Accel-Redirect;
    proxy_set_header X-Accel-Mapping /home/deployer/apps/my_app/releases/=/__send_file_accel/;
    {% endhighlight %}

###测试

下面是一个通过Rails控制器传送文件的小例子：


    {% highlight ruby %}
    send_file Rails.root.join("tmp", "file.zip")
    {% endhighlight %}

在下载的过程中，你将会在Rails的`production.log`中看到如下的内容：

    {% highlight ruby %}
    code:
    Sent file /home/deployer/apps/nginx-test/releases/20151011032644/tmp/file.zip (0.2ms)
    Completed 200 OK in 2ms (ActiveRecord: 0.0ms)
    {% endhighlight %}

在响应头中，你将会看到是Rails还是Nginx在传送文件的线索。如果响应头部中包含`X-Request-Id`,这就意味这是Rails在伺服。

    {% highlight ruby %}
    code:
    X-Request-Id: a4d62cdb-569b-4120-b1ff-e2adbf77039a
    X-Runtime: 0.005559
    {% endhighlight %}

如果所有的配置都正确的话，你将不会看到以上内容，也就是说Nginx在传递文件，而不是Rails。

这就是所有的步骤，你现在将传递文件的责任从Rails传递给Nginx了。


参考文献:

- [Accelerated Rails Downloads with NGINX](https://mattbrictson.com/accelerated-rails-downloads?utm_source=rubyweekly&utm_medium=email "Accelerated Rails Downloads with NGINX")
