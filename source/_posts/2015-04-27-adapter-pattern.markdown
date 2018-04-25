---
layout: post
title:  "【转载】适配器模式"
date:   2015-04-27
keywords: ["Ruby","设计模式"]
description: "适配器模式"
category: "design-pattern"
tags: ["Ruby","设计模式"]
---
适配器模式可以用于对不同的接口进行包装以及提供统一的接口，或者是让某一个对象看起来像是另一个类型的对象。在静态类型的编程语言里，我们经常使用它去满足类型系统的特点，但是在类似Ruby这样的弱类型编程语言里，我们并不需要这么做。尽管如此，它对于我们来说还是有很多意义的。

将一个类的接口转换成客户希望的另外一个接口。Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。——Gang of Four

假如你现在要管理游戏的服务器，而这个游戏有三个服务器，以服已经开放一段时间，二服和三服刚刚开放，现在你需要设计一个接口，要求传入不同的参数就能够获取到对应服的人数，如果传入的参数不合法，则返回-1。

首先定义一个父类，功能是统计在线人数。
    Ruby Code:
	class PlayerCount
	
		def server_name
			raise "You should override this method in subclass."
		end
		
		def player_count
			raise "You should override this method in subclass."
		end
	
	end
接着定义三个子类，非别对应一服、二服、三服，代码如下。
    Ruby Code:
	class ServerOne < PlayerCount
	
		def server_name
			"一服"
		end
		
		def player_count
			Utility.online_player_count(1)
		end
	
	end
	
	class ServerTwo < PlayerCount
	
		def server_name
			"二服"
		end
		
		def player_count
			Utility.online_player_count(2)
		end
	
	end
	
	class ServerThree < PlayerCount
	
		def server_name
			"三服"
		end
		
		def player_count
			Utility.online_player_count(3)
		end
	
	end
接着，定义一个XMLBuilder类，用来将不同服的数据封装成XML格式输出。
	Ruby Code:
	class XMLBuilder
	
	  def self.build_xml player
			builder = ""
			builder << "<root>"
			builder << "<server>" << player.server_name << "</server>"
			builder << "<player_count>" << player.player_count.to_s << "</player_count>"
			builder << "</root>"
	  end
	
	end
通过如下方式来查询各个服的玩家的数量。
	Ruby Code:
	XMLBuilder.build_xml(ServerOne.new)
	XMLBuilder.build_xml(ServerTwo.new)
	XMLBuilder.build_xml(ServerThree.new)
这看起来很好，但是因为一服已经运行了一段时间，已经有了查询玩家在线数量的方法了，使用的是ServerFirst这个类。当时写Utility.online_player_count()这个方法主要是为了针对新开的二服和三服，就没把一服的查询功能再重复做一遍。这种情况下可以使用适配器模式，这个模式就是为了解决接口之间不兼容的问题而出现的。其实适配器模式的使用非常简单，核心思想就是只要能让两个互不兼容的接口能正常对接就行了。上面的代码中，XMLBuilder中使用PlayerCount来拼装XML，而ServerFirst并没有继承PlayerCount，这个时候就需要一个适配器类来为XMLBuilder和ServerFirst之间搭起一座桥梁，毫无疑问，ServerOne就将充当适配器类的角色。修改ServerOne的代码，如下所示：
	Ruby Code:
	class ServerOne < PlayerCount
	
		def initialize
			@serverFirst = ServerFirst.new
		end
	
		def server_name
			"一服"
		end
		
		def player_count
			@serverFirst.online_player_count
		end
	
	end
需要值得注意的一点是，适配器模式不并是那种会让架构变得更合理的模式，更多的时候它只是充当救火队员的角色，帮助解决由于前期架构设计不合理导致的接口不匹配的问题。更好的做法是在设计的时候就尽量把以后可能出现的情况多考虑一些。

参考资料：

- [https://martin91.github.io/blog/2014/03/03/jie-du-rails-gua-pei-qi-mo-shi/](https://martin91.github.io/blog/2014/03/03/jie-du-rails-gua-pei-qi-mo-shi/ "jie-du-rails-gua-pei-qi-mo-shi")
- [http://blog.csdn.net/guolin_blog/article/details/9400153](http://blog.csdn.net/guolin_blog/article/details/9400153 " Ruby设计模式透析之 —— 适配器(Adapter)")
