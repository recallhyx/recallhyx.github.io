---
title: 配置Jenkins
date: 2017-10-19 21:01:57
categories: 运维
tags: Jenkins
---
哈哈哈，终于搞定持续集成的配置了，买来的阿里云服务器也能发挥一点作用了，来记录一下怎么在阿里云服务器上配置持续集成
#### 首先，什么是持续集成
简单来说，就是代码提交到 `Git` 或者 `SVN` 上的时候，会帮你`build`项目，然后跑测试，如果在build项目或者跑测试的时候出现错误，构建就会失败，然后能够通知你哪里出错了，如果没有错误，就会帮你发布项目（这么一看是不是很爽，每次提交代码就能自动构建）
我们使用的持续集成工具就是 [Jenkins](https://jenkins.io/)
#### 下载Jenkins
![](http://upload-images.jianshu.io/upload_images/5834506-a52fd187509d35b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们下载的是 `.war`后缀的，下载下来是一个包，需要装 `Java` 才能进行操作，怎么在服务器上装 `Java` 就不详细描述了
#### 启动Jenkins
下载好 `Jenkins` ，安装且配置好 `Java` 后，我们只需要输入这条语句
`java -jar jenkins.war` 就能启动`Jenkins`
第一次启动 `Jenkins`会给一个秘钥

![](http://upload-images.jianshu.io/upload_images/5834506-89e8812552c09bad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果忘记了秘钥，就可以到`.jenkins/secrets/initialAdminPassword`这个文件找
__注意：这个文件是隐藏的，要在命令行用`ls -a`这个命令来显示__
启动后，就可以在浏览器输入网址来看，默认是8080端口
`http://localhost:8080`

__注意：如果是把jenkins放在tomcat上，那么进入jenkins的网址就是 `http://localhost:8080/jenkins`__

![](http://upload-images.jianshu.io/upload_images/5834506-8915095ca986ce1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
把上一步得到的秘钥输进去就能进到`jenkins`的页面了


安装推荐的插件就够用了， 之后也能自己下载
![](http://upload-images.jianshu.io/upload_images/5834506-a2432f46e235c8f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这就是界面了
#### Github配置
由于我们使用Github来做代码管理的，所以我们在自己的服务器上安装Git后生成ssh，添加到我们Github仓库

![](http://upload-images.jianshu.io/upload_images/5834506-bb2c1dad9c422e11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 新建项目

![](http://upload-images.jianshu.io/upload_images/5834506-65dd69fc4b508b18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
新建一个自由风格的项目

![](http://upload-images.jianshu.io/upload_images/5834506-db8d059fa1032f9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击保存，退出到主界面，点击立即构建

![](http://upload-images.jianshu.io/upload_images/5834506-4f7339c56eceebeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
就构建好了
在`/root/.jenkins/workspace`可以看到我们的项目
以后每次提交代码都会触发构建，至于发布都要写在配置里面的Excute Shell里面，也可以写成一个脚本文件，到时候执行这个脚本

__总的来说是挺多坑坑坑坑坑坑坑的，国庆配这个jenkins配得怀疑人生，有很多坑其实是脚本的坑，要自己多尝试__
