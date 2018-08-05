---
title: Python调用C++的方式
date: 2017-09-22 21:47:18
categories: 后端
tags: python
---
（~~国庆前夕来完成每月一篇博文的任务~~）
这次来讲讲怎么将 `c++`和 `python` 结合在一起，主要是由 `python` 来调用 `c++`的函数    
__首先__，我们需要一个东西 [swig](http://www.swig.org/) （~~这网站看起来很旧，但其实swig是有在更新的~~）

![](http://upload-images.jianshu.io/upload_images/5834506-056de9539dfec04c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击下载最新版就行了

__什么是SWIG ?__

>SWIG是一个能将C或者C++编写的程序与其它各种高级语言如Perl, Python, Ruby, 和 Tcl进行联接的开发工具。其原理是从C/C++头文件中找到申明并利用他们生成脚本语言访问C/C++代码所必须的封装代码。SWIG具有高度可自定义的特点，它能帮助你生成适合你的应用程序的封装包。

所以，我们借助 `SWIG` 把 `c++`代码编译成 `python` 能够引用的模块，就能在`python`里调用`c++`了

下载解压完成之后我们还要配环境变量

![](http://upload-images.jianshu.io/upload_images/5834506-02214127a877ddba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样第一步就行了
__第二步__，我们要先写 `c++` 的函数
我们需要三个文件
`hello.h------------头文件` 
`hello.cpp---------源文件`
`hello.i-------------配置文件`

我们在`hello.h`声明一个函数 `Hello(char *str)`
*这里不能用string，python调用有问题，具体不清楚，参数为char \*则可以调用*


![](http://upload-images.jianshu.io/upload_images/5834506-421df0c43cb5b265.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后在 `hello.cpp`实现这个函数

![](http://upload-images.jianshu.io/upload_images/5834506-96bbd0f953b2ae24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后配置文件`hello.i`

![](http://upload-images.jianshu.io/upload_images/5834506-145236b8fcec6483.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`#define SWIG_FILE_WITH_INIT`这句规定了要被编译成 `python`的模块
`#include "hello.h"`给出需要包含的头文件
`void Hello(char * str);`在 `hello.i`这个文件的最后给出想要编译的函数
__第三步__，使用命令编译打包
在命令行输入
`swig -python -c++ hello.i`
编译，成功之后就会生成`hello_wrap.cxx`这个文件
__第四步__，我们还需要写一个名为`setup.py`的python文件

![](http://upload-images.jianshu.io/upload_images/5834506-7fc3f187a18b4a18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在第三步打包的模块还缺少动态链接库，所以我们需要这个 `setup.py`来生成python使用的动态链接库
在命令行中输入
`python setup.py build_ext --inplace`
成功后会生成`_hello.cp35-win_amd64.pyd`这个文件
__第五步__，把生成的两个文件都放到要用的python目录下，就可以在python里面调用了

![](http://upload-images.jianshu.io/upload_images/5834506-4ddbff4031cc6acd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/5834506-5f559dcc48bd189b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
__大功告成 !成功的调用了c++的函数，大家可以自行发挥了__

