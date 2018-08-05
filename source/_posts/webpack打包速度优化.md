---
title: webpack打包速度优化
date: 2018-07-14 10:24:03
tags: webpack
categories: 前端
---

# 前言
webpack 打包速度一直以来是个问题，之前用 webpack 打包 Angular 要等个几分钟，所以需要优化一下打包速度。

# 前期准备
我们需要准备一个很大的项目，去网上找几个 UI 库安装下载就差不多了
![](https://upload-images.jianshu.io/upload_images/5834506-45fc0301d04f28f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# HappyPack 
我们就看一下 `HappyPack` 是怎么配置的。
首先，`HappyPack` 是个插件来的，所以我们要在 webpack 的 `plugins` 里面 `new` 一个：
![](https://upload-images.jianshu.io/upload_images/5834506-b2d9cc383136adc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`happyThreadPool` 就是要用多少子进程来进行编译，我们在文件头声明一个变量
```
var happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });
```
这个就是获取操作系统的 cpu。
然后我们就把原来的 `loader` 替换：

![](https://upload-images.jianshu.io/upload_images/5834506-5d50e0bf36c3572a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

前面是一样的，后面的 `id` 就是指定的 `HappyPack` 配置，也就是说我们可以有多个配置，只需要改变 `id` 和 `loader` 就能启用 `HappyPack`。

现在，我们把 `HappyPack` 去掉，看一下打包要花多久：
![](https://upload-images.jianshu.io/upload_images/5834506-a6624387a1889575.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

快了 10 秒，效果还不错。
# DllPlugin
那我们再用另外一个插件 `DllPlugin` ， 它要和 `DllReferencePlugin ` 配合使用。前者是用来打包 `dll.js`，后者是用在打包主流程时引用刚才的 `dll.js` 的。
`Dll` 是动态链接库的意思，实际上就是将这些 `npm` 打包生成一个 `JSON` 文件，这个文件里包含了 `npm` 包的路径对应信息。
需要新建一个文件夹 `webpack.dll.conf.js` 来使用第一个插件 `DllPlugin`：
```
const webpack = require('webpack');
const path = require('path');

module.exports = {
    output: {
        // 将会生成 lib.js文件
        path: path.resolve(__dirname, '../statics/'),
        filename: '[name].js',
        library: '[name]',
    },
    entry: {
        "lib": [
          // ...其它库
          'vue/dist/vue.esm.js',
          'echarts',
          'element-ui',
          'iview',
          'vue-material',
          'vuetify',
        ],
    },
    plugins: [
        new webpack.DllPlugin({
            // 生成的映射关系文件
            path: path.resolve(__dirname, '../statics/[name]-mainfest.json'),
            name: '[name]',
            context: __dirname,// 执行的上下文环境，对之后DllReferencePlugin有用
        }), 
    ],
};
```
在 `package.json` 里面写个脚本：
```
  "scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "build": "node build/build.js",
    "build-dll": "webpack --config build/webpack.dll.conf.js"
  },
```
执行 `npm run build-dll`，就会生成下面的文件： 
![](https://upload-images.jianshu.io/upload_images/5834506-48dd61bfb063eb52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一步完成，然后我们在 `webpack.prod.conf.js` 里面添加我们的第二个插件 `DllReferencePlugin` ：
```
  plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      manifest: require('../statics/lib-mainfest.json') // 指向这个json
    }),
  ]
```
然后就可以啦，让我们试试添加插件之后的打包速度：
![](https://upload-images.jianshu.io/upload_images/5834506-22ec88cae463ae35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

变成 30 秒了， `HappyPack` 和 `DllPlugin` 一起用比什么都没有快了一倍 ！减少构建时间能让我们更快的开发调试，还是挺有用的。

