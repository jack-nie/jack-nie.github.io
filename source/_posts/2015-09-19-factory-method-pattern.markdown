---
layout: post
title:  "工厂方法模式"
date:   2015-09-19
keywords: ["factory method", "工厂方法模式", "ruby"]
description: "factory method"
category: "design-pattern"
tags: ["Ruby"]
---

### 意图

定义一个用于创建对象的接口，让子类决定实例化哪一个类，Factory Method使一个类的实例化延迟到其子类。

工厂方法模式是面向对象设计中最为常用的模式，这种模式是创建对象的最好方式之一。在工厂方法模式中，我们创建对象不需要向客户暴露创建的逻辑，转而使用通用的接口去引用一个新创建的对象。


### 实现

下面是一个具体的实现。
   
      class Car 
        def run
          raise NotImplementedError, "must implement run method in subclass"
        end
      end

      class BMW < Car
        def run
          p "BMW on the way!"
        end
      end

      class Audi < Car
        def run
          p "Audi on the way!"
        end
      end

      class CarFactory
        def make_car car
          car.safe_constantize.new
        end
      end

      class DriveCar
        def drive
          car = CarFactory.new.make_car "BMW"
          car.run
        end
      end
   在上面的实现中，我们首先中创建一个`Car`, 其中定义了必须被子类复写的方法。然后创建两个子类，`BMW`, `Audi`类。这两个类都覆写了父类`run`方法, 然后定义一个`CarFactory`用来返回特定的车辆实例。工厂方法不在将与特定应用有关的类绑定到代码中。

### 适用性

* 当一个类不知道它所要创建的对象的类的时候。
* 当一个类希望由它的子类来指定要创建的对象的时候。
* 当类将创建的对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者的这一信息局部化的时候。

参考文献:

- [Creational Design Patterns](https://practicingruby.com/articles/creational-design-patterns "Creational Design Patterns")
- [计算机科学丛书：设计模式 可复用面向对象软件的基础](http://www.amazon.cn/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6%E4%B8%9B%E4%B9%A6-%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E5%8F%AF%E5%A4%8D%E7%94%A8%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%BD%AF%E4%BB%B6%E7%9A%84%E5%9F%BA%E7%A1%80-Erich-Gamma/dp/B001130JN8 "计算机科学丛书：设计模式 可复用面向对象软件的基础")
- [The factory design pattern explained by example](https://www.binpress.com/tutorial/the-factory-design-pattern-explained-by-example/142 "The factory design pattern explained by example")
