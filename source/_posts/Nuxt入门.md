---
title: Nuxt入门
date: 2018-05-16 10:04:48
tags: nuxt
categories: 前端
---
# 什么是Nuxt
简单来说，Nuxt 就是基于 Vue 的服务端渲染框架（SSR），使用 Nuxt 能够让我们更快速的完成开发。
>服务端渲染的好处都有啥？
我们用框架（Angular2，React，Vue）开发 SPA 应用，都会面临一个问题就是首屏加载速度慢的问题。对于 SSR 来说，服务器返回的是（结构相对完整的）HTML文件，（通过解析HTML文件），浏览器就能渲染出页面。而对 CSR 来说，浏览器拿到的只是包含 JavaScript 代码的 HTML 文件，（换句话，在浏览器渲开始渲染出页面之前，需要动态创建 HTML 标签）。这也就意味着，SSR 可以让浏览器在边下载 JavaScript 文件的同时边渲染 HTML 页面，换句话说，浏览器再也不需要等到所有的 JavaScript 文件下载并执行完之后才去渲染页面。
# 安装使用
需要提前安装 vue-cli：
`npm install -g vue-cli`
然后利用 vue-cli 安装使用：
`vue init nuxt-community/starter-template <project-name>`
![](https://upload-images.jianshu.io/upload_images/5834506-6aaaef3687f949c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后在项目目录下安装依赖：
`npm install`
等到安装完毕，`npm run dev` 就能启动项目，在浏览器输入 `http://localhost:3000` 就能看到项目
![](https://upload-images.jianshu.io/upload_images/5834506-8b5f5a8969f11b2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 目录结构
![](https://upload-images.jianshu.io/upload_images/5834506-39d1c41fb0249642.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意两个东西： `pages` 文件夹和 `components` 文件夹，这两个文件夹下都是 `*.vue`文件 ，但是Nuxt.js 会依据 `pages` 目录中的所有 `*.vue` 文件生成应用的路由配置，而 `components` 只是用于组织应用的 Vue.js 组件，Nuxt.js 不会扩展增强该目录下 Vue.js 组件，即这些组件不会像页面组件那样有 `asyncData` 方法的特性。
>asyncData方法会在组件（限于页面组件）每次加载之前被调用。它可以在服务端或路由更新之前被调用。你可以利用 asyncData 方法来获取数据，Nuxt.js 会将 asyncData 返回的数据融合组件 data 方法返回的数据一并返回给当前组件。

还有 `assets` 文件夹和 `static` 文件夹，对于不需要通过 Webpack 处理的静态资源文件，可以放置在 `static` 目录中。而资源目录 `assets` 用于组织未编译的静态资源如 `LESS` 、`SASS` 或 `JavaScript` 。
# 小例子
## 资源引入问题
我要使用 `vuetify` 这个框架，看到需要在 `index.js` 或 `main.js` 引入，但是 nuxt 并没有这两个文件，怎么办？重点就是 `plugins` 文件夹，我们在该文件夹目录下新建一个文件夹 `vuetify.js`，当然，我们需要先下载 `vuetify`：`npm install vuetify --save`：
```
import Vue from 'vue'
import Vuetify from 'vuetify'
import 'vuetify/dist/vuetify.min.css'
Vue.use(Vuetify)
```
然后, 在 `nuxt.config.js` 内配置 `plugins` 如下：
```
module.exports = {
  build: {
    vendor: ['~/plugins/vuetify']
  },
  plugins: [
    {src: '~/plugins/vuetify', ssr: false}
  ]
}
```
其实只需要配置 `plugins` 就可以跑起来了，实际上， `vuetify` 会被打包至应用的脚本代码里， 但是它属于第三方库，我们理应将它打包至库文件里以获得更好的缓存效果，所以需要在 `build` 里面配置 `vendor`。（应用代码比库文件修改频繁，应尽量将第三方库打包至单独的文件中去）。
至于后面的 `ssr:false` ，因为我们的插件只在浏览器端使用，所以用 `ssr: false` 变量来配置插件只从客户端还是服务端运行。
但是还没完，我们还需要引入谷歌的字体，如果我们直接使用 CDN 的话，只需要在 `nuxt.config.js` 里面的 `head` 中的 `link` 添加谷歌字体 CDN ：
```
link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' },
      { rel: 'stylesheet', href: 'https://fonts.googleapis.com/css?family=Roboto:300,400,500,700|Material+Icons' }
]
```
但是你懂得，谷歌在墙外，所以要下载谷歌字体下来很慢很慢，所以我们就只能抛弃 CDN ，直接文件引入。我们先打开谷歌 CDN 这个链接看看有啥：
![](https://upload-images.jianshu.io/upload_images/5834506-0935a1b31e4d38ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一大堆东西，都是不同的字体，我们只需要选择一种字体下载并导入即可
我们在 `assets` 目录新建一个目录 `style`并创建 `app.css` 文件：
```
/* app.css*/
@font-face {
  font-family: 'Material Icons';
  font-style: normal;
  font-weight: 400;
  src: local('Material Icons'), local('MaterialIcons-Regular'), url(../font/materialfont.woff2) format('woff2');
}

.material-icons {
  font-family: 'Material Icons';
  font-weight: normal;
  font-style: normal;
  font-size: 24px;
  line-height: 1;
  letter-spacing: normal;
  text-transform: none;
  display: inline-block;
  white-space: nowrap;
  word-wrap: normal;
  direction: ltr;
  -webkit-font-feature-settings: 'liga';
  -webkit-font-smoothing: antialiased;
}
```
然后在 `assets` 目录创建一个 `font` 文件夹放我们的字体文件：
![](https://upload-images.jianshu.io/upload_images/5834506-79a18701fcf60707.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来我们实验一下能不能成功，在 `components` 新建一个 `expansion.vue` 文件，然后从 `vuetify` 官网复制的例子：
```
// expansion.vue
<template>
  <v-app>
    <v-expansion-panel>
      <v-expansion-panel-content v-for="(item,i) in 5" :key="i">
        <div slot="header">Item</div>
        <v-card>
          <v-card-text>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.</v-card-text>
        </v-card>
      </v-expansion-panel-content>
    </v-expansion-panel>
  </v-app>
</template>
```
在 `pages` 的 `index.vue` 文件中引入我们的组件：
```
// index.vue
<template>
  <div>
    <expansion></expansion>
  </div>

</template>

<script>
import Expansion from '~/components/expansion.vue'

export default {
  components: {
    Expansion
  }
}
</script>

<style>

</style>
```
![](https://upload-images.jianshu.io/upload_images/5834506-fb441e054522d0c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

嗯，很好的工作了，接下来我们打包放到服务器上看看
执行：`npm generate` 就会打包成文件夹 `dist`，里面放着我们的静态文件
![](https://upload-images.jianshu.io/upload_images/5834506-1a6388e0a9293b19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
把这个文件夹整个放到服务器上
![](https://upload-images.jianshu.io/upload_images/5834506-8aa4256b4eeb6cf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
怎么了，我们的图标怎么不见了？ F12 打开看一下
![](https://upload-images.jianshu.io/upload_images/5834506-e3b543c3eacaab6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

报错没有找到资源，我们仔细看看，发现，路径是以 `http://recallhy.xin:8080/_nuxt/` 开头的，我们把资源放到的是 `dist` 目录下，而不是根目录下，当然会找不到资源了，所以我们要修改一下路径，在 `nuxt.config.js` 里，加入一段代码：
```
module.exports = {
  ...
  router:{
    base: './'
  }
}
```
然后我们再把 `npm generate` 生成的东西放到服务器
![](https://upload-images.jianshu.io/upload_images/5834506-63cbec15235bc2bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到我们的资源可以找到了。

其实 `vuetify` 官方已经配好了 `nuxt` 的框架，直接就可以使用组件不用这么麻烦的配置，具体请看 [官方github](https://github.com/vuetifyjs/nuxt)，但是我们配其他框架套路也是一样的。

暂时就是这样，以后在使用 `nuxt` 的过程中遇到问题会持续记录。
