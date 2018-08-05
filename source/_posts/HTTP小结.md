---
title: HTTP小结
date: 2018-03-19 21:23:04
tags: http
categories: HTTP
---
# 前言
主要是对比HTTP1.0，HTTP1.1 ，HTTPS，SPDY，HTTP2.0，还有对HTTP缓存的一些记录

# HTTP1.0，HTTP1.1 ，HTTPS，SPDY，HTTP2.0
## HTTP1.0 与 HTTP1.1 区别
HTTP1.0最早在网页中使用是在 1996 年，那个时候只是使用一些较为简单的网页上和网络请求上，而HTTP1.1则在 1999 年才开始广泛应用于现在的各大浏览器网络请求中，同时 HTTP1.1 也是当前使用最为广泛的HTTP协议。 主要区别主要体现在：
1. 缓存处理，在HTTP1.0中主要使用header里的 If-Modified-Since , Expires 来做为缓存判断的标准，HTTP1.1 则引入了更多的缓存控制策略例如 Entity tag，If-Unmodified-Since ,  If-Match ,  If-None-Match 等更多可供选择的缓存头来控制缓存策略。

2. 带宽优化及网络连接的使用，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了 range 头域，它允许只请求资源的某个部分，即返回码是 206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

3. 错误通知的管理，在HTTP1.1中新增了 24 个错误状态响应码，如 409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

4. Host头处理，在 HTTP1.0 中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1 的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

5. 长连接，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启 Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

## HTTP 与 HTTPS 的一些区别

1. HTTPS 协议需要到 CA 申请证书，一般免费证书很少，需要交费。

2. HTTP 协议运行在 TCP 之上，所有传输的内容都是明文，HTTPS 运行在 SSL/TLS 之上，SSL/TLS 运行在 TCP 之上，所有传输的内容都经过加密的。

3. HTTP 和 HTTPS 使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

4. HTTPS 可以有效的防止运营商劫持，解决了防劫持的一个大问题。

5. HTTP是 3 次握手建立连接，HTTPS的话则是 9 次握手
![](http://misc.linkedkeeper.com/misc/img/blog/201611/linkedkeeper0_e09e93a0-5e7f-47db-b332-9c8ce3e0a405.jpg)

### HTTPS 9次握手

![](https://elliotsomething.github.io/images/post-SSL-01.png)

(1) 客户端通过 `Client Hello` 消息将它支持的SSL版本、加密算法、密钥交换算法、MAC算法等信息发送给SSL服务器。

(2) 服务器确定本次通信采用的版本和加密套件，并通过 `Server Hello` 消息通知给客户端。如果服务器允许客户端在以后的通信中重用本次会话，则服务器会为本次会话分配会话ID，并通过 `Server Hello` 消息发送给SSL客户端。

(3) 服务器将携带自己公钥信息的数字证书通过 `Certificate` 消息发送给客户端。

(4) 服务器发送 `Server Hello Done` 消息，通知客户端版本和加密套件协商结束，开始进行密钥交换。

(5) 客户端验证服务器的证书合法后，利用证书中的公钥加密客户端随机生成的premaster secret，并通过 `Client Key Exchange` 消息发送给SSL服务器。

(6) 客户端发送 `Change Cipher Spec` 消息，通知服务器后续报文将采用协商好的密钥和加密套件进行加密和MAC计算。

(7) 客户端计算已交互的握手消息（除Change Cipher Spec消息外所有已交互的消息）的Hash值，利用协商好的密钥和加密套件处理Hash值（计算并添加MAC值、加密等），并通过 `Finished` 消息发送给服务器。服务器利用同样的方法计算已交互的握手消息的Hash值，并与 `Finished` 消息的解密结果比较，如果二者相同，且MAC值验证成功，则证明密钥和加密套件协商成功。

(8) 同样地，SSL服务器发送 `Change Cipher Spec` 消息，通知客户端后续报文将采用协商好的密钥和加密套件进行加密和MAC计算。

(9) 服务器计算已交互的握手消息的Hash值，利用协商好的密钥和加密套件处理Hash值（计算并添加MAC值、加密等），并通过 `Finished` 消息发送给客户端。客户端利用同样的方法计算已交互的握手消息的Hash值，并与 `Finished` 消息的解密结果比较，如果二者相同，且MAC值验证成功，则证明密钥和加密套件协商成功。

(10) 客户端接收到服务器发送的 `Finished` 消息后，如果解密成功，则可以判断服务器是数字证书的拥有者，即服务器身份验证成功，因为只有拥有私钥的服务器才能从Client Key Exchange消息中解密得到 premaster secret，从而间接地实现了客户端对服务器的身份验证。

### HTTPS加密通信的流程
HTTPS 相对于 HTTP 有哪些不同呢？其实就是在 HTTP 跟 TCP 中间加多了一层加密层 TLS/SSL。
![](https://upload-images.jianshu.io/upload_images/5834506-d9daf52be2e2a943.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将数据加密后再传输，而不是任由数据在复杂而又充满危险的网络上明文裸奔，在很大程度上确保了数据的安全。这样的话，即使数据被中间节点截获，坏人也看不懂。
而且 HTTPS 不仅仅用非对称加密（加密数据用的密钥（公钥），跟解密数据用的密钥（私钥）是不一样的。通过公钥加密的数据，只能通过私钥解开。通过私钥加密的数据，只能通过公钥解开。），但同时也结合了其他手段，如对称加密（加密数据用的密钥，跟解密数据用的密钥是一样的），来确保授权、加密传输的效率、安全性。
具体的流程就是：
1. 小明访问网站A，网站A将自己的证书给到小明（其实是给到浏览器，小明不会有感知）。
2. 浏览器从证书中拿到网站的公钥X。
3. 浏览器生成一个只有自己自己的对称密钥Y，用公钥X加密，并传给网站A（其实是有协商的过程，这里为了便于理解先简化）。
4. 网站A通过私钥解密，拿到对称密钥Y。
5. 浏览器、网站A 之后的数据通信，都用对称密钥Y进行加密。

## SPDY 与 HTTP2.0 的联系
SPDY（发音如英语：speedy）可以说是综合了 HTTPS 和 HTTP 两者有点于一体的传输协议，主要解决：
1. 降低延迟，针对 HTTP 高延迟的问题，SPDY 优雅的采取了多路复用（multiplexing）。多路复用通过多个请求stream共享一个 tcp 连接的方式，解决了HOL blocking 的问题，降低了延迟同时提高了带宽的利用率。

2. 请求优先级（request prioritization）。多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY 允许给每个 request 设置优先级，这样重要的请求就会优先得到响应。比如浏览器加载首页，首页的 html 内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。

3. header 压缩。前面提到 HTTP1.x 的 header 很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。

4. 基于 HTTPS 的加密协议传输，大大提高了传输数据的可靠性。

5. 服务端推送（server push），采用了 SPDY 的网页，例如我的网页有一个`sytle.css` 的请求，在客户端收到 `sytle.css` 数据的同时，服务端会将 `sytle.js` 的文件推送给客户端，当客户端再次尝试获取 `sytle.js` 时就可以直接从缓存中获取到，不用再发请求了。

SPDY 位于 HTTP 之下，TCP 和 SSL 之上，这样可以轻松兼容老版本的 HTTP 协议（将HTTP1.x的内容封装成一种新的frame格式），同时可以使用已有的 SSL 功能。

后来 SPDY 未能单独成为正式标准，不过 SPDY 开发组的成员全程参与了HTTP/2的制定过程。HTTP/2 协议完成之后，Google认为 SPDY 可以功成身退了，于是最终Google Chrome 淘汰对 SPDY 的支持，全面改为采用 HTTP/2 。

HTTP2.0 可以说是 SPDY 的升级版（原本也是基于SPDY设计的），但是，HTTP2.0 跟 SPDY 仍有不同的地方，主要是以下两点：

1. HTTP2.0 支持明文 HTTP 传输，而 SPDY 强制使用 HTTPS

2. HTTP2.0 消息头的压缩算法采用 HPACK，而非 SPDY 采用的 DEFLATE

### HTTP2.0 新特性
1. 新的二进制格式（Binary Format），HTTP1.x 的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑 HTTP2.0 的协议解析决定采用二进制格式，实现方便且健壮。

2. 多路复用（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个 request 对应一个 id ，这样一个连接上可以有多个 request，每个连接的request 可以随机的混杂在一起，接收方可以根据 request 的 id 将 request 再归属到各自不同的服务端请求里面。
![](http://7jpo3s.com1.z0.glb.clouddn.com/http-6.png)
3. header 压缩，如上文中所言，对前面提到过 HTTP1.x 的header带有大量信息，而且每次都要重复发送，HTTP2.0 使用 encoder 来减少需要传输的 header 大小，通讯双方各自 cache 一份 header fields 表，既避免了重复 header 的传输，又减小了需要传输的大小。

4. 服务端推送（server push），同SPDY一样，HTTP2.0 也具有server push功能。

## 完整图
![](https://www.centos.bz/wp-content/uploads/2017/09/6-21.png)

# HTTP缓存
HTTP 头信息控制缓存大致分为两种：强缓存和协商缓存。强缓存如果命中缓存不需要和服务器端发生交互，而协商缓存不管是否命中都要和服务器端发生交互，强制缓存的优先级高于协商缓存。
我们在上面说到，HTTP1.1 引入了更多的缓存控制策略，接下来我们详细讲一下这些缓存头。

## 缓存头
1. 通用首部字段
![](https://upload-images.jianshu.io/upload_images/5834506-bb5edb1a50339232.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. 请求首部字段
![](https://upload-images.jianshu.io/upload_images/5834506-581c7c490ab43e7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 响应首部字段
![](https://upload-images.jianshu.io/upload_images/5834506-d3f4888458525488.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4. 实体首部字段
![](https://upload-images.jianshu.io/upload_images/5834506-3862ce8c2cc24ce4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## HTTP1.0 用到的缓存头
### Pragma
当该字段值为 `no-cache` 的时候（事实上现在RFC中也仅标明该可选值），会知会客户端不要对该资源读缓存，即每次都得向服务器发一次请求才行。
Pragma 属于通用首部字段，在客户端上使用时，常规要求我们往 html 上加上这段meta 元标签
```
<meta http-equiv="Pragma" content="no-cache">
```
### Expires
有了 Pragma 来禁用缓存，自然也需要有个东西来启用缓存和定义缓存时间，对http1.0 而言，Expires 就是做这件事的首部字段。 Expires 的值对应一个GMT（格林尼治时间），比如Mon, 22 Jul 2002 11:12:01 GMT来告诉浏览器资源缓存过期时间，如果还没过该时间点则不发请求。
Expires 指缓存过期的时间，超过了这个时间点就代表资源过期。有一个问题是由于使用具体时间，如果时间表示出错或者没有转换到正确的时区都可能造成缓存生命周期出错。并且 Expires 是 HTTP/1.0 的标准，现在更倾向于用 HTTP/1.1 中定义的 Cache-Control。两个同时存在时也是 Cache-Control 的优先级更高。
响应报文中 Expires 所定义的缓存时间是相对服务器上的时间而言的，如果客户端上的时间跟服务器上的时间不一致（特别是用户修改了自己电脑的系统时间），那缓存时间可能就没啥意义了。

## Cache-Control
针对上述的“ Expires 时间是相对服务器而言，无法保证和客户端时间统一”的问题，http1.1 新增了 Cache-Control 来定义缓存过期时间。注意：若报文中同时出现了 Expires 和 Cache-Control，则以 Cache-Control 为准。
也就是说优先级从高到低分别是 Pragma -> Cache-Control -> Expires 。
Cache-Control 也是一个通用首部字段，这意味着它能分别在请求报文和响应报文中使用。在RFC中规范了 Cache-Control 的格式为：`"Cache-Control" ":" cache-directive`
其中，作为请求首部时，`cache-directive` 的可选值有：
![](https://upload-images.jianshu.io/upload_images/5834506-b1c6af665b522367.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
作为响应首部时，`cache-directive` 的可选值有：
![](https://upload-images.jianshu.io/upload_images/5834506-5ae63e577ad67f1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Cache-Control 允许自由组合可选值，例如：`Cache-Control: max-age=3600, must-revalidate` 它意味着该资源是从原服务器上取得的，且其缓存（新鲜度）的有效时间为一小时，在后续一小时内，用户重新访问该资源则无须发送请求。 当然这种组合的方式也会有些限制，比如 `no-cache` 就不能和 `max-age`、`min-fresh`、`max-stale` 一起搭配使用。

以上，`Expires` 和 `Cache-Control` 都是属于强缓存，接下来介绍协商缓存

## Last-modified
服务器将资源传递给客户端时，会将资源最后更改的时间以“Last-Modified: GMT”的形式加在实体首部上一起返回给客户端。
```
Last-Modified: Fri, 22 Jul 2016 01:47:00 GMT
```
客户端会为资源标记上该信息，下次再次请求时，会把该信息附带在请求报文中一并带给服务器去做检查，若传递的时间值与服务器上该资源最终修改时间是一致的，则说明该资源没有被修改过，直接返回 304 状态码，内容为空，这样就节省了传输数据量 。如果两个时间不一致，则服务器会发回该资源并返回 200 状态码，和第一次请求时类似。这样保证不向客户端重复发出资源，也保证当服务器有变化时，客户端能够得到最新的资源。一个 304 响应比一个静态资源通常小得多，这样就节省了网络带宽。

至于传递标记起来的最终修改时间的请求报文首部字段一共有两个：
1. If-Modified-Since: Last-Modified-value
该请求首部告诉服务器如果客户端传来的最后修改时间与服务器上的一致，则直接回送304 和响应报头即可。
当前各浏览器均是使用的该请求首部来向服务器传递保存的 Last-Modified 值。
![](https://upload-images.jianshu.io/upload_images/5834506-b417d5f9d1f5c4a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. If-Unmodified-Since: Last-Modified-value
该值告诉服务器，若 Last-Modified 没有匹配上（资源在服务端的最后更新时间改变了），则应当返回412(Precondition Failed) 状态码给客户端。 Last-Modified 存在一定问题，如果在服务器上，一个资源被修改了，但其实际内容根本没发生改变，会因为 Last-Modified 时间匹配不上而返回了整个实体给客户端（即使客户端缓存里有个一模一样的资源）。

![](https://upload-images.jianshu.io/upload_images/5834506-d950395719e0986b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意：如果响应头中有 Last-modified 而没有 Expire 或 Cache-Control 时，浏览器会有自己的算法来推算出一个时间缓存该文件多久，不同浏览器得出的时间不一样，所以 Last-modified 要记得配合 Expires/Cache-Control 使用。
## ETag
为了解决上述Last-Modified可能存在的不准确的问题，Http1.1还推出了 ETag 实体首部字段。 服务器会通过某种算法，给资源计算得出一个唯一标志符（比如md5标志），在把资源响应给客户端的时候，会在实体首部加上“ETag: 唯一标识符”一起返回给客户端。例如：`Etag: "5d8c72a5edda8d6a:3239"`
客户端会保留该 ETag 字段，并在下一次请求时将其一并带过去给服务器。服务器只需要比较客户端传来的ETag跟自己服务器上该资源的ETag是否一致，就能很好地判断资源相对客户端而言是否被修改过了。
如果服务器发现ETag匹配不上，那么直接以常规GET 200回包形式将新的资源（当然也包括了新的ETag）发给客户端；如果ETag是一致的，则直接返回304知会客户端直接使用本地缓存即可。


那么客户端是如何把标记在资源上的 ETag 传回给服务器的呢？请求报文中有两个首部字段可以带上 ETag 值：
1. If-None-Match: ETag-value
告诉服务端如果 ETag 没匹配上需要重发资源数据，否则直接回送304 和响应报头即可。 当前各浏览器均是使用的该请求首部来向服务器传递保存的 ETag 值。
![](https://upload-images.jianshu.io/upload_images/5834506-eccf6a522590fddf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. If-Match: ETag-value
告诉服务器如果没有匹配到 ETag，或者收到了“*”值而当前并没有该资源实体，则应当返回412(Precondition Failed) 状态码给客户端。否则服务器直接忽略该字段。
需要注意的是，如果资源是走分布式服务器（比如 CDN）存储的情况，需要这些服务器上计算 ETag 唯一值的算法保持一致，才不会导致明明同一个文件，在服务器A和服务器B上生成的 ETag 却不一样。

## 缓存头部对比


头部  |优势和特点| 劣势和问题
---|---|---
Expires|    1、HTTP 1.0 产物，可以在HTTP 1.0和1.1中使用，简单易用。2、以时刻标识失效时间。  |1、时间是由服务器发送的(UTC)，如果服务器时间和客户端时间存在不一致，可能会出现问题。2、存在版本问题，到期之前的修改客户端是不可知的。
Cache-Control   |1、HTTP 1.1 产物，以时间间隔标识失效时间，解决了Expires服务器和客户端相对时间的问题。2、比Expires多了很多选项设置。 |1、HTTP 1.1 才有的内容，不适用于HTTP 1.0 。2、存在版本问题，到期之前的修改客户端是不可知的。
Last-Modified|1、不存在版本问题，每次请求都会去服务器进行校验。服务器对比最后修改时间如果相同则返回304，不同返回200以及资源内容。 |1、只要资源修改，无论内容是否发生实质性的变化，都会将该资源返回客户端。例如周期性重写，这种情况下该资源包含的数据实际上一样的。2、以时刻作为标识，无法识别一秒内进行多次修改的情况。3、某些服务器不能精确的得到文件的最后修改时间。
ETag|   1、可以更加精确的判断资源是否被修改，可以识别一秒内多次修改的情况。2、不存在版本问题，每次请求都回去服务器进行校验。 |1、计算ETag值需要性能损耗。2、分布式服务器存储的情况下，计算ETag的算法如果不一样，会导致浏览器从一台服务器上获得页面内容后到另外一台服务器上进行验证时发现ETag不匹配的情况。
## 用户刷新/访问行为
我们可以把刷新/访问界面的手段分成三类：
+ 在URI输入栏中输入然后回车/通过书签访问
+ F5/点击工具栏中的刷新按钮/右键菜单重新加载
+ Ctrl+F5

在浏览器中，有时候你会发现通过不同的手段访问/刷新界面页面的呈现速度是不一样的，那么它们到底有什么区别呢?
以下对这三种访问情况进行实践与讨论。
准备工作：
为了模拟第一次访问某网站，清除相关缓存内容。首次访问网页，查看请求与响应信息可以看到请求头部没有任何关于 http 缓存相关的信息。而返回的 HTTPresponse 包含了以下头部信息。
```
Cache-Control: max-age=31104000
Expires: Thu, 20 Jul 2017 02:18:41 GMT
Last-Modified: Fri, 15 Jul 2016 04:11:51 GMT
```
浏览器会对该文件进行缓存，直到该文件过期、用户清空cache或者用户强制刷新资源时间。
### 在URI输入栏中输入然后回车
我们可以看到返回响应码是 200 OK (from cache)，浏览器发现该资源已经缓存了而且没有过期（通过Expires头部或者Cache-Control头部），没有跟服务器确认，而是直接使用了浏览器缓存的内容。其中响应内容和之前的响应内容一模一样，例如其中的Date时间是上一次响应的时间。
![](https://upload-images.jianshu.io/upload_images/5834506-438b1abf929b82d2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
所以我们也能看到该资源的Size为 `from cache`
![](https://upload-images.jianshu.io/upload_images/5834506-968f175b73b0a47e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### F5/点击工具栏中的刷新按钮/右键菜单重新加载
F5的作用和直接在 URI 输入栏中输入然后回车是不一样的，F5会让浏览器无论如何都发一个 HTTP Request 给Server，即使先前的响应中有Expires头部。所以，当我在当前 网页中按F5的时候，浏览器会发送一个HTTP Request给Server，但是包含这样的Headers:
```
Cache-Control: max-age=0
If-Modified-Since: Fri, 15 Jul 2016 04:11:51 GMT
```
其中Cache-Control是Chrome强制加上的，而If-Modified-Since是因为获取该资源的时候包含了Last-Modified头部，浏览器会使用If-Modified-Since头部信息重新发送该时间以确认资源是否需要重新发送。 实际上Server没有修改这个index.css文件，所以返回了一个304(Not Modified)，这样的响应信息很小，所消耗的route-trip不多，网页很快就刷新了。
![](https://upload-images.jianshu.io/upload_images/5834506-bcfb5caf5031f69a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面的例子中没有ETag，如果Response中包含ETag，F5引发的 Http Request 中也是会包含 If-None-Match 的。
### Ctrl+F5
那么Ctrl+F5呢？ Ctrl+F5 也叫强制刷新，要的是彻底的从Server拿一份新的资源过来，所以不光要发送 HTTP request 给Server，而且这个请求里面连 If-Modified-Since/If-None-Match 都没有，这样就逼着 Server 不能返回304，而是把整个资源原原本本地返回一份，这样，Ctrl+F5 引发的传输时间变长了，自然网页 Refresh 的也慢一些。我们可以看到该操作返回了200，并刷新了相关的缓存控制时间。
![](https://upload-images.jianshu.io/upload_images/5834506-8c404ac67a27e4f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
实际上，为了保证拿到的是从Server上最新的，Ctrl+F5不只是去掉了If-Modified-Since/If-None-Match，还需要添加一些HTTP Headers。按照HTTP/1.1协议，Cache不光只是存在Browser终端，从Browser到Server之间的中间节点(比如Proxy)也可能扮演Cache的作用，为了防止获得的只是这些中间节点的Cache，需要告诉他们，别用自己的Cache敷衍我，往Upstream的节点要一个最新的copy吧。
在Chrome 51 中会包含两个头部信息， 作用就是让中间的Cache对这个请求失效，这样返回的绝对是新鲜的资源。
```
Cache-Control: no-cache
Pragma: no-cache
```
## 总结
![](https://upload-images.jianshu.io/upload_images/5834506-8d74c29f4e792e10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>参考：
[HTTP1.0、HTTP1.1 和 HTTP2.0 的区别](https://juejin.im/entry/5981c5df518825359a2b9476)
[HTTP,HTTP2.0,SPDY,HTTPS你应该知道的一些事](http://www.linkedkeeper.com/detail/blog.action?bid=152)
[http的变迁（http1.0-http2.0，https）](https://www.centos.bz/2017/09/http%E7%9A%84%E5%8F%98%E8%BF%81%EF%BC%88http1-0-http2-0%EF%BC%8Chttps%EF%BC%89/)
[SPDY](https://zh.wikipedia.org/wiki/SPDY)
[HTTP 缓存机制一二三](https://zhuanlan.zhihu.com/p/29750583)
[HTTP缓存控制小结](http://imweb.io/topic/5795dcb6fb312541492eda8c)
[浅谈浏览器http的缓存机制](http://www.cnblogs.com/vajoy/p/5341664.html)
[HTTPS科普扫盲帖](http://imweb.io/topic/56d67baaca5e865230c1d4fa)