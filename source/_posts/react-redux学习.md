---
title: react-redux学习
date: 2018-12-23 14:41:48
tags: react
categories: 前端
---
# 前言
生活还是对我下手了，react 全家桶一个都不能少，这次是 redux。

# redux介绍
> Redux 是 JavaScript 状态容器，提供可预测化的状态管理。可以让你构建一致化的应用，运行于不同的环境（客户端、服务器、原生应用），并且易于测试。

redux 有三大原则：__单一数据源__，__State 是只读的__，__使用纯函数来执行修改__

# 几个概念
## action
**唯一改变 state 的方法就是触发 [action](https://www.redux.org.cn/docs/Glossary.html#action)，action 是一个用于描述已发生事件的‘普通对象’。**

这样确保了视图和网络请求都不能直接修改 `state`，相反它们只能表达想要修改的意图。因为所有的修改都被集中化处理，且严格按照一个接一个的顺序执行，因此不用担心 `race condition` 的出现。 `action` 就是**普通对象**而已，因此它们可以被日志打印、序列化、储存、后期调试或测试时回放出来。

```
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```
这就是一个 `action`。我们约定，`action` 内必须使用一个字符串类型的 `type` 字段来表示将要执行的动作。除了 `type` 字段外，`action` 对象的结构完全由你自己决定。

## action creator
因为一个应用有很多 `action`，不可能自己写死，所以我们用一个函数来生成 `action`，其实这个函数只是返回一个对象，这个对象就是 `action`：
```
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```
## reducer
**Reducers** 指定了应用状态的变化如何响应 [actions](https://www.redux.org.cn/docs/basics/Actions.html) 并发送到 `store` 的，记住 `actions` 只是描述了*有事情发生了*这一事实，并没有描述应用如何更新 `state`。
`reducer` 就是一个纯函数，接收旧的 `state` 和 `action`，返回新的 `state`。
```
(previousState, action) => newState
```
保持 `reducer` 纯净非常重要。永远不要在 `reducer` 里做这些操作：

+ 修改传入参数；
+ 执行有副作用的操作，如 API 请求和路由跳转；
+ 调用非纯函数，如 Date.now() 或 Math.random()。

一个完整的 `reducer`：
```
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default:
      return state
  }
}
```
根据传进来的 `action` 里面的 `type` 来执行不同的操作，然后返回一个新的 `state`，**注意：不要修改 state。在 default 情况下返回旧的 state。遇到未知的 action 时，一定要返回旧的 state。**

当然，当应用越来越大，需要对 `reducer` 进行拆分。Redux提出通过定义多个`reducer`对数据进行拆解访问或者修改，最终再通过`combineReducers`函数将零散的数据拼装回去：
```
// Function names don't matter!
function processA(state, action) { ... }
function doSomethingWithB(state, action) { ... }


let reducer = combineReducers({
  a: processA,
  b: doSomethingWithB
});

/* 这种方法等价于 combineReducers
function reducer(state = {}, action) {
  return {
    a: processA(state.a, action),
    b: doSomethingWithB(state.b, action)
  };
}
*/

let store = createStore(reducer);
```
每一个`子reducer`都将直接对应数据源（`store`）的某一个字段，这样将所有的 `reducer` 组合之后，就形成一棵状态树：
```
import { combineReducers } from 'redux';

// 叶子reducer
function aReducer(state = 1, action) {/*...*/}
function cReducer(state = true, action) {/*...*/}
function eReducer(state = [2, 3], action) {/*...*/}

const dReducer = combineReducers({
  e: eReducer
});

const bReducer = combineReducers({
  c: cReducer,
  d: dReducer
});

// 根reducer
const rootReducer = combineReducers({
  a: aReducer,
  b: bReducer
});
```
![](https://upload-images.jianshu.io/upload_images/5834506-cf5397cf7ca3d35e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## store
redux 的 `store` 将整个应用的 [state](https://www.redux.org.cn/docs/Glossary.html#state) 储存在一棵 `object tree`中，并且这个 `object tree` 只存在于唯一一个 [store](https://www.redux.org.cn/docs/Glossary.html#store) 中。简单来说，`store`就是把 `action` 和 `reducer` 连接起来。
```
import { createStore } from 'redux'
import todoApp from './reducers' // 合成的 reducer
// 用 createStore 生成 store
let store = createStore(todoApp)
```
`store` 有几个方法

+ [`getState()`](https://www.redux.org.cn/docs/api/Store.html#getState) 方法获取 state；
+ [`dispatch(action)`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 方法更新 state；
+ [`subscribe(listener)`](https://www.redux.org.cn/docs/api/Store.html#subscribe) 注册监听器;

# react 和 redux 结合
上面那些全都是 redux 的内容，完全可以不用 react 就能跑起来。用 react 写的展示组件和用 redux 写的容器组件 通过 `connect()` 函数结合起来。
![](https://upload-images.jianshu.io/upload_images/5834506-c40333f7a4ce43fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用 `connect()` 前，需要先定义 `mapStateToProps` 这个函数来指定如何把当前 `Redux store state` 映射到展示组件的 `props` 中。其实就是从完整的 `store`树里面抽出这个展示组件需要的一些 `state`，这样这个组件就不会感知到其他 `state` 的存在。
```
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    case 'SHOW_ALL':
    default:
      return todos
  }
}

const mapStateToProps = state => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```
`mapStateToProps` 始终返回一个对象。
除了读取 `state`，容器组件还能分发 `action`。类似的方式，可以定义 `mapDispatchToProps()` 方法接收 [`dispatch()`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 方法并返回期望注入到展示组件的 `props` 中的回调方法。
```
const mapDispatchToProps = dispatch => {
  return {
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}
```
`mapDispatchToProps` 始终返回一个对象。
最后，使用 `connect()` 创建 `VisibleTodoList`，并传入这两个函数。
```
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```
当用 `connect()` 连接后，我们在 `APP` 组件都是用这个连接之后的组件了。
```
import VisibleTodoList from '../containers/VisibleTodoList'

const App = () => (
  <div>
    <VisibleTodoList />
  </div>
)

export default App
```
最后一步，传入 `store` 使用指定的 React Redux 组件 [`<Provider>`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) 来 [魔法般的](https://facebook.github.io/react/docs/context.html) 让所有容器组件都可以访问 `store`，而不必显示地传递它。只需要在渲染根组件时使用即可。
```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```
最后用一张图来总结：
![](https://upload-images.jianshu.io/upload_images/5834506-8eb2f1894bc2b409.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 异步
之前的都是我们的 `action` 都是同步操作，但是我们的网络请求或者其他需要异步的操作，单纯的 redux 并不能提供异步的操作，所以又需要其他库（惊不惊喜），这里介绍一下算是用的比较多的库：redux-thunk 和 redux-saga，这些库都是用到 redux midlleware 的特性，简单来说就是它提供的是位于 `action` 被发起之后，到达 `reducer` 之前的扩展点。你可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。
## redux-thunk
这个库是官网里面作为教程的，所以也有一定的意义。通过使用指定的 `middleware`，`action 创建函数`除了返回 `action` 对象外还可以返回函数。这时，这个 `action` 创建函数就成为了 [thunk](https://en.wikipedia.org/wiki/Thunk)。当 `action 创建函数`返回函数时，这个函数会被 Redux Thunk middleware 执行。这个函数并不需要保持纯净；它还可以带有副作用，包括执行异步 API 请求。这个函数还可以 `dispatch action`，就像 `dispatch` 前面定义的同步 `action` 一样。
```
// 来看一下我们写的第一个 thunk action 创建函数！
// 虽然内部操作不同，你可以像其它 action 创建函数 一样使用它：
// store.dispatch(fetchPosts('reactjs'))
export function fetchPosts(subreddit) {

  // Thunk middleware 知道如何处理函数。
  // 这里把 dispatch 方法通过参数的形式传给函数，
  // 以此来让它自己也能 dispatch action。

  return function (dispatch) {

    // 首次 dispatch：更新应用的 state 来通知
    // API 请求发起了。
    // 这时 dispatch 的是一个同步的 action
    dispatch(requestPosts(subreddit))

    // thunk middleware 调用的函数可以有返回值，
    // 它会被当作 dispatch 方法的返回值传递。

    // 这个案例中，我们返回一个等待处理的 promise。
    // 这并不是 redux middleware 所必须的，但这对于我们而言很方便。

    return fetch(`http://www.subreddit.com/r/${subreddit}.json`)
      .then(
        response => response.json(),
        // 不要使用 catch，因为会捕获
        // 在 dispatch 和渲染中出现的任何错误，
        // 导致 'Unexpected batch number' 错误。
        // https://github.com/facebook/react/issues/6895
         error => console.log('An error occurred.', error)
      )
      .then(json =>
        // 可以多次 dispatch！
        // 这里，使用 API 请求结果来更新应用的 state。
        // 这个也是一个同步的 action
        dispatch(receivePosts(subreddit, json))
      )
  }
}
```
然后，我们在在 dispatch 机制中引入 Redux Thunk middleware ，我们使用了 [`applyMiddleware()`](https://www.redux.org.cn/docs/api/applyMiddleware.html)
```
import thunkMiddleware from 'redux-thunk'
import { createLogger } from 'redux-logger'
import { createStore, applyMiddleware } from 'redux'
import { selectSubreddit, fetchPosts } from './actions'
import rootReducer from './reducers'
// 另外一个 middleware
const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // 允许我们 dispatch() 函数
    loggerMiddleware // 一个很便捷的 middleware，用来打印 action 日志
  )
)

store.dispatch(selectSubreddit('reactjs'))
store
  .dispatch(fetchPosts('reactjs'))
  .then(() => console.log(store.getState())
)
```
这个异步 action 创建函数还可以有另外一个参数 `getState`：
```
export function fetchPostsIfNeeded(subreddit) {

  // 注意这个函数也接收了 getState() 方法
  // 它让你选择接下来 dispatch 什么。

  // 当缓存的值是可用时，
  // 减少网络请求很有用。

  return (dispatch, getState) => {
    if (shouldFetchPosts(getState(), subreddit)) {
      // 在 thunk 里 dispatch 另一个 thunk！
      return dispatch(fetchPosts(subreddit))
    } else {
      // 告诉调用代码不需要再等待。
      return Promise.resolve()
    }
  }
}
```
## 最后
关于 redux-saga ，由于现在接触的项目没有怎么用，也不好说，所以等实际使用之后再来记录吧。