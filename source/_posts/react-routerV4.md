---
title: react-routerV4
date: 2018-07-05 13:25:00
tags: react-router
---

# React-router 和 React-router-dom

## V4 版本的不同

 - 在react-router v4里，路由不再是集中在一起。它成了应用布局，UI的一部分
 - 浏览器用的router在`react-router-dom`里。所以，浏览器里使用的时候只需要import `react-router-dom`就可以。
 - 增加了新的`BrowerRouter`和`HashRouter` 。
 - 不在使用`{props.children}`来处理嵌套的路由。
 - v4的路由默认不再排他（exact），会有多个匹配。而v3是默认排他的，只会有一个匹配被使用。

`react-router`被分为：

 - react-router：浏览器和原生应用的通用部分
 - react-router-dom：用于浏览器
 - react-router-native：用于原生应用

## react-router

react-router提供了一些router的核心api，包括router，route，switch等。但是它没有提供dom操作进行跳转的api。

## react-router-dom

React-router-dom提供了BrowserRouter, Route, Link等api,我们可以通过dom的事件控制路由。 

## HashRouter 和 BrowserRouter

它们两个是路由的基本，需要将它们包裹在最外层，只要选择其中一个就可以。

### HashRouter

`HashRouter` 是一种特定的`<Router>`， `HashRouter` 使用 URL 的 hash (例如：`window.location.hash`) 来保持 UI 和 URL 的同步。 

**注意**

使用 hash 的方式记录导航历史不支持 `location.key` 和 `location.state`。 在以前的版本中，我们为这种行为提供了 shim，但是仍有一些问题我们无法解决。 任何依赖此行为的代码或插件都将无法正常使用。 由于该技术仅用于支持传统的浏览器，因此在用于浏览器时可以使用 `<BrowserHistory>` 代替 

如果你使用过react-router2或3或者vue-router，你经常会发现一个现象就是url中会有个#，例如localhost:3000/#，HashRouter就会出现这种情况，它是通过hash值来对路由进行控制。如果你使用HashRouter，你的路由就会默认有这个#。 

```react
ReactDOM.render(
	<HashRouter>
    	<Route path='/' commponent={home}></Route>
    </HashRouter>
)
```

**参数**

- basename: string

  当前位置的基准 URL。正确的 URL 格式是前面有一个前导斜杠，但不能有尾部斜杠。 

  ```react
  <HashRouter basename="/calendar"/> // 默认在路由前加上了/calendar
  <Link to="/today"/> // renders <a href="#/calendar/today">
  ```

- getUserConfirmation：func

  当导航需要确认时执行的函数。默认使用window.confirm

  ```react
  // 使用默认的确认函数
  const getConfirmation = (message, callback) => {
    const allowTransition = window.confirm(message)
    callback(allowTransition)
  }
  
  <HashRouter getUserConfirmation={getConfirmation}/>
  ```

- hashTyoe：string

  `window.location.hash` 使用的 hash 类型。有如下几种：

  - `"slash"` - 后面跟一个斜杠，例如 `#/` 和 `#/sunshine/lollipops`
  - `"noslash"` - 后面没有斜杠，例如 `#` 和 `#sunshine/lollipops`
  - `"hashbang"` - Google 风格的 `ajax crawlable`，例如 `#!/` 和 `#!/sunshine/lollipops`

  默认为 `"slash"`。

- children：node

  渲染单一子组件（元素）

### BrowserRouter

`<Router>`使用 HTML5 提供的 history API(`pushState`, `replaceState` 和 `popstate` 事件) 来保持 UI 和 URL 的同步 。路由中没有#

```react
import { BrowserRouter } from 'react-router-dom'

<BrowserRouter
  basename={optionalString}
  forceRefresh={optionalBool}
  getUserConfirmation={optionalFunc}
  keyLength={optionalNumber}
>
  <App/>
</BrowserRouter>
```

**参数**

- basename：string

  当前位置的基准 URL。如果你的页面部署在服务器的二级（子）目录，你需要将 `basename` 设置到此子目录。 正确的 URL 格式是前面有一个前导斜杠，但不能有尾部斜杠。 

  ```react
  <BrowserRouter basename="/calendar"/>
  <Link to="/today"/> // 渲染为 <a href="/calendar/today">
  ```

- getUserConfirmation：func

  当导航需要确认时执行的函数。默认使用window.confirm

  ```react
  // 使用默认的确认函数
  const getConfirmation = (message, callback) => {
    const allowTransition = window.confirm(message)
    callback(allowTransition)
  }
  
  <BrowserRouter getUserConfirmation={getConfirmation}/>
  ```

- forceRefresh：bool

  当设置为`true`时，在导航的过程中整个页面将会刷新。只有当浏览器不支持`HTML5` 的`history API`时，才设置为`true`.

   ```react
  const supportsHistory = 'pushState' in window.history
  <BrowserRouter forceRefresh={!supportsHistory}/>
   ```

- keyLength：number

  location.key的长度。默认是6

  ```react
  <BrowserRouter keyLength={12} />
  ```

- children：node

  渲染单一子组件（元素）

## route

路由组件是路由器中最重要的组件，它最基本的职责是在一个位置与路径的路径匹配时呈现一些UI。 

```react
import { BrowserRouter as Router, Route } from 'react-router-dom'

<Router>
  <div>
    <Route exact path="/" component={Home}/>
    <Route path="/news" component={NewsFeed}/>
  </div>
</Router>
```

**属性**

- path（string）：它的值就是用来匹配url的.    
- component：它的值是一个组件。在path匹配成功之后会绘制这个组件
- exact：这个属性用来指明这个路由是不是排他的匹配
- strict：这个属性指明路径只匹配以斜线结尾的路径

可以用`render `  `children` 来代替`component`

- render：一个返回react组件的方法
- children：返回一个React组件的方法。只不过这个总是会绘制，即使没有匹配的路径的时候

多数的时候是用`component`属性就可以满足。但是，某些情况下你不得不使用`render`或`children`属性。 

```react
// 一般使用component
<Route exact path="/" component={HomePage} />

// 使用render
<Route path="/" render={()=><div>HomePage</div>} />

// 使用children
<ul>
  <ListItemLink to="/somewhere" />
  <LinkItemLink to="/somewhere-else" />
</ul>

const ListItemLink = ({to, ...rest}) => (
  <Route path={to} children={({math}) => (
    <li className={match ? 'active' : ''}>
      <Link to={to} {...rest} />
    </li>
  )} />
)
```

Router props

- match
- location
- history

component 组件可以根据this.props.match 等来获取数据

```react
<Route path="/user/:username" component={User}/>

const User = ({ match }) => {
  return <h1>Hello {match.params.username}!</h1>
}
```

render : function

```react
// convenient inline rendering
<Route path="/home" render={() => <div>Home</div>}/>

// wrapping/composing
const FadingRoute = ({ component: Component, ...rest }) => (
  <Route {...rest} render={props => (
    <FadeIn>
      <Component {...props}/>
    </FadeIn>
  )}/>
)

<FadingRoute path="/cool" component={Something}/>
```

children: func

```react
<ul>
  <ListItemLink to="/somewhere"/>
  <ListItemLink to="/somewhere-else"/>
</ul>

const ListItemLink = ({ to, ...rest }) => (
  <Route path={to} children={({ match }) => (
    <li className={match ? 'active' : ''}>
      <Link to={to} {...rest}/>
    </li>
  )}/>
)
//-----------
<Route children={({ match, ...rest }) => (
  {/* Animate will always render, so you can use lifecycles
      to animate its child in and out */}
  <Animate>
    {match && <Something {...rest}/>}
  </Animate>
)}/>
```

path: string

```react
<Route path="/users/:id" component={User}/>
```

exact: bool

```react
<Route exact path="/one" component={About}/>
```

strict: bool

```react
<Route strict path="/one/" component={About}/>
/one/	/one	no
/one/	/one/	yes
/one/	/one/two	yes



<Route exact strict path="/one" component={About}/>
/one	/one	yes
/one	/one/	no
/one	/one/two	no

```

## Link

`Link`是react router v4特有的一个组件 。使用`Link`可以在React应用的不同页面之间跳转，Link只会重新加载页面里和当前url可以匹配的部分

```react
import { Link } from 'react-router-dom'

<Link to="/about">关于</Link>
```

**to (string)**

需要跳转到的路径(path)或者(location)

```react
<Link to="/courses"/>
```

**to (object)**

需要跳转到的地址（location） 

```react
<Link to={{
  pathname: '/courses',
  search: '?sort=name',
  hash: '#the-hash',
  state: { fromDashboard: true }
}}/>
```

**replace (bool)**

当设置为`true`时，点击链接后将使用新地址替换掉访问历史记录里面的原地址

当设置为`false`时，点击链接后将在原有访问历史记录的基础上添加一个新的记录

默认为`false`

```react
<Link to="/courses" replace />
```

**三种传参**

1. props.params

   ```react
   <Route path='/user/:name' component={UserPage}></Route>
   // 1.Link组件实现跳转
   <Link to="/user/sam">用户</Link>
   // 2.history跳转
   hashHistory.push("/user/sam");
   
   // UserPage页面取值
   this.props.params.name
   
   //上面的方法可以传递一个或多个值，但是每个值的类型都是字符串，没法传递一个对象,如果传递的话可以将json对象转换为字符串，然后传递过去，传递过去之后再将json字符串转换为对象将数据取出来
   
   <Route path='/user/:data' component={UserPage}></Route>
   
   var data = {id:3,name:sam,age:36};
   data = JSON.stringify(data);
   var path = `/user/${data}`;
   
   <Link to={path}>用户</Link>
   
   hashHistory.push(path);
   
   // 获取数据
   var data = JSON.parse(this.props.params.data);
   var {id,name,age} = data;
   ```

2. query

   ```react
   //query方式使用很简单，类似于表单中的get方法，传递参数为明文
   <Route path='/user' component={UserPage}></Route>
   
   var data = {id:3,name:sam,age:36};
   var path = {
     pathname:'/user',
     query:data,
   }
   
   <Link to={path}>用户</Link>
   
   hashHistory.push(path);
   
   // 获取数据
   var data = this.props.location.query;
   var {id,name,age} = data;
   //query方式可以传递任意类型的值，但是页面的URL也是由query的值拼接的，URL很长
   ```

3. state

   ```react
   //state方式类似于post方式，使用方式和query类似
   <Route path='/user' component={UserPage}></Route>
   
   var data = {id:3,name:sam,age:36};
   var path = {
     pathname:'/user',
     state:data,
   }
   
   <Link to={path}>用户</Link>
   
   hashHistory.push(path);
   
   //获取数据
   var data = this.props.location.state;
   var {id,name,age} = data;
   //state方式依然可以传递任意类型的数据，而且可以不以明文方式传输。
   ```

   

## NavLink

`NavLink`是`Link`的一个子类，在Link组件的基础上增加了绘制组件的样式 

```react
<NavLink to="/me" activeStyle={{SomeStyle}} activeClassName="selected">
  My Profile
</NavLink>
// selected 是选中状态的类名，我们可以为其添加样式

<NavLink
  to="/faq"
  activeStyle={{
    fontWeight: 'bold',
    color: 'red'
   }}
>FAQs</NavLink>
```

**activeStyle(object)**

NavLink的样式

```react
<NavLink
  to="/faq"
  activeStyle={{
    fontWeight: 'bold',
    color: 'red'
   }}
>FAQs</NavLink>
```

**exact(bool)**

```react
<NavLink
  exact
  to="/profile"
>Profile</NavLink>
```

**strict(bool)**

当正确的时候，当确定位置是否与当前URL匹配时，将考虑位置路径名上的斜杠 

```react
<NavLink
  strict
  to="/events/"
>Events</NavLink>
```

**isActive(func)**

一个函数来添加额外的逻辑来确定链接是否处于活动状态。如果您想要做更多的事情，而不是验证链接的路径名是否与当前URL的路径名匹配，那么应该使用这个方法 

```react
// only consider an event active if its event id is an odd number
const oddEvent = (match, location) => {
  if (!match) {
    return false
  }
  const eventID = parseInt(match.params.eventID)
  return !isNaN(eventID) && eventID % 2 === 1
}

<NavLink
  to="/events/123"
  isActive={oddEvent}
>Event 123</NavLink>
```

## Redirect

渲染一个“直接”将导航到一个新的位置。新的位置将覆盖历史堆栈中的当前位置

路由重定向

```react
import { Route, Redirect } from 'react-router'

<Route exact path="/" render={() => (
  loggedIn ? (
    <Redirect to="/dashboard"/>
  ) : (
    <PublicHomePage/>
  )
)}/>
```

**to(string)**

重定向到的URL

```react
<Redirect to="/somewhere/else"/>
```

**to(object)**

一个重定向到的location

```react
<Redirect to={{
  pathname: '/login',
  search: '?utm=your+face',
  state: { referrer: currentLocation }
}}/>
```

 **push(bool)**

如果是真的，重定向将把一个新条目推到历史上，而不是替换当前的条目

```react
<Redirect push to="/somewhere/else"/>
```

**from(string)**

A pathname to redirect from. This can be used to match a location when rendering a `<Redirect>` inside of a `<Switch>`. 

```react
<Switch>
  <Redirect from='/old-path' to='/new-path'/>
  <Route path='/new-path' component={Place}/>
</Switch>
```

## Switch

Switch常常会用来包裹Route，它里面不能放其他元素，用来只显示一个路由。 

只匹配第一个`<Route>` 或者 `<Redirect>`

**children(node)**

所有`<Switter >`的孩子应该是`<Route>`或`<Redirect>`元素。仅呈现匹配当前位置的第一个子项。

```react
<Switch>
  <Route exact path="/" component={Home}/>

  <Route path="/users" component={Users}/>
  <Redirect from="/accounts" to="/users"/>

  <Route component={NoMatch}/>
</Switch>
```

## match

match是在使用router之后被放入props中的一个属性，在class创建的组件中我们需要通过this.props.match来获取match之中的信息 

## prompt

用于在位置跳转之前给予用户一些确认信息。当你的应用程序进入一个应该阻止用户导航的状态时（比如表单只填写了一半），弹出一个提示。 

```javascript
import { Prompt } from 'react-router-dom';

<Prompt
  when={formIsHalfFilledOut}
  message="你确定要离开当前页面吗？"
/>
```

**message: string**

当用户尝试离开某个位置时弹出的提示

```javascript
<Prompt message="你确定要离开当前页面吗？" />
```

**message: func**

将在用户试图导航到下一个位置时调用。需要返回一个字符串以向用户显示提示，或者返回 `true` 以允许直接跳转 

```react
<Prompt message={location => {
  const isApp = location.pathname.startsWith('/app');

  return isApp ? `你确定要跳转到${location.pathname}吗？` : true;
}} />
```

> 译注：上例中的 `location` 对象指的是下一个位置（即用户想要跳转到的位置）。你可以基于它包含的一些信息，判断是否阻止导航，或者允许直接跳转。 

**when: bool**

在应用程序中，你可以始终渲染 `<Prompt>` 组件，并通过设置 `when={true}` 或 `when={false}` 以阻止或允许相应的导航，而不是根据某些条件来决定是否渲染 `<Prompt>` 组件。 

> 译注：`when` 只有两种情况，当它的值为 `true` 时，会弹出提示信息。如果为 `false` 则不会弹出。见[阻止导航](https://reacttraining.com/react-router/web/example/preventing-transitions)示例。 

```react
<Prompt when={true} message="你确定要离开当前页面吗？" />
```

