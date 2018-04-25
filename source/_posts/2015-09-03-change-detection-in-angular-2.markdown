---
layout: post
title:  "【翻译】Angular2 变更检测"
date:   2015-08-24
keywords: ["angular2", "JavaScript"]
description: "Angular2 core concepts"
category: "JavaScript"
tags: ["JavaScript"]
---
本文讲深入探讨Angular2中的变更检测系统。

### 概览

一个Angular2的应用是由组件组成的树。
![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo1_1280.png)
一个Angular2应用是一个反应式的系统，变更检测是其核心。
![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo2_1280.png)
每一个组件都有一个变更检测器，用来检测在末班中定义的绑定。一个绑定的例子：`{{todo.text}}`和`[todo]='t'`。变更检测器使用深度优先策略遍历绑定。在Angular2中并没有一个通用的机制来实现双向绑定（但是你仍然可以实现双向绑定和`ng-model`，你可以通过阅读["这篇"](http://victorsavkin.com/post/119943127151/angular-2-template-syntax)文章了解更多)。这也是为什么变更检测是没有环的树，这使得该系统拥有更好的性能。更重要的是Angular2能够保证更好的预测系统的行为和更容易推理。

Angular2到底有多快？
变更检测器默认遍历树的每一个节点以检测其是否变化，这种行为会在每一个浏览器事件发生时被触发。虽然看起来性能非常的低，但是实际上Angular2能够在几毫秒的时间内进行成百上千的简单检测（具体的数量依赖于不同的浏览器平台）。怎么实现这一令人印象深刻的结果是另一篇文章将要探讨的内容。

因为Javascript并没有提供对象变化突变保障，所以Angular2表现的很保守，每次都会进行所有的检测。但是我们知道可以通过使用不可变对象或者可观察对象来持有属性。Angular2以前不可以利用这种特性，现在可以了。

### 不可变对象
如果一个组件仅仅依赖与其绑定，并且绑定是不可变的，那么该组件只有在其中的一个绑定发生了变化时才变化。因此，我们可以跳过变更探测树的子树直到该事件发生。当事件发生时，我们只用检测子树一次，然后就关闭它直到下一次变化发生。

![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo3_1280.png)

如果我们非常激进的处理不可变对象，大多数时间变更检测树的很大一部分都可以被关闭。
![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo4_1280.png)
实现这一功能，只需要将变更检测的策略设置为`ON_PUSH`。

    @Component({changeDetection:ON_PUSH})
    class ImmutableTodoCmp {
      todo:Todo;
    }

### 可观察对象

如果一个组件仅仅依赖于其绑定，并且该绑定是一个可观察对象，那么该组件只有在绑定触发了一个事件时才会变化。所以我们能够跳过变更检测树的子树直到改事件发生。当改事件发生时，我们只用检测子树一次，然后关闭它直到下乙烯事件发生。

虽然看起来和不可变对象的例子很像，实际上他们之间是有很大的区别的。如果你拥有一个由拥有不可变绑定对象的组件组成的树，一个变化需要从根节点开始遍历组件。这并不是我们处理可观察对象的方式。

让我们使用一个小例子来说明这一问题。

    type ObservableTodo = Observable<Todo>;
    type ObservableTodos = Observable<Array<ObservableTodo>>;

    @Component({selector:’todos’})
    class ObservableTodosCmp {
      todos:ObservableTodos;
      //...
    }

模板示例：

    <todo *ng-for="var t of todos" todo = "t"></todo>

 ObservableTodoCmp:

    @Component({selector:’todo’})
      class ObservableTodoCmp {
        todo:ObservableTodo;
        //...
      }

如你所见，Todos组件只用有一个对由todos数组组成的可观察对象的引用，所以它检测不到单个todos组件的变化。

处理的方式是在可观察对象todo触发一个事件时，检查从根到该变化的todo组件的路径。变更检测系统能够确保这种事情的发生。

假设我们的系统仅仅使用可观察对象，当系统启动时，Angular2将会检测检测所有的对象。
![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo5_1280.png)
所以经过了第一步之后，状态将会变成如下所示的样子。
![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo6_1280.png)

让我们假设todo可观测对象触发了一个事件，那么系统状态装回转变成如下所示的样子。
![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo7_1280.png)

当检测完了App_ChangeDetector, Todos_ChangeDetector和Todo_ChangeDetector之后，将会回到如下所示的状态。
![Alt "angular2"](/assets/images/tumblr_njb2puhhEa1qc0howo6_1280.png)

假设这些变化极少发生，组件组成一颗平衡树，那么使用可观测对象将会使变更检测的复杂度由O(N)变为O(logN)，其中N是系统中绑定对象的数量。

### 可观测对象会造成级联更新么？
可观测对象的名声不太好，是因为他们会造成级联更新。任何一个有个大规模系统设计经验并且用到了可观测模型的开发者将会明白我在说什么。一个可观测对象的更新会造成一系列的可观测对象触发更新。沿着这条路经的某个view也会随着更新，这种系统会很难进行推测。

在Angular2中使用可观测对象将不会造成这种问题。通过一个可观察对象触发的事件只是标示从该组件到根的路径作为下次将要检测的路径。所以更新的顺序和是否使用客人观察对象没有关系。这是非常重要的，使得使用可观察对象变成一个非常简单的优化，不会改变你处理系统的方式。

### 是否需要到处使用可观察对象/不可变对象

不，你不需要。你可以在你的系统中部分的使用可观察对象（比如一些巨大的表），这一部分将会获得性能上的改进。更进一步，你可以任意组合不同类型的组件，并且获得他们所有的好处。例如，一个可观测组件可以包含一个不可变组件，同时自身也可以包含一个可观察对象。即使在这种情况下，对象检测系统会使得蔓延式变化所需要的检测的数量最小化。

### 没有特殊情况

对可变化对象和可观察对象的支持并没有固化在变更检测系统中。这种类型的组件并没有什么特殊之处，所以你可以书写你自己的指令，以一种更只能的方式来使用变更检测。例如，想象一下一条指令每个几秒就更新一下它的内容。

### 总结

* 一个Angular2应用是一个响应式的系统。
* 变更检测系统从根到叶子节点传播绑定。
* 不像Angular1.×，变化检测图是一个有向树。结果是，改系统拥有更好的性能和可预测。
* 默认情况下，变更检测系统遍历整个树，但是如果你使用不可变对象或者可观察对象，你将利用其优势，当他们真正变化时，你只需要检测树的一部分。
* 这些优化可以随意组合，不会破坏变更检测提供的保证。

原文：

- [CHANGE DETECTION IN ANGULAR 2](http://victorsavkin.com/post/110170125256/change-detection-in-angular-2 "CHANGE DETECTION IN ANGULAR 2")
