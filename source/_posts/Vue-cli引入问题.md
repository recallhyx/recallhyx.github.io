---
title: Vue-cli引入问题
date: 2018-05-25 22:08:07
tags: vue
categories: 前端
---

# 前言
最近在各种方面的引入时出现了一些问题。比如，用到剪切图片的第三方库  [cropperjs](https://github.com/fengyuanchen/cropperjs)，但是在引入的时候却不能正常显示。

# 引入第三方库的问题
一开始像官网的写法一样
先 `npm install cropperjs`
然后在 `index.html` 里面插入以下代码
```
    <link  href="./node_modules/cropperjs/dist/cropper.min.css" rel="stylesheet">
    <script src="./node_modules/cropperjs/dist/cropper.esm.js"></script>
```
但是效果却是这样的：
![](https://upload-images.jianshu.io/upload_images/5834506-0a1cbad8185bf800.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5834506-186a4ff836610295.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
而且页面拉到下面，滚动滚轮的时候下面两张图还会放大缩小，根本就不是想要的效果(╯‵□′)╯︵┻━┻。

如果我们用把 `index.html` 中的代码换成 cdn：
```
    <link  href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.3.6/cropper.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.3.6/cropper.js"></script>
```
![](https://upload-images.jianshu.io/upload_images/5834506-fe87cb7b81098431.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结果又正常引入了

# 解决方法
其实在 Vue-cli 里面，引入插件需要在组件里面的两个地方引入：在 `<script>` 标签中引入 js 文件，在 `<style>`  标签中引入 css 文件：
```
// component.vue
<template>
...
</template>
<script>
import  Cropper from 'cropperjs' // 引入 js 文件
...
</script>
<style>
@import '../../node_modules/cropperjs/dist/cropper.min.css'; // 引入 css 文件
...
</style>
```
删掉 `index.html` 里面的引入，看看效果：
![](https://upload-images.jianshu.io/upload_images/5834506-8130fd9580871e4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功啦。
如果想全局使用，还是一样的，只不过引入位置的组件改为 `App.vue`。

# 图片路径的问题
一开始想着用 Vue-cli 自带的 `logo.png` 图片做测试，但是在直接对图片的 `src` 属性赋值的时候 `img.src = ../assets/logo.png` 图片却找不到，而在 `<img>` 标签里面的 `src` 属性赋值图片却可以正常显示。

# 解决方法
1. 使用 `require` ，因为在 `assets` 中的文件会被 `webpack` 处理，所以要使用的话就需要 `img.src = require('../assets/logo.png')`
2. 把 `logo.png` 放到 `static` 目录下，因为在 `static` 目录下的文件不会被webpack 处理，所以通过正常的方式就能直接使用 `img.src = '/static/logo.png'` 使用绝对路径引用。