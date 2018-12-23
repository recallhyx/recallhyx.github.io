---
title: react-router学习
date: 2018-11-23 15:02:26
tags: react
categories: 前端
---
# 前言
最近新入职阿里，我们部门用的是 React 全家桶，之前只自学了 React，所以这次要学一下 React-router

<!-- more -->

# 使用方法
```
import { BrowserRouter as Router, Route, Link } from "react-router-dom";
const Home = () => <h2>home</h2>;
const About = () => <h2>about</h2>;
const Topic = ({match}) => (
  <h3>Requested Param: {match.params.id}</h3>
);
const Topics = ({match}) => (
  <div>
    <Route path={`${match.path}/:id`} component={Topic}/>
    <Route
      exact
      path={match.path}
      render={()=><h3>Please select a topic.</h3>}
    />
    <Route
      render={()=><h1>Always render</h1>}
    />

    <h2>Topics</h2>

    <ul>
      <li>
        <Link to={`${match.url}/components`}>Components</Link>
      </li>
      <li>
        <Link to={`${match.url}/props-v-state`}>Props-v-state</Link>
      </li>
    </ul>
  </div>
);
const App = () => {
    <Router>
      <div>
        <ul>
          <li>
            <Link to="/">Home</Link>
          </li>
          <li>
            <Link to="/about">About</Link>
          </li>
          <li>
            <Link to="/topics">Topics</Link>
          </li>
        </ul>

        <hr />

        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
        <Route path="/topics" component={Topics} />
      </div>
    </Router>
};
```
## Router
这是简单的结构，从第一行开始看，引入了 `BrowserRouter`，这个其实还有另外一个兄弟 `HashRouter `
区别：`BrowserRouter` 用的是浏览器的HTML5 history API (pushState, replaceState, popstate event)，`HashRouter`根据URL后面的 `#` 的内容来进行刷新，这个区别在 Vue-router 里面也有。
联系：都是 React-router 的核心，有 Router 才能继续接下来的工作。

## Link
这个组件就相当于 `a` 标签，当然还有其他类似的组件：`<NavLink> `，`<Redirect>`，根据不同场景使用

## Route
这个组件就是如果匹配到了 `path` 就会渲染 `component`属性对应的组件，__注意：如果没有 `path` 属性，则一定会渲染这个 route__，对于 `exact` 属性，有一张图
![](https://upload-images.jianshu.io/upload_images/5834506-4dd27e8eae2b7db0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以很清楚的理解 `exact` 属性

## match
这个就是在路由匹配到的时候可以知道匹配的路由里面的东西，具体的属性看下表：
| 属性 | 类型 | 说明/作用 |
| ------ | ------ | ------ |
| params | object | 从 URL 传进来的键值对 |
| isExact | boolean | 是否和 URL 完全匹配 |
| path | string | 值是 path，也就是Route的属性，用作嵌套<Route> |
| url | string | 值是匹配的 URL，用作嵌套<Link> | 

# 值的注意的一些东西
## <Switch> 和 <Route>
上面说了 `<Route>` 会根据 `path` 渲染匹配到路由的组件，而且不设置 `path` 组件也会渲染，也就是说 `<Route>` 能够同时渲染多个组件，但是我们有时候只想渲染一个组件，这个时候 `<Switch>` 就出来了，用法就是在 `<Route>` 外面包住：
```
import { Switch, Route } from 'react-router'

<Switch>
  <Route exact path="/" component={Home}/>
  <Route path="/about" component={About}/>
  <Route path="/:user" component={User}/>
  <Route component={NoMatch}/>
</Switch>
```
当匹配到一个 route，就会停止接下来的查找，也就是自始自终只有一个组件会被渲染，这在做路由跳转动画的时候会比较有利。

## 同时渲染侧边栏内容和主页内容
具体场景就是点击侧边栏的内容，在主内容跳转渲染的同时，同时改变侧边栏样式，这里只是简单的版本，具体根据自己需要修改：
```
import React from "react";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

const routes = [
  {
    path: "/",
    exact: true,
    sidebar: () => <div>home!</div>,
    main: () => <h2>Home</h2>
  },
  {
    path: "/bubblegum",
    sidebar: () => <div>bubblegum!</div>,
    main: () => <h2>Bubblegum</h2>
  },
  {
    path: "/shoelaces",
    sidebar: () => <div>shoelaces!</div>,
    main: () => <h2>Shoelaces</h2>
  }
];
function SidebarExample() {
  return (
    <Router>
      <div style={{ display: "flex" }}>
        <div
          style={{
            padding: "10px",
            width: "40%",
            background: "#f0f0f0"
          }}
        >
          <ul style={{ listStyleType: "none", padding: 0 }}>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/bubblegum">Bubblegum</Link>
            </li>
            <li>
              <Link to="/shoelaces">Shoelaces</Link>
            </li>
          </ul>

          {routes.map((route, index) => (
            <Route
              key={index}
              path={route.path}
              exact={route.exact}
              component={route.sidebar}
            />
          ))}
        </div>

        <div style={{ flex: 1, padding: "10px" }}>
          {routes.map((route, index) => (
            // Render more <Route>s with the same paths as
            // above, but different components this time.
            <Route
              key={index}
              path={route.path}
              exact={route.exact}
              component={route.main}
            />
          ))}
        </div>
      </div>
    </Router>
  );
}

export default SidebarExample;
```
其实很简单，首先有一个 `routes` 的配置，在里面设置对应的组件，然后传给 `<Route>`，它就会根据传进来的东西生成一个列表，但是真正渲染的只有匹配到路由的那个组件。

## withRouter
用法：
```
import React from "react";
import PropTypes from "prop-types";
import { withRouter } from "react-router";

class ShowTheLocation extends React.Component {
  static propTypes = {
    match: PropTypes.object.isRequired,
    location: PropTypes.object.isRequired,
    history: PropTypes.object.isRequired
  };

  render() {
    const { match, location, history } = this.props;

    return <div>You are now at {location.pathname}</div>;
  }
}
const ShowTheLocationWithRouter = withRouter(ShowTheLocation);
```
其实就是通过  `withRouter` 高阶组件访问 `history` 对象的属性和最近（UI 结构上靠的最近）的 `<Route>` 的 `match` 对象。当组件渲染时，`withRouter` 会将更新后的 `match`、`location` 和 `history` 传递给它。