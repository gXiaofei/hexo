---
title: redux的几个API
date: 2018-07-03 15:05:18
tags:
---



链接： http://cn.redux.js.org/

# createStore

createStore(reducer, [preloadedState],enhancer)

创建一个Redux store来存放应用中的state，有且仅有一个store

参数 ：

 	1. reducer（function）：reducer函数，该函数有两个参数，当前的state和要处理的action，返回新的state
		2. [preloadedState]：初始化时的state
		3. enhancer（function）：Store enhancer 是一个组合 store creator 的高阶函数，返回一个新的强化过的 store creator。这与 middleware (中间件)相似，它也允许你通过复合函数改变 store 接口。 

返回值：(store): 保存了应用所有 state 的对象。改变 state 的惟一方法是 dispatch action。

```javascript
import { createStore } from 'redux'

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([action.text])
    default:
      return state
  }
}

let store = createStore(todos, ['Use Redux'])
// 应用中存在一个store
store.dispatch({
  type: 'ADD_TODO',
  text: 'Read the docs'
})

console.log(store.getState())
// [ 'Use Redux', 'Read the docs' ]
```

# store

store就是用来维持应用所有state的一个对象。改变store里面的state的唯一途径时对它dispatch一个action

`const store = createStore(reducer,[preloadedState],enhancer)`来创建store

store的方法

 - getState()
 - dispatch(action)
 - subscribe(listenter)
 - replaceReducer(nextReducer)

## getState()

返回当前的state，它与 store 的最后一个 reducer 返回值相同 

## dispatch(action)

分发action。是触发state变化的唯一途径

```javascript
import { createStore } from 'redux'
const store = createStore(todos, ['Use Redux'])

function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

store.dispatch(addTodo('Read the docs'))
store.dispatch(addTodo('Read about the middleware'))
```

## subscribe(listener)

添加一个变化监听器。每当 dispatch action 的时候就会执行，state 树中的一部分可能已经变化。你可以在回调函数里调用 `getState`来拿到当前 state。

参数：

	- listener(function)：用来监听执行的回调函数， 每当 dispatch action 的时候都会执行的回调。state 树中的一部分可能已经变化。你可以在回调函数里调用 `getState`来拿到当前 state。store 的 reducer 应该是纯函数，因此你可能需要对 state 树中的引用做深度比较来确定它的值是否有变化。 

返回值：(*Function*): 一个可以解绑变化监听器的函数 

```javascript
function select(state) {
  return state.some.deep.property
}

let currentValue
// 每次dispatch都会执行
function handleChange() {
  let previousValue = currentValue
  currentValue = select(store.getState()) // 当前的state
  // 进行对比
  if (previousValue !== currentValue) {
    console.log(
      'Some deep nested property changed from',
      previousValue,
      'to',
      currentValue
    )
  }
}

const unsubscribe = store.subscribe(handleChange)
// 返回一个可以解绑的function
unsubscribe()
```

## replaceReducer(nextReducer)

替换 store 当前用来计算 state 的 reducer。

这是一个高级 API。只有在你需要实现代码分隔，而且需要立即加载一些 reducer 的时候才可能会用到它。在实现 Redux 热加载机制的时候也可能会用到。

参数：

	- nextReducer(function)：store会使用的下一个reducer

# combineReducers

reducer函数可能是多个单独的函数。combineReducers辅助函数的作用是把一个由多个不同的reducer函数作为value的object，合并成一个最终的reducer函数，然后就可以对这个reducer调用createStore方法

合并后的 reducer 可以调用各个子 reducer，并把它们返回的结果合并成一个 state 对象。由 combineReducers() 返回的 state 对象，会将传入的每个 reducer 返回的 state 按其传递给 combineReducers() 时对应的 key 进行命名。 

参数：

	- reducers（）：一个对象，它的值(value)对应不同的reducer函数，这些reducer函数后面会被合并成一个

返回值：

​	(function)：一个调用reducers对象里所有reducer的reducer，并且构造一个与reducers对象结构相同的state对象

```javascript
// counter.js
export default function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}
//todos.js
export default function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    return state.concat([action.text])
  default:
    return state
  }
}
// index.js
import { combineReducers } from 'redux'
import todos from './todos'
import counter from './counter'

export default combineReducers({  // 合并上面两个reducer
  todos,
  counter
})
// App.js
import { createStore } from 'redux'
// 返回一个整合起来的reducer
import reducer from './reducers/index'

let store = createStore(reducer)
console.log(store.getState())
// {
//   counter: 0,
//   todos: []
// }

store.dispatch({
  type: 'ADD_TODO',
  text: 'Use Redux'
})
console.log(store.getState())
// {
//   counter: 0,
//   todos: [ 'Use Redux' ]
// }
```

# applyMiddleware(...middleware)

middleware可以让你包装store的dispatch方法来达到你想要的目的。多个middleware可以被组合在一起使用。形成middleware链

例如：redux-thunk支持dispatch function，以此让 action creator 控制反转。被 dispatch 的 function 会接收dispatch作为参数，并且可以异步调用它。这类的 function 就称为 *thunk*。另一个 middleware 的示例是redux-promise，它支持 dispatch 一个异步的 promise action，并且在 Promise resolve 后可以 dispatch 一个普通的 action。

参数：

	- ...middleware :  遵循 Redux *middleware API* 的函数。每个 middleware 接受`store`的`dispatch`和`getState`函数作为命名参数，并返回一个函数。该函数会被传入 被称为 `next` 的下一个 middleware 的 dispatch 方法，并返回一个接收 action 的新函数，这个函数可以直接调用 `next(action)`，或者在其他需要的时刻调用，甚至根本不去调用它。调用链中最后一个 middleware 会接受真实的 store 的`dispatch`方法作为 `next` 参数，并借此结束调用链。所以，middleware 的函数是 `({ getState, dispatch }) => next => action`。 

返回值：(*Function*) 一个应用了 middleware 后的 store enhancer。这个 store enhancer 的签名是 `createStore => createStore`，但是最简单的使用方法就是直接作为最后一个 `enhancer`参数传递给`createStore`函数。

 **自定义Logger Middleware**

```react
import { createStore, applyMiddleware } from 'redux'
import todos from './reducers'

function logger({ getState }) {
  return next => action => {
    console.log('will dispatch', action)

    // 调用 middleware 链中下一个 middleware 的 dispatch。
    let returnValue = next(action)

    console.log('state after dispatch', getState())

    // 一般会是 action 本身，除非
    // 后面的 middleware 修改了它。
    return returnValue
  }
}

let store = createStore(
  todos,
  ['Use Redux'],
  applyMiddleware(logger) // 传入中间件
)

store.dispatch({
  type: 'ADD_TODO',
  text: 'Understand the middleware'
})
// (将打印如下信息:)
// will dispatch: { type: 'ADD_TODO', text: 'Understand the middleware' }
// state after dispatch: [ 'Use Redux', 'Understand the middleware' ]
```

**使用Thunk Middleware 来做异步Action**

```react
import { createStore, combineReducers, applyMiddleware } from 'redux'
import thunk from 'redux-thunk'
import * as reducers from './reducers'

let reducer = combineReducers(reducers)
// applyMiddleware 为 createStore 注入了 middleware:
let store = createStore(reducer, applyMiddleware(thunk))

function fetchSecretSauce() {
  return fetch('https://www.google.com/search?q=secret+sauce')
}

// 这些是你已熟悉的普通 action creator。
// 它们返回的 action 不需要任何 middleware 就能被 dispatch。
// 但是，他们只表达「事实」，并不表达「异步数据流」
function makeASandwich(forPerson, secretSauce) {
  return {
    type: 'MAKE_SANDWICH',
    forPerson,
    secretSauce
  }
}

function apologize(fromPerson, toPerson, error) {
  return {
    type: 'APOLOGIZE',
    fromPerson,
    toPerson,
    error
  }
}

function withdrawMoney(amount) {
  return {
    type: 'WITHDRAW',
    amount
  }
}

// 即使不使用 middleware，你也可以 dispatch action：
store.dispatch(withdrawMoney(100))

// 但是怎样处理异步 action 呢，
// 比如 API 调用，或者是路由跳转？

// 来看一下 thunk。
// Thunk 就是一个返回函数的函数。
// 下面就是一个 thunk。
function makeASandwichWithSecretSauce(forPerson) {

  // 控制反转！
  // 返回一个接收 `dispatch` 的函数。
  // Thunk middleware 知道如何把异步的 thunk action 转为普通 action。
  return function (dispatch) {
    return fetchSecretSauce().then(
      sauce => dispatch(makeASandwich(forPerson, sauce)),
      error => dispatch(apologize('The Sandwich Shop', forPerson, error))
    )
  }
}

// Thunk middleware 可以让我们像 dispatch 普通 action
// 一样 dispatch 异步的 thunk action。
store.dispatch(makeASandwichWithSecretSauce('Me'))

// 它甚至负责回传 thunk 被 dispatch 后返回的值，
// 所以可以继续串连 Promise，调用它的 .then() 方法。
store.dispatch(makeASandwichWithSecretSauce('My wife')).then(() => {
  console.log('Done!')
})

// 实际上，可以写一个 dispatch 其它 action creator 里
// 普通 action 和异步 action 的 action creator，
// 而且可以使用 Promise 来控制数据流。
function makeSandwichesForEverybody() {
  return function (dispatch, getState) {
    if (!getState().sandwiches.isShopOpen) {

      // 返回 Promise 并不是必须的，但这是一个很好的约定，
      // 为了让调用者能够在异步的 dispatch 结果上直接调用 .then() 方法。
      return Promise.resolve()
    }

    // 可以 dispatch 普通 action 对象和其它 thunk，
    // 这样我们就可以在一个数据流中组合多个异步 action。
    return dispatch(makeASandwichWithSecretSauce('My Grandma'))
      .then(() =>
        Promise.all([
          dispatch(makeASandwichWithSecretSauce('Me')),
          dispatch(makeASandwichWithSecretSauce('My wife'))
        ])
      )
      .then(() => dispatch(makeASandwichWithSecretSauce('Our kids')))
      .then(() =>
        dispatch(
          getState().myMoney > 42
            ? withdrawMoney(42)
            : apologize('Me', 'The Sandwich Shop')
        )
      )
  }
}

// 这在服务端渲染时很有用，因为我可以等到数据
// 准备好后，同步的渲染应用。

import { renderToString } from 'react-dom/server'

store
  .dispatch(makeSandwichesForEverybody())
  .then(() => response.send(renderToString(<MyApp store={store} />)))

// 也可以在任何导致组件的 props 变化的时刻
// dispatch 一个异步 thunk action。

import { connect } from 'react-redux'
import { Component } from 'react'

class SandwichShop extends Component {
  componentDidMount() {
    this.props.dispatch(makeASandwichWithSecretSauce(this.props.forPerson))
  }

  componentWillReceiveProps(nextProps) {
    if (nextProps.forPerson !== this.props.forPerson) {
      this.props.dispatch(makeASandwichWithSecretSauce(nextProps.forPerson))
    }
  }

  render() {
    return <p>{this.props.sandwiches.join('mustard')}</p>
  }
}

export default connect(state => ({
  sandwiches: state.sandwiches
}))(SandwichShop)
```

# bindActionCreators(actionCreators, dispatch)

唯一会使用带bindActionCreators的场景时当你需要把action creator往下传到下一个组件上，却不想让这个组件觉察到Redux的存在，而且不希望把dispatch或Redux store传给它。

为方便起见，你也可以传入一个函数作为第一个参数，它会返回一个函数

**参数**

 - actionCreators(function object):一个action creator，或者一个value是action creator 的对象
 - dispatch(function): 一个由Store实例提供的dispatch函数

**返回值：** （Function or *Object*): 一个与原对象类似的对象，只不过这个对象的 value 都是会直接 dispatch 原 action creator 返回的结果的函数。如果传入一个单独的函数作为 `actionCreators`，那么返回的结果也是一个单独的函数。 

```react
// TodoActionCreators.js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  };
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  };
}
// SomeComponent.js
import { Component } from 'react';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';

import * as TodoActionCreators from './TodoActionCreators';
console.log(TodoActionCreators);
// {
//   addTodo: Function,
//   removeTodo: Function
// }

class TodoListContainer extends Component {
  constructor(props) { 
    super(props);

    const {dispatch} = props;

    // 这是一个很好的 bindActionCreators 的使用示例：
    // 你想让你的子组件完全不感知 Redux 的存在。
    // 我们在这里对 action creator 绑定 dispatch 方法，
    // 以便稍后将其传给子组件。

    this.boundActionCreators = bindActionCreators(TodoActionCreators, dispatch);
    console.log(this.boundActionCreators);
    // {
    //   addTodo: Function,
    //   removeTodo: Function
    // }
  }

  componentDidMount() {
    // 由 react-redux 注入的 dispatch：
    let { dispatch } = this.props;

    // 注意：这样是行不通的：
    // TodoActionCreators.addTodo('Use Redux')

    // 你只是调用了创建 action 的方法。
    // 你必须要同时 dispatch action。

    // 这样做是可行的：
    let action = TodoActionCreators.addTodo('Use Redux');
    dispatch(action);
  }

  render() {
    // 由 react-redux 注入的 todos：
    let { todos } = this.props;

    return <TodoList todos={todos} {...this.boundActionCreators} />;

    // 另一替代 bindActionCreators 的做法是
    // 直接把 dispatch 函数当作 prop 传递给子组件，但这时你的子组件需要
    // 引入 action creator 并且感知它们

    // return <TodoList todos={todos} dispatch={dispatch} />;
  }
}

export default connect(
  state => ({ todos: state.todos })
)(TodoListContainer);
```

# compose(...function)

从右往左来组合多个函数

当需要把多个store增强其依次执行的时候，需要用到它

**参数**

	- （arguments）：需要合成的多个函数。预计每个函数都接收一个参数。它的返回值将作为一个参数提供给它左边的函数，以此类推。例外是最右边的参数可以接受多个参数，因为它将为由此产生的函数提供签名。（译者注：`compose(funcA, funcB, funcC)` 形象为 `compose(funcA(funcB(funcC())))`） 

**返回值**

（funciton）： 从右往左把接收到的函数合成后的最终函数

```react
// 使用compose增强store，这个store于applyMiddleware和redux-devtools-起使用
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import DevTools from './containers/DevTools'
import reducer from '../reducers'

const store = createStore(
  reducer,
  compose(
    applyMiddleware(thunk),
    DevTools.instrument()
  )
)
// compose做的只是让你在写深度嵌套的函数时，避免了代码的向右偏移
```





​	