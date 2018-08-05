---
title: Vue组件开发流程
date: 2018-06-12 21:19:17
tags: vue
categories: 前端
---

# 需求分析
首先分析好需求，确定做什么，怎么做。
画流程图或原型图能够帮我们梳理一下重点，难点。
推荐工具：[ProcessOn](https://www.processon.com/)、[Axure](https://www.axure.com.cn/)
# 开发组件
![](https://upload-images.jianshu.io/upload_images/5834506-dbcf1306acda8a92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
推荐组件：[awasome-vue](https://github.com/vuejs/awesome-vue#components--libraries)
# 测试
开发完组件后需要测试一下，这里使用  `npm link ` 的方式，简单快速。
首先我们在开发组件的项目下，打开命令行，输入： `npm link`
![](https://upload-images.jianshu.io/upload_images/5834506-4ce00fcfda102815.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个时候就会提示已经连接到项目  `avatar`  下
然后我们新建一个项目，在新项目目录下，打开命令行，输入：`npm link avatar`，

![](https://upload-images.jianshu.io/upload_images/5834506-65e041ecace2416a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

连接成功，在 VsCode 里面，也可以在 `node_modules`看到被连接的项目
![](https://upload-images.jianshu.io/upload_images/5834506-23000a844b54e303.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后我们就正常引入组件，在新的项目里面测试看看功能是否正常

# 打包
## package.json
### 修改
`package.json` 有几个地方需要修改的，因为我们要发布组件，所以 `private` 选项要改成 `false`，

![](https://upload-images.jianshu.io/upload_images/5834506-563a413cce843375.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



然后把 `name`，`version`，`description`，`author`，看自己的情况修改。
注意：`name` 字段如果和 npm 上面已有的包重名，会发布不成功的。以后修改了组件但是发布的时候 `version` 没有修改也会发布失败。

### 添加
1. 添加 `files` 字段，这个字段是指定打包之后，包中存在的文件夹
  ![](https://upload-images.jianshu.io/upload_images/5834506-d470727f6a98e619.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 添加 `main` 字段，这个字段是当我们用 `import avatar from 'vue-crop-avatar'` 时，会引入的文件
  ![](https://upload-images.jianshu.io/upload_images/5834506-296b5fe452e531a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  即我们用 `import avatar from 'vue-crop-avatar'` 实际上是引入 `vue-avatar.min.js` 这个文件
3. 添加 `keywords` 字段，这个字段可以在 npm 上搜索的时候更容易搜索到这个包
  ![](https://upload-images.jianshu.io/upload_images/5834506-9c6fe76f3ecf8527.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4. 添加 `license` 字段，代表使用的协议
  ![](https://upload-images.jianshu.io/upload_images/5834506-5618d4275ffd7ec2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
5. 添加 `repository` 字段，即 Github 地址
```
"repository":{
  type: "git",
  url: "git+https://github/com/your/repository"
}
```
## index.js
我们在 `src` 目录下新建一个 `index.js` 的文件，这个文件主要是用来给 webpack 设置入口的，
```
// index.js
import Avatar from './components/Avatar.vue'

Avatar.install = function(Vue){
  Vue.component(Avatar.name,Avatar);
}
export default Avatar
```
主要是定义了 `install` 方法，这个方法会在 `Vue.use()` 的时候被调用，其实就是在 Vue 下注册一个组件。
## 配置webpack
我们自己在 `build` 目录下新建一个 `webpack.generate.conf.js` 文件：
```
// webpack.generate.conf.js
'use strict'
const path = require('path')
const utils = require('./utils')
const config = require('../config')
const vueLoaderConfig = require('./vue-loader.conf')

const ExtractTextPlugin = require("extract-text-webpack-plugin");
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

var webpack = require('webpack');

function resolve (dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
  entry: {
    app: './src/index.js'
  },
  output: {
    path: config.build.assetsRoot,
    filename: 'vue-crop-avatar.min.js',
    publicPath: '/dist/',
    library: 'vue-crop-avatar',
    libraryTarget: 'umd',   // 输出格式
    umdNamedDefine: true    // 是否将模块名称作为 AMD 输出的命名空间
  },
  externals: {
    vue: {
        root: 'Vue',
        commonjs: 'vue',
        commonjs2: 'vue',
        amd: 'vue'
    }
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    }
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test'), resolve('node_modules/webpack-dev-server/client')]
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      },
    ],

  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify('production')
      }
    }),
    new webpack.optimize.UglifyJsPlugin( {
      minimize : true,
      sourceMap : false,
      mangle: true,
      compress: {
        warnings: false
      }
    }),
  ]
}
```
之所以不在 `webpack.prod.conf.js` 的基础上修改，是因为我们发布组件不需要太多的配置，这样就足够了，需要注意的配置也就只有 `entry` 字段和 `output` 字段。
`entry` 字段需要改成我们之前新建的 `index.js`，这样 webpack 就会根据这个文件把依赖的东西都打包。 
`output` 字段就是输出的东西，
```
  output: {
    path: config.build.assetsRoot,
    filename: 'vue-crop-avatar.min.js', // 输出的名字
    publicPath: '/dist/', // 输出的目录
    library: 'vue-crop-avatar', // 引入组件时的名字
    libraryTarget: 'umd',   // 输出格式
    umdNamedDefine: true    // 是否将模块名称作为 AMD 输出的命名空间
  },
```
其中，输出格式有一些区别，就是各种规范，可以看这篇文章[从Webpack的Output看模块规范 ](https://recallhyx.github.io/2018/06/13/%E4%BB%8EWebpack%E7%9A%84Output%E7%9C%8B%E6%A8%A1%E5%9D%97%E8%A7%84%E8%8C%83/)

## 命令
我们在 `package.json` 的 `script` 字段可以自己写命令，
```
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js ",
    "start": "npm run dev",
    "build": "node build/build.js",
    "build-component": "webpack --config build/webpack.generate.conf.js " // 自己写的
  },
```
就是指定 webpack 用哪个配置文件进行打包
然后我们
`npm run build-component`
就能在项目里面看到新增 `dist` 目录和输出的文件。
# 发布
要发布到 npm 首先要有 npm 的账号，到[官网](https://www.npmjs.com/)注册一个账号
在项目下 `npm login`，输入用户名，密码，然后 `npm publish` ，如果失败了，请修改 `package.json` 文件中的 `name` 和 `version`。
# 使用
先将我们之前 `npm link` 取消，用 `npm unlink avatar` 就能解绑，然后我们输入 `npm install vue-crop-avatar` 即 `package.json` 里面的 `name` 字段的内容
```
<template>
  <div class="center">
    <vue-crop-avatar
      :isRound="true"
      :minCropWidth="200"
      :minCropHeight="200"
      :maxCropWidth="500"
      :maxCropHeight="500"
      @getrawdata="getCanvasData"
      :defaultImgPath="'../../static/logo.png'"
    ></vue-crop-avatar>
  </div>
</template>

<script>
import VueCropAvatar from 'vue-crop-avatar'
export default {
  name: 'HelloWorld',
  components:{
    VueCropAvatar
  },
}
</script>
```
![](https://upload-images.jianshu.io/upload_images/5834506-c954d79e40411450.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到能够正常使用了。
# 最后感悟
因为是第一次发布组件，整个过程中踩的坑比开发组件中踩的坑还要多，但是踩过坑之后，解决问题就很快了