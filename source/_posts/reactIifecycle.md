---
title: react生命周期
date: 2018-10-10 14:28:44
tags:
---

# react生命周期

-----

## react 16版本新特性之前

​	初始化     更新     销毁

![](C:\Users\HP\Desktop\hexo\hexo\source\_posts\reactIifecycle.jpg)

### 初始化

***1.getDefaultProps()* **

```
设置默认的props，也可以用dufaultProps设置组件的默认属性.
```

**2.getInitialState() **

```
在使用es6的class语法时是没有这个钩子函数的，可以直接在constructor中定义this.state。此时可以访问this.props
```

**3.componentWillMount()**

```
组件初始化时只调用，以后组件更新不调用，整个生命周期只调用一次，此时可以修改state。在发布的新增特性中，这个生命周期要加上 UNSAFE_ 前缀，表明其不安全性，并将在未来版本将其移除
```

**4.render**

```
react最重要的步骤，创建虚拟dom，进行diff算法，更新dom树都在此进行。此时就不能更改state了。要不然会死循环。
```

**5.componentDidMount**

```
组件渲染之后调用，只调用一次。一般用来发送初始化数据请求。
```

### 更新

**6.componentWillReceiveProps(nextProps)**

```
组件初始化时不调用，组件接受新的props时调用。此生命周期可能会造成死循环。在发布的新增特性中，这个生命周期要加上 UNSAFE_ 前缀，表明其不安全性，并将在未来版本将其移除。可用新提供的 
static getDerivedStateFromProps(prevProps, prevState) 来替代。
```

**7.shouldComponentUpdate(nextProps, nextState)**

```
react性能优化非常重要的一环。组件接受新的state或者props时调用，我们可以设置在此对比前后两个props和state是否相同，如果相同则返回false阻止更新，因为相同的属性状态一定会生成相同的dom树，这样就不需要创造新的dom树和旧的dom树进行diff算法对比，节省大量性能，尤其是在dom结构复杂的时候
```

**8.componentWillUpdate(nextProps, nextState)**

```
组件初始化时不调用，只有在组件将要更新时才调用。在发布的新增特性中，这个生命周期要加上 UNSAFE_ 前缀，表明其不安全性，并将在未来版本将其移除
```

**9.render()**

```
组件渲染
```

**10.componentDidUpdate()**

```
组件初始化时不调用，组件更新完成后调用，此时可以获取dom节点。
```

### 卸载

**11.componentWillUnmount()**

```
组件将要卸载时调用，一些事件监听和定时器需要在此时清除。
```

## react 16版本新增特性

![](C:\Users\HP\Desktop\hexo\hexo\source\_posts\reactlifecycle2.png)

https://react.docschina.org/docs/react-component.html   

* 给`componentWillMount` `componentWillReceiveProps` `componentWillUpdate`生命周期加上`UNSAFE_`前缀，表明其不安全性，并将在未来版本将其移除  
* 官网文档指出使用这些生命周期的代码会在未来版本的`react`中更容易产生`bug`，尤其是对于异步渲染的版本
* 新增生命周期`static getDerivedStateFromProps(prevProps, prevState)`、`getSnapshotBeforeUpdate(prevProps, prevState)` 、`componendDidCatch(error, info)`

**1.constructor(props)**

```
React组件的构造函数将会在装配之前被调用。当为一个React.Component子类定义构造函数时，你应该在任何其他的表达式之前调用super(props)。否则，this.props在构造函数中将是未定义，并可能引发异常.
```

**2.static getDerivedStateFromProps(nextProps, prevState)**

```
组件实例化后和接受新属性时将会调用getDerivedStateFromProps。它应该返回一个对象来更新状态，或者返回null来表明新属性不需要更新任何状态。

在每次渲染之前都会调用，不管造成重新渲染的原因，不管初始挂载还是后面的更新都会调用，这一点和UNSAFE_componentWillReceiveProps不同（只有当父组件造成重新渲染时才调用），每次都应该返回一个对象作为state的更新，或者返回null表示不更新

它的使用场景一般为依赖于props的变化去做一些状态的更新,让我们能够根据props的变化去更新内部的状态，以前我们经常在componentWillReceiveProps中完成该操作

注意，如果父组件导致了组件的重新渲染，即使属性没有更新，这一方法也会被调用。如果你只想处理变化，你可能想去比较新旧值。

调用this.setState() 通常不会触发 getDerivedStateFromProps()。
```

**3.getSnapshotBeforeUpdate()**

```
getSnapshotBeforeUpdate()在最新的渲染输出提交给DOM前将会立即调用。它让你的组件能在当前的值可能要改变前获得它们。这一生命周期返回的任何值将会 作为参数被传递给componentDidUpdate()的第三个参数。
```

**4.componentDidCatch(error, info)**

```
如果一个组件定义了componentDidCatch生命周期，则他将成为一个错误边界(错误边界会捕捉渲染期间、在生命周期方法中和在它们之下整棵树的构造函数中的错误，就像使用了try catch，不会将错误直接抛出了，保证应用的可用性)

注意： 错误边界只捕捉树中发生在它们之下组件里的错误。一个错误边界并不能捕捉它自己内部的错误。
```

-----

### `forceUpdate()`

```
component.forceUpdate(callback)

默认情况，当你的组件或状态发生改变，你的组件将会重渲。若你的render()方法依赖其他数据，你可以通过调用forceUpdate()来告诉React组件需要重渲。

调用forceUpdate()将会导致组件的 render()方法被调用，并忽略shouldComponentUpdate()。这将会触发每一个子组件的生命周期方法，涵盖，每个子组件的shouldComponentUpdate() 方法。若当标签改变，React仅会更新DOM。

通常你应该尝试避免所有forceUpdate() 的用法并仅在render()函数里从this.props和this.state读取数据。
```

