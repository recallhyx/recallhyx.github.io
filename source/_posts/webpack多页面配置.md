---
title: webpack多页面配置
date: 2018-08-05 22:40:54
tags: webpack
categories: 前端
---

# 前言
之前一直用 vue 开发单页面应用，最后打包生成的只有一个 .html 和 几个 js 文件，然后把生成的 js 文件都嵌进这个页面。但是在实习的时候遇到的项目是用 vue 开发多页面应用，这让我措手不及，还是要记录一下。
# 多页面 VS 单页面
## 多页面
### 优点：首屏渲染快，SEO 效果好。
为什么多页应用的首屏时间快？
首屏时间叫做页面首个屏幕的内容展现的时间，当我们访问页面的时候，服务器返回一个 html，页面就会展示出来，这个过程只经历了一个 http 请求，所以页面展示的速度非常快。
为什么搜索引擎优化效果好（SEO）？
搜索引擎在做网页排名的时候，要根据网页内容才能给网页权重，来进行网页的排名。搜索引擎是可以识别 html 内容的，而我们每个页面所有的内容都放在 html 中，所以这种多页应用，SEO 排名效果好。
### 缺点：路由跳转慢
因为每次跳转都需要发出一个 http 请求，如果网络比较慢，在页面之间来回跳转时，就会发现明显的卡顿。
## 单页面
### 优点：路由切换快
第一次进入页面的时候会请求一个 html 文件和其他 js 文件。切换路由，其实就是切换到其他组件，此时路径也相应变化，但是并没有新的html文件请求，页面内容也变化了。
原理是：js 会感知到 url 的变化，通过这一点，可以用 js 动态的将当前页面的内容清除掉，然后将下一个页面的内容挂载到当前页面上，这个时候的路由不是后端来做了，而是前端来做，判断页面到底是显示哪个组件，清除不需要的，显示需要的组件。这种过程就是单页应用，每次跳转的时候不需要再请求 html 文件了，这样就节约了很多http发送时延。
### 缺点：首屏时间慢，SEO差
单页应用的首屏时间慢，首屏时需要请求一次 html ，同时还要发送一次 js 请求，两次请求回来了，首屏才会展示出来。相对于多页应用，首屏时间慢。
SEO 效果差，因为搜索引擎只认识 html 里的内容，不认识 js 的内容，而单页应用的内容都是靠 js 渲染生成出来的，搜索引擎不识别这部分内容，也就不会给一个好的排名，会导致单页应用做出来的网页在百度和谷歌上的排名差。
## 如何选择 
选择单页面还是多页面主要看场景
1.大部分后台系统，ToB 系统，等对数据操作，页面切换频繁，比较注重的是操作的流畅性，spa来做当然会更合适。
2. 很多ToC 项目，追求的是首屏的展现速度，注重的是观看体验，操作较少，就比较适合多页面，配合服务端渲染。
  但是现在单页面应用框架不断发展，已经有很多解决方案了，比如 SEO， vue 有 `nuxt` ，react 有 `next`。首屏展现速度，可以用路由懒加载。相比之下，多页面应用却不能解决每次跳转路由重新下载资源的问题，所以现在单页面应用是一个趋势，但是为啥还要讲 webpack 的多页面配置，（~~因为还要维护旧项目啊~~）不同场景下有不同的解决方案。
# webpack 多页面配置
## 目录
进入正题，首先，我们要把目录搞好
![](https://upload-images.jianshu.io/upload_images/5834506-e07dc391947a222c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


关注 `src` 目录，因为我们是多页面，所以 `moduleA` 和 `moduleB` 是一个模块，`insideA` 和 `insideB` 就是我们实际的页面，外层有一个大的 `module` 包住 `moduleA` 和 `moduleB`，之后会解释。
可以看到，一个页面，即 `insideA` 这个文件夹，至少需要三个文件，分别是 `app.vue`，`index.html`，`index.js`，内容其实很简单：
```
// app.vue
<template>
  <div>this is from moduleA</div>  
</template>

<script>
export default {
  
}
</script>

<style>
</style>
```
```
// index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>moduleA</title>
</head>
<body>
<div id="app"></div>
</body>
</html>
```
```
// index.js
import Vue from 'vue';
import App from './app';

new Vue({
  el: '#app',
  template: '<App/>',
  components: { App },
});
```
因为每次路由跳转都是一个新的页面，所以每个页面都需要一个 vue 实例，如果需要自己写多个 vue 文件，只需要想写单页面文件那样，把写好的 vue 文件 以组件的形式引入到 `app.vue` 文件里面就能正常使用了。
## webpack entry
因为我们是多页面，所以 `webpack.base.conf.js` 里面的 `entry` 需要改一下：
```
// webpack.base.conf.js
module.exports = {
  ...
  // entry: {
  //   app: './src/main.js'
  // },
  entry: entries,
  ...
}
```
这个 `entries` 就是我们的目录，这个时候我们就需要一个方法 `getEntry` 来获取所有打包的入口，我们在 `utils` 文件里面写这个函数：
```
// utils.js
// Get entry list from global path
const glob = require('glob');
exports.getEntry = function (globPath) {
  const entries = {};

  glob.sync(globPath).forEach(function (entry) {
    const tmp = entry.split('/').splice(-3);
    const pathname = `${tmp.splice(0, 1)}/${tmp.splice(0, 1)}`;
    entries[pathname] = entry;
    console.log(chalk.green('module: ', pathname));
    console.log(chalk.cyan('template: ', entry));
  });
  return entries;
};
```
然后我们这么用：
```
const entries = utils.getEntry('./src/'+config.dev.tag+'/**/*.js');
```
最后得到的结果：
```
{ 'moduleA/insideA': './src/module/moduleA/insideA/index.js',
  'moduleB/insideB': './src/module/moduleB/insideB/index.js' }
```
注意到 `config.dev.tag`，上面说过为什么要用 `module` 包含 `moduleA` 和 `moduleB`，因为我们在开发的时候，有时只改动一个文件，却要编译整个项目，而且整个项目编译一次要等好久，拉低开发效率，所以我们有一个 `tag` ，只编译 `tag` 目录下的文件，这个时候只会把 `tag` 下的文件当成 `entry`，减少编译的文件，加快我们的开发。设置这个 `tag` 也很简单，在 `config/index.js`：
```
// index.js
module.exports = {
  dev: {
    ...
    tag: 'module',
    ...
  }
}
```
这个时候就只会打包 `module` 目录下的文件了。
## HtmlWebpackPlugin
要知道，多页面应用最特别的就是，每个页面有自己的 js 和 公共的 js，彼此能访问到公共的 js，但是访问不了别人特有的 js，用 webpack 的 `HtmlWebpackPlugin` 就能帮我们解决这一问题：
```
// webpack.dev.conf.js
const pages = utils.getEntry('./src/'+config.dev.tag+'/**/*.html');
Object.keys(pages).forEach((pathname) => {
  const conf = {
    filename: `${pathname}.html`,
    template: pages[pathname],
    minify: {
      removeComments: true,
    },
    inject: 'body',
    cache: true,
    chunks: [pathname],
  };
  devWebpackConfig.plugins.push(new HtmlWebpackPlugin(conf));
});
```
循环文件，设置好之后再 `push` 进 `devWebpackConfig` 的 `plugins` 里面，`chunks` 里面就是插进 `html` 里面的 js，可以把公共的 js 放到里面。
## dev-server
其实就是开启服务器调试环境：
```
require('./check-versions')();

const config = require('../config');

if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV);
}

const opn = require('opn');
const path = require('path');
const express = require('express');
const webpack = require('webpack');
const proxyMiddleware = require('http-proxy-middleware');
const webpackConfig = process.env.NODE_ENV === 'testing'
  ? require('./webpack.prod.conf')
  : require('./webpack.dev.conf');
console.log("process.env.NODE_ENV",process.env.NODE_ENV);
// default port where dev server listens for incoming traffic
const port = process.env.PORT || config.dev.port;
// automatically open browser, if not set will be false
const autoOpenBrowser = config.dev.autoOpenBrowser;
// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
//const proxyTable = config.dev.proxyTable;

const app = express();
const compiler = webpack(webpackConfig);

const devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: false,
});

const hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {
  },
});
// force page reload when html-webpack-plugin template changes
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' });
    cb();
  });
});

// proxy api requests
// Object.keys(proxyTable).forEach(function (context) {
//   let options = proxyTable[context];
//   if (typeof options === 'string') {
//     options = { target: options };
//   }
//   app.use(proxyMiddleware(options.filter || context, options));
// });

// handle fallback for HTML5 history API
const connectHistoryApiFallbackConf = {
  // logger: console.log.bind(console),
  // verbose: true,
  rewrites: [{
    from: /^\/\w+\/\w+$/,
    to: function(context) {
      return context.parsedUrl.pathname + ".html";
    }
  }]
}
app.use(require('connect-history-api-fallback')(connectHistoryApiFallbackConf));

// serve webpack bundle output
app.use(devMiddleware);

// enable hot-reload and state-preserving
// compilation error display
app.use(hotMiddleware);

// serve pure static assets
const staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory);
app.use(staticPath, express.static('./statics'));

// const uri = `http://localhost:${port}`;
const uri = `http://0.0.0.0:${port}`;

devMiddleware.waitUntilValid(function () {
  console.log(`> Listening at ${uri}\n`);
});

module.exports = app.listen(port, function (err) {
  if (err) {
    console.log(err);
    return;
  }

  // when env is testing, don't need open it
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri);
  }
});
```
在 webpack-dev-server 里使用路由，能够实现访问 /a 时候显示 a.html，/b 显示 b.html。
## package.json
就是把 `script` 里面的 `npm run dev` 改成 `node --optimize_for_size build/dev-server.js` 启动我们的 webpack 服务器。
# 效果
![](https://upload-images.jianshu.io/upload_images/5834506-25067bc22127c364.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5834506-86531c5dfa34916a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>参考
>[使用webpack构建多页面应用](https://segmentfault.com/a/1190000005920125)
>[单页应用和多页应用](https://www.jianshu.com/p/4c9c29967dd6)
>[webpack搭建多页面系统（三） 理解webpack.config.js的四个核心概念](https://segmentfault.com/a/1190000012744818)


