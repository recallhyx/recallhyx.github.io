---
title: Vuex的一些总结
date: 2018-03-29 16:41:57
tags: vue
categories: 前端
---
# 前言
最近在学 Vue 全家桶，遇到 Vuex 这个东西，记录一下学习过程及遇到的不懂的地方和一些坑。

# Vuex 是个啥
>Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 [devtools extension](https://github.com/vuejs/vue-devtools)，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

举个例子，父组件和子组件嵌套，父组件能通过 `props` 传递参数给子组件，这个在只嵌套一层子组件的情况下是推荐使用的，但是如果子组件又嵌套子组件，这样我们最里层的孙子组件，要获取状态只能子组件通过 `props` 传递，这样要写多几次 `props`，而且到后期嵌套的组件越多都不知道哪个 `props` 是从哪里传进来的。

Vuex就是把组件的共享状态抽取出来，以一个全局单例模式管理。在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为。

对于中大型应用，才有使用 Vuex 的必要，如果是小型的应用，完全不需要使其复杂化。
# Vuex 安装 
我是在 vue-cli 基础上进行学习的，不知道 vue-cli 的可以看一下我之前的文章 [Vue-loader 入门](https://recallhyx.github.io/2018/03/11/Vue-loader%E5%85%A5%E9%97%A8/)
首先我们安装 Vuex
```
npm install vuex --save
```
如果成功，我们可以在 `package.json` 文件中看到：
![](https://upload-images.jianshu.io/upload_images/5834506-1e95c37b37ccbb7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Vuex 使用
首先，我们要在 `src` 目录下新建一个文件夹 `store` ，然后在里面新建一个叫 `index.js` 的文件，这是我们组装模块并导出 `store` 的地方。
然后我们再分别新建 `mutations.js ` 和 `actions.js` 这两个文件，作为根级别的 `mutation` 和 根级别的 `action` ，就是抽出来公用的部分。
最后我们新建一个文件夹 `modules`，里面放着我们的模块，然后我们在 `modules` 里面新建我们的模块 `hello.js` 。
当然，我们还可以新建一个 `mutation-types.js` 的文件，直接在`mutations.js ` 中映射相关的方法过来使用，这个文件是为了多人合作的时候方便管理。
全部完成后的 `store` 目录如下：
![](https://upload-images.jianshu.io/upload_images/5834506-4b1cb38856bcaec6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 挂载
接下来我们就要将 Vuex 挂载到 Vue 上，在 `index.js`文件中：
```
// index.js
import Vue from 'vue';
import Vuex from 'vuex';
import * as actions from './actions';
import * as mutations from './mutations';
import hello from './modules/hello'

Vue.use(Vuex);

const store = new Vuex.Store({
  actions,
  mutations,
  modules:{
    hello
  }
});
export default store;
```
然后我们在 `main.js` 里面，将我们的 `store` 引入：
```
// main.js
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store/index'
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,// 将store引入
  components: { App },
  template: '<App/>'
})

```
## hello 模块
接下来我们跳过 `mutations.js` 和 `actions.js`，因为我们这个只是示例，没有需要抽出来的公共的部分，将注意放在我们的 module 模块。当然，模块里面也可以再分 `state.js`，`getters.js`，`mutations.js` 和 `actions.js` ，然后在模块中引入，但是这就分的太细了。我们先定义 `state`，`getters`，`mutations` 和 `actions`：
```
// hello.js
const state = {};
const getters = {};
const mutations = {};
const actions = {};

export default {
  state,
  getters,
  mutations,
  actions
}
```
我们的原始 `state` 就一个 `msg` 属性：
```
const state = {
  msg: 'hello world',
};
```
然后我们在 `getters` 里面定义一个方法，返回这个 `msg`：
```
const getters = {
  getMsg: state => state.msg,
};
```
为了能够清楚的知道我们定义了什么 `mutations` 方法，我们可以在 `mutation-types.js`中定义方法名：
```
// mutation-types.js
export const SET_MSG = 'SET_MSG';
```
然后在 `hello.js` 导入这个文件：
```
// hello.js
import * as type from '../mutation-types'
...
const mutations = {
  [type.SET_MSG](state,msg){
    state.msg = msg;
  }
};
```
这种做法是基于 ES6的 [对象可计算属性名](http://es6.ruanyifeng.com/#docs/object#属性名表达式)
最后我们用 `axios` 来进行异步操作，在 `actions` 里面定义一个方法：
```
const actions = {
  changeMsg({commit}){
    axios.get('http://127.0.0.1:8888')
      .then(res=>{
        commit([type.SET_MSG],res.data);
      })
      .catch(err=>{
        throw new Error(err);
      })
  }
};
```
这里我自己用 node 写了个小小的后端用来获取数据：
```
// example.js
const http = require('http');

const hostname = '127.0.0.1';
const port = '6666';

const server = http.createServer((req,res)=>{
    res.statusCode = 200;
    res.setHeader('Content-Type','text/plain');
    res.setHeader('Access-Control-Allow-Origin','*')// 解决跨域
    res.write('你好')
    res.end();
}).listen(8888);
```
在命令行用 `node example.js` 就可以开启后端并监听 `8888` 端口。
至此，我们的 `hello.js` 模块就完成了，让我们看一下完整的代码：
```
import * as type from '../mutation-types'
import axios from 'axios';

const state = {
  msg: 'hello world',
};

const getters = {
  getMsg: state => state.msg,
};

const mutations = {
  [type.SET_MSG](state,msg){
    state.msg = msg;
  }
};

const actions = {
  changeMsg({commit}){
    axios.get('http://127.0.0.1:8888')
      .then(res=>{
        commit(type.SET_MSG,res.data);
      })
      .catch(err=>{
        throw new Error(err);
      })
  }
};

export default {
  state,
  getters,
  mutations,
  actions
}
```
## 组件
我们就在 vue-cli 生成的 `HelloWorld.vue`这个文件中，应用我们的 `hello.js` 模块
![](https://upload-images.jianshu.io/upload_images/5834506-e3a2182e9cfabbe5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
// Hello
<template>
  <div>
    <h1>我是HelloWorld组件</h1>
    <button @click="changeMsg">改变Msg</button>
    <p>目前Msg是:{{getMsg}}</p>
  </div>
</template>

<script>
import {mapGetters} from 'vuex';
import {mapActions} from 'vuex';
export default {
  name: 'HelloWorld',
  computed:{
    ...mapGetters(['getMsg'])
  },
  methods:{
    ...mapActions(['changeMsg'])
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped >

</style>
```
这里用到了两个辅助函数 `mapGetters` 和 `mapActions`，辅助函数将组件中的 getters 或 actions 映射为 store.commit 调用（需要在根节点注入 store）即直接使用函数如 `<p>目前Msg是:{{getMsg}}</p>` 里面的 `getMsg`，否则我们就要多写几个字：`<p>目前Msg是:{{$store.getters.getMsg}}</p>`。当然也有另外两个辅助行数 `mapState` 和 `mapMutations`，用法也都是一样的。而 `...mapGetters` 前面的 `...` 则是对象展开运算符，如果我们有多个 `getters` ，我们就可以这样写：
```
computed:{
    ...mapGetters([
      'getMsg',
      'getAnotherMsg',// 其他的getter
    ])
  },
```
## 最终效果
![未点击按钮](https://upload-images.jianshu.io/upload_images/5834506-b9b2c67c3dfc8f8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![点击按钮后](https://upload-images.jianshu.io/upload_images/5834506-eb842e21aabaf21a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
加载完成后从 `getMsg` 这个 `getter` 方法获取`state`的状态 `msg` 并显示在屏幕上，点击按钮后，触发函数 `changeMsg` 这个 `action` 方法请求本地数据并且提交 `mutation` 最终 `SET_MSG` 这个 `mutation` 方法将传入的数据赋值给 `msg`。
