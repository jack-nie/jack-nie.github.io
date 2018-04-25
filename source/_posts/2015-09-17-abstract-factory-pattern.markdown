---
layout: post
title:  "抽象工厂模式"
date:   2015-09-17
keywords: ["abstract factory"]
description: "abstract factory"
category: "design-pattern"
tags: ["Ruby"]
---

### 意图

抽象工厂模式提供一系列相关或者相互依赖的对象的接口，而无需指定它们具体的类。客户类不需要直接构建对象，它会调用该接口提供的方法。

### 实现

下面是一个具体的实现。
   
    class AbstractMazeFactory
      def make_maze
        raise NotImplementedError, "You should implement this method"
      end

      def make_wall
        raise NotImplementedError, "You should implement this method"
      end

      def make_room
        raise NotImplementedError, "You should implement this method"
      end
    end

    class MazeFactory < AbstractMazeFactory
       def make_maze
         Maze.new
       end

       def make_wall wall
         wall.camelize.constantlize.new
       end

       def make_room
         Room.new
       end
    end

    class Room
      def room_number
        p "i am room number 1"
      end
    end

    class Maze
      def maze
        p "maze builded complete"
      end
    end

    class AbstractWall
      def feature
        raise NotImplementedError, "You should implement this method"
      end
    end

    class IronWall < AbstractWall
      def feature
        p "I am an Iron Wall"
      end
    end

    class BrickWall < AbstractWall
      def feature
        p "I am a Brick Wall"
      end
    end

    class ClayWall < AbatractWall
      def feature
        p "I am a Clay Wall"
      end
    end

    class Client
      def make_maze
        maze_factory = MazeFactory.new
        room = maze_factory.make_room
        room.room_number
        iron_wall = maze_factory.make_wall "iron_wall"
        iron_wall.feature
        brick_wall = maze_factory.make_wall "brick_wall"
        brick_wall.feature
        clay_wall =  maze_factory.make_wall "clay_wall"
        clay_wall.feature
        maze = maze_factory.make_maze
        maze.maze
      end
    end
  在上面的实现中，我们首先中创建一个`AbstractMazeFactory`,其中定义了必须被子类复写的方法。然后创建一个`MazeFactory`类，
  复写父类所有的方法，并返回`Room`,`Maze`及`Wall`子类的实例。最后利用`Client`类构造出一个迷宫。

### 适用性

* 一个系统要独立于它的产品的创建、组合或者实现时。
* 一个系统要由多个产品系列中的一个来配置时。
* 当你强调一系列相关的产品的对象的设计以便进行联合使用时。
* 当你提供一个产品类库，而只想显示它的接口而不是实现时。

### 优缺点

* 它分离了具体的类。
* 它使得易于交换产品序列。
* 它有利于产品的一致性。
* 它难以支持新种类的产品

参考文献:

- [Creational Design Patterns](https://practicingruby.com/articles/creational-design-patterns "Creational Design Patterns")
- [Design Patterns in Ruby: Abstract Factory](http://www.devinterface.com/blog/en/2010/06/design-patterns-in-ruby-abstract-factory/ "Design Patterns in Ruby: Abstract Factory")
- [计算机科学丛书：设计模式 可复用面向对象软件的基础](http://www.amazon.cn/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6%E4%B8%9B%E4%B9%A6-%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E5%8F%AF%E5%A4%8D%E7%94%A8%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%BD%AF%E4%BB%B6%E7%9A%84%E5%9F%BA%E7%A1%80-Erich-Gamma/dp/B001130JN8 "计算机科学丛书：设计模式 可复用面向对象软件的基础")
- [The factory design pattern explained by example](https://www.binpress.com/tutorial/the-factory-design-pattern-explained-by-example/142 "The factory design pattern explained by example")
