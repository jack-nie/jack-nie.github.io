---
layout: post
title:  "React中的props和state的比较"
date:   "2017-01-17"
keywords: ["react"]
description: "javascript, react"
category: "JavaScript"
tags: ["JavaScript", "react"]
---
Props和state是相关的，一个组件的state经常可以变成子组件的props。props可以通过父组件的render方法的第二个参数传递给子组件，如果使用jsx，则可以通过这种方式。`<MyChild name={this.state.childsName} />`。父亲组件的state的值变成了自组件的`this.props.name`,从子组件的角度来看，name prop是不可改变的，如果它需要被改变，父组件必须只改变内部的state。`this.setState({ childsName: 'New name' });`,然后react会自动的将改变后的值传递给子组件，如果子组件需要改变他自己的name prop呢？这通常是通过子组件的事件和父组件的回调来实现的。子组件向外暴露一个时间，那么父组件就可以通过传递一个回调的handler来订阅这个事件。

```
<MyChild name={this.state.childsName} onNameChanged={this.handleName} />
```

子组件将新的名字作为一个参数传递给事件回调函数，然后父组件会在事件处理器中使用这个值来更新state。

```
handleName: function(newName) {
   this.setState({ childsName: newName });
}

```

### 参考文献

- [What is the difference between state and props in React?](https://stackoverflow.com/questions/27991366/what-is-the-difference-between-state-and-props-in-react)
- [Thinking in react](https://facebook.github.io/react/docs/thinking-in-react.html#step-4-identify-where-your-state-should-live)
- [props vs state](https://github.com/uberVU/react-guide/blob/master/props-vs-state.md)
