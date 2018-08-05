---
title: Vue-loader入门
date: 2018-03-11 17:39:56
tags: vue
categories: 前端
---
## Vue-loader 介绍
Vue-loader是一个集成了 webpack 的loader，能把 Vue 组件转化成JavaScript模块，而且 webpack 的配置官方已经给你配好了，可以省去配置的步骤，省时省力。
## Vue-loader 使用
### 安装
Vue-loader 可以通过官方的 vue-cli 脚手架来使用
首先，安装 vue
`npm install vue`

全局安装webpack
`npm install --global webpack`

全局安装 vue-cli
`npm install --global vue-cli`

新建一个 vue-cli 项目，把项目名替换成自己的项目名
`vue init webpack {项目名}`
回车确认后，会让你进行一系列的配置，可以根据自己的需求进行配置，默认回车
![](https://upload-images.jianshu.io/upload_images/5834506-e4cc57e04d6ba521.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
建议安装 vue-router
>__什么是vue-router？__
vue-router是Vue.js官方的路由插件，它和vue.js是深度集成的，适合用于构建单页面应用。vue的单页面应用是基于路由和组件的，路由用于设定访问路径，并将路径和组件映射起来。传统的页面应用，是用一些超链接来实现页面切换和跳转的。在vue-router单页面应用中，则是路径之间的切换，也就是组件的切换。

### 目录结构
![](https://upload-images.jianshu.io/upload_images/5834506-9205fd81de5bc06c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

|--build  //webpack相关代码文件夹
|  |--build.js //生产环境结构代码
|  |--check-version.js //检查node、npm、等版本
|  |--dev-client.js  //热加载相关代码
|  |--dev-server.js  //本地服务器
|  |--utils.js  //构建工具
|  |--webpack.base.conf.js //webpack基本配置
|  |--webpack.dev.conf.js //webpack开发环境配置
|  |--webpack.prod.conf.js  //webpack生产环境配置
|--config  //项目开发环境配置
|  |--dev.env.js  //开发环境变量
|  |--index.js  //项目基本配置（proxyTable:{ //配置请求代理}） 
|  |--dev.env.js  //开发环境变量
|  |--prod.env.js  //生产环境变量
|--dist  //执行npm run build，生成打包发布的目录 
|--node_modules  //初始化 npm install，生成的依赖包目录（注意，不要提交到svn！）
|--src  //项目源代码目录
|  |--components  //组件目录
|  |--assets  //Vue默认logo目录
|  |--router  //路由目录
|  |--APP.vue  //默认组件，入口文件
|  |--main.js  //程序入口文件，引用、加载各种组件
|--static  //静态文件目录，比如：CSS、图片、等等静态文件
|--index.html  //入口文件
### 启动
使用`npm install `之后，就在项目目录下使用命令`npm run dev`就能启动
然后打开浏览器，输入`localhost:8080`，如果出现以下图片，就算启动成功了
![](https://upload-images.jianshu.io/upload_images/5834506-aed6d68020aae8ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意：有一些`8080`端口被占用了，这个时候 vue-loader会去找其他空闲的端口，具体看命令行：
![](https://upload-images.jianshu.io/upload_images/5834506-36db4a35d86e9a12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
它会说当前应用在哪个端口运行

### Vue-loader 与 Vue 的不同
#### 区别一、写法不同
刚开始接触 Vue-loader ，发现完全上不了手，之前学的 Vue 语法都没怎么看到
Vue-loader 语法
```
//App.vue
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```
Vue 语法
```
<div id="vue_example">
    <h1>message : {{site}}</h1>
    <h1>{{details()}}</h1>
</div>
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue_example',
        data: {
            message: 'hello world'
        },
        methods: {
            details: function() {
                return  this.message + "first vue application";
            }
        }
    })
</script>
```

相比较之下，我们可以看到，Vue-loader 的写法是
```
<template> 
  <!-- 组件模板写在这里 --> 
</template> 
<script>
   // 组件逻辑相关代码写这里 
</script> 
<style> 
    /* 组件样式写在这里 */ 
</style>
```

`<template>`

*   默认语言：`html`。
*   每个 `.vue` 文件最多包含一个 `<template>` 块。
*   内容将被提取为字符串，将编译并用作 Vue 组件的 `template` 选项。

`<script>`
*   默认语言：`js` (在检测到 `babel-loader` 或 `buble-loader` 配置时自动支持ES2015)。
*   每个 `.vue` 文件最多包含一个 `<script>` 块。
*   该脚本在类 CommonJS 环境中执行 (就像通过 webpack 打包的正常 js 模块)，这意味着你可以 `require()` 其它依赖。在 ES2015 支持下，你也可以使用 `import` 和 `export` 语法。
*   脚本必须导出 Vue.js 组件对象。也可以导出由 `Vue.extend()` 创建的扩展对象，但是普通对象是更好的选择。

`<style>`
*   默认语言：`css`。
*   一个 `.vue` 文件可以包含多个 `<style>` 标签。
*   `<style>` 标签可以有 `scoped` 或者 `module` 属性 (查看 [CSS 作用域](https://vue-loader.vuejs.org/zh-cn/features/scoped-css.html)和 [CSS Modules](https://vue-loader.vuejs.org/zh-cn/features/css-modules.html)) 以帮助你将样式封装到当前组件。具有不同封装模式的多个 `<style>` 标签可以在同一个组件中混合使用。
*   默认情况下，将会使用 `style-loader` 提取内容，并通过 `<style>` 标签动态加入文档的 `<head>` 中，也可以[配置 webpack 将所有 styles 提取到单个 CSS 文件中](https://vue-loader.vuejs.org/zh-cn/configurations/extract-css.html)。


而 Vue 的写法是
```
<div>
<!--组件模板写在这里-->
</div>
<script>
   // 组件逻辑相关代码写这里 
</script> 
```
#### 区别二、组件定义
Vue-loader 和 Vue 的组件定义有点不一样
```
//HelloWorld.vue
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  }
}
```
Vue-loader 推荐，将一个文件作为一个组件，并用 `export deafult`的方式，导出组件，在其他地方使用 `import` 引入组件
而在很多 Vue 项目中，我们使用 `Vue.component` 来定义全局组件，紧接着用 `new Vue({ el: '#container '})` 在每个页面内指定一个容器元素。
这种方式在很多中小规模的项目中运作的很好，在这些项目里 `JavaScript` 只被用来加强特定的视图。但当在更复杂的项目中，或者你的前端完全由 `JavaScript` 驱动的时候，下面这些缺点将变得非常明显：
全局定义 (`Global definitions`) 强制要求每个 `component` 中的命名不得重复
字符串模板 (`String templates`) 缺乏语法高亮，在 `HTML` 有多行的时候，需要用到丑陋的 \
不支持 CSS (`No CSS support`) 意味着当 `HTML` 和 `JavaScript` 组件化时，CSS 明显被遗漏
没有构建步骤 (`No build step`) 限制只能使用 `HTML` 和 `ES5 JavaScript`, 而不能使用预处理器，如 `Pug` (`formerly Jade`) 和 `Babel`

总的来说，Vue 是写在以 `.js` 的 JavaScript 脚本里面的，而 Vue-loader 则是写在以`.vue` 的 vue 脚本里面的。

#### 区别三、组件选项：data
Vue-loader里面，`data`属性是要返回值的，写法是
```
data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
```
对比 Vue ：
```
data: {
            message: 'hello world'
        },
```
这点小区别也是前期需要注意的。
#### 区别四、el
我们知道，在 Vue组件里面，我们是需要 `el` 来确定一个组件的，上面的例子
```
<div id="vue_example">
    <h1>message : {{site}}</h1>
    <h1>{{details()}}</h1>
</div>
<script type="text/javascript">
    var vm = new Vue({
        el: '#vue_example',
        data: {
            message: 'hello world'
        },
        methods: {
            details: function() {
                return  this.message + "first vue application";
            }
        }
    })
</script>
```
它提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标。一般我们使用 `new` 关键字创建 Vue 组件，都要使用它的 `el` 选项。但是我们在 Vue-loader里面，可以看到，它并没有 `el`选项，因为 Vue-loader 会将 Vue 组件转换成 JavaScript模块。
### 引入组件
先看代码：OtherComponent.vue
```
// OtherComponent.vue
<template>
  <div>
    {{message}}
  </div>
</template>

<script>
  export default {
    name: 'other',
    data(){
      return {
        message: 'I am Other Component'
      }
    }
  }
</script>

<style>
</style>
```
然后我们在 HelloWorld.vue 中引入
```
<script>
import OtherComponent from './OtherComponent'

export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  components:{
    OtherComponent
  }
}
</script>
```
我们在`<script>`标签里面写我们的组件，自然也要在`<script>`标签里面引入组件，然后，我们还要添加一个选项`components`，然后把 `OtherComponent` 写进去。
注意，这里涉及到 ES6 的语法 ：
>export default命令，为模块指定默认输出。
其他模块加载该模块时，import命令可以为该匿名函数指定任意名字。

即` import OtherComponent from './OtherComponent'` 这里的 `OtherComponent`是我们自己取的名字，我们也可以取成 `othercomponent` 等名字，这都无所谓，但是，要确保`components:{ OtherComponent}`中对应的名字是一样的，否则就找不到这个组件。

然后我们要在`<template>`标签里面使用
```
<template>
  ...
  <OtherComponent></OtherComponent>
</template>
```
我们是这么想的，直接把组件加到后面，但是当我们看我们的页面的时候，会发现报错了。
```
Component template should contain exactly one root element. If you are using v-if on multiple elements, use v-else-if to chain them instead.
```
这是因为，`<template>` 只允许一个根节点，在原生的 `HelloWorld.vue`的`<template>`中，已经有`<div>` 这个标签作为根节点了，所以我们要解决这个问题，只需要在外面包上一层 `<div>` 就能解决这个问题了。
```
<template>
  <div>  
      ...
    <OtherComponent></OtherComponent>
  </div>
</template>
```
![](https://upload-images.jianshu.io/upload_images/5834506-30fe78fa51f0edbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

还有一点，
```
  components:{
    OtherComponent
  }
```
这样写其实本质上是这样的：
```
  components:{
    OtherComponent:OtherComponent
  }
```
我们也可以手动更改名字
第一种：
```
  components:{
    'other-component':OtherComponent
  }
```
或者，第二种：
```
  components:{
    othercomponent:OtherComponent
  }
```
这两种语法有什么区别呢？仔细看一下就会发现，第一种用了 `-` 这个符号，所以需要用引号来括住，第二种没有用 `-`，所以不用括住。
相对应的，我们使用组件就要改成我们重新命名的名字
```
<template>
  <div>  
      ...
    <other-component></other-component>
  </div>
</template>
```
我们再来看看 `OtherComponent`
```
<script>
  export default {
    name: 'other',
    data(){
      return {
        message: 'I am Other Component'
      }
    }
  }
</script>
```
注意到，我们使用了 `name` 这个选项，但是我们并没有在引入该组件的时候用到过`name`的值`other`，那这个选项有什么用呢？官方给出的解释是
>允许组件模板递归地调用自身。注意，组件在全局用 `Vue.component()` 注册时，全局 ID 自动作为组件的 name。
指定 `name` 选项的另一个好处是便于调试。有名字的组件有更友好的警告信息。另外，当在有 [vue-devtools](https://github.com/vuejs/vue-devtools)，未命名组件将显示成 `<AnonymousComponent>`，这很没有语义。通过提供 `name` 选项，可以获得更有语义信息的组件树。

![](https://upload-images.jianshu.io/upload_images/5834506-deb134bb102fc8d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在开发者工具里面，就可以看到我们的`other`（变成了首字母大写）
