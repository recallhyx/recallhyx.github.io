---
title: Angular2移除console
date: 2017-09-04 19:19:16
tags: angular2
categories: 前端
---
(~~拖更一个月啦，又来记录一些在项目开发中用到的东西~~)
这是只是一个小技巧而已
***
我们在Angular开发过程中，都会用`console.log()`这个函数来输出一些东西
在开发完成后，我们想去掉 `console.log()` ，当然可以一个个找到并删除，但是这样太费时费力了，我也试过在配置文件 `tslint.json`里面加入 `"no-console":{true,"log"}`这条规则来直接去除，但是也没什么卵用，后来找到一种方法，下面为介绍这种方法
首先，我们先找到 `environment.ts` 这个文件 ，我的是在 `src->app`目录下

![](http://upload-images.jianshu.io/upload_images/5834506-84af913b77dda1e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
让我们点开来看看：

![](http://upload-images.jianshu.io/upload_images/5834506-d8f3346e2a65e042.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实`environment.ts`这个文件是用来根据当前是处于开发环境还是发布环境来进行一些设置的，我们看到`if('production' === ENV)` 就是说，当当前的 __环境__ 是 __production__ 也就是发布环境的时候，进行一些操作，对应的`else`就是说在开发环境，所以我们要在 __发布环境__做一些东西

![](http://upload-images.jianshu.io/upload_images/5834506-20c9082cf35c1a87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其实只要这一句`window.console.log=function () {}`这句话的意思就是 替换 `console.log()`这个函数为空，为空就不会进行任何操作了嘛
这样的好处是：
__1、原代码不用进行任何改动__
__2、在开发环境中`console.log()`这个函数还是能正常工作__