---
layout: post
title:  "Statement completion values"
date:   "2017-01-14"
keywords: ["JavaScript"]
description: "javaScript, statement completion value"
category: "JavaScript"
tags: ["JavaScript"]
---
{% include JB/setup %}

最近Paul Irish在Twitter上提了一个JavaScript的小问题，觉得很有趣，所以写篇文章来记录一下。

在浏览器的console中运行如下的JavaScript片段：

```
'omg'; var x = 4;
```
结果返回`omg`，看到这种情况，最先想到的就是变量提升，所以写了下面的代码：

```
var x; 'omg'; x = 4;
```
结果返回`4`，这就让人很迷惑了。于是请教了下公司的前端，得到的答案是 `var x = 4`被提升了，但是我并不认同。
所以写了下面的代码加以验证：

```
console.log(y, y === undefined); 'omg'; var y = 4;
```
结果输出`undefined true`，返回`omg`，事实证明只有`var y`被提升了。
最后Javascript的创建者Brendan Eich回答了这个问题：

> It's not a return value, rather a statement completion value.

但是什么是`statement completion value`，和`return value`又有什么区别呢？

### statement completion value

直观的看， `statement completion value`是执行一段代码块所得到的值，例如：

```
>Math.random()
<0.853512441173391
>a = 5
<5
>var b
<undefined
```

实际上，目前捕获`statement completion value`的唯一方式是使用`eval`:

```
>eval("Math.random()")
<0.03151934138190282
>eval("a = 5")
<5
>eval("var b")
<undefined
```

需要指出的是并不是所有的表达式都有`completion value`, 例如`for(var x = 42;false;);`。

那么JavaScript又是如何处理`statement lists`的呢？[ECMA规范](http://www.ecma-international.org/ecma-262/6.0/#sec-block-runtime-semantics-evaluation)有下面一段话：

>The value of a StatementList is the value of the last value producing item in the StatementList.

依据上面的规范，下面的列表都会返回 1:

```
eval("1;;;;;")
eval("1;{}")
eval("1;var a;")
```
依此可以得到结论：一个`statement lists`总是返回最后一个非空表达式的值。所以就能够很好的解释本文开头提出的问题了。
`omg;`的`completion value`是`omg`, `var x = 4`的`completion value`是`undefined`,他们组合到一起就返回`omg`。

ES7已经有提案要支持`statement completion value`的捕获了，具体的语法是：

```
var x = do {
  ...
}
```

### 参考文献

- [What's a statement completion value in JavaScript?](http://www.mattzeunert.com/2017/01/10/whats-a-statement-completion-value-in-javascript.html)
- [ECMA262](http://www.ecma-international.org/ecma-262/6.0/#sec-block-runtime-semantics-evaluation)
- [You Don't Know JS: Types & Grammar](https://github.com/getify/You-Dont-Know-JS/blob/master/types%20%26%20grammar/ch5.md)
