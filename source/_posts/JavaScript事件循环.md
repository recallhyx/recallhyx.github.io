---
title: JavaScript事件循环
date: 2018-03-14 15:48:33
tags: javascript
categories: 前端
---
## 引入例子
先来个例子：
```
setTimeout(function () {
    console.log(1);
}, 0);
console.log(2);
//2
//1
```
我们都知道，调用 `setTimeout` 时，会把函数参数，放到事件队列中，等主程序运行完，再调用。所以会先输出`2`再输出`1`。不过 `setTimeout` 第二个参数是延迟多少秒，都会放到事件队列。

那我们再来看一个例子：
```
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

//Promise
//Hi!
//resolved.
```
可以看到，先是输出了 `Promise`，再输出 `Hi!`，最后才输出 `resolved.`。这是因为，`Promise` 在声明的时候就会执行，而它的方法 `then` 指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以resolved最后输出。 当然了，如果把`Promise` 中的 `resolve()` 这个函数去掉，将不会改变 `Promise`的状态，也不会触发 `then` 方法，也就不会输出 `resolved`。

如果我们将两个例子结合起来，会怎么样呢。
```
setTimeout(function () {
    console.log(1);
}, 0);
let promise = new Promise(function(resolve, reject) {
  console.log('2');
  resolve();
});

promise.then(function() {
  console.log('3');
});

console.log('4');
```
答案是输出`2 4 3 1`。有几个我们在上面的例子可以确定，因为 `Promise`一创建就会运行，所以会先输出`2`，`setTimeout` 和 `then`都是异步操作，就会放到事件队列，主线程输出`4`。那么问题来了，`setTimeout` 和 `then` 哪个先输出呢，这就引出了事件循环的概念。

## 事件循环
浏览器（或者说JS引擎）执行JS的机制是基于事件循环。
由于JS是单线程，所以同一时间只能执行一个任务，其他任务就得排队，后续任务必须等到前一个任务结束才能开始执行。
为了避免因为某些长时间任务造成的无意义等待，JS引入了异步的概念，用另一个线程来管理异步任务。
同步任务直接在主线程队列中顺序执行，而异步任务会进入另一个任务队列，不会阻塞主线程。等到主线程队列空了（执行完了）的时候，就会去异步队列查询是否有可执行的异步任务了（异步任务通常进入异步队列之后还要等一些条件才能执行，如ajax请求、文件读写），如果某个异步任务可以执行了便加入主线程队列，以此循环。
### 事件循环（Event Loop） 规范
1、每个浏览器环境，至多有一个`event loop`。
2、一个 `event loop` 可以有1个或多个任务队列（`task queue`） 。
3、一个 `task queue` 是一列有序的任务（`task`），用来做以下工作： `Events task`， `Parsing task`， `Callbacks task`， `Using a resource task`， `Reacting to DOM manipulation task` 等。
![](https://upload-images.jianshu.io/upload_images/5834506-2482a71f0144a75b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

每个 `task` 都有自己相关的文档（ `document`），比如一个 `task 在某个元素（`element`）的上下文中进入队列，那么它的 `document` 就是这个 `element` 的 `document`。

每个 `task` 定义时都有一个任务源（`task source`），从同一个 `task source` 来的 `task `必须放到同一个 `task queue`，从不同源来的则被添加到不同队列。

每个(`task source`对应的) `task queue` 都保证自己队列的先进先出的执行顺序，但`event loop` 的每个循环（`turn`），是由浏览器决定从哪个`task source`挑选`task`。这允许浏览器为不同的`task source`设置不同的优先级，比如为用户交互设置更高优先级来使用户感觉流畅。

### 工作和工作队列（Jobs and Job Queues） 规范
ES6规范里，新增了 `Jobs and Job Queues` 这一概念，它有点类似于上面提到的任务队列（`task queue`）
一个 Job Queue 是一个先进先出的队列。一个ECMAScript实现必须至少包含以下两个 Job Queue ：

| Name        | Purpose                                  |
| ----------- | ---------------------------------------- | 
| ScriptJobs  | Jobs that validate and evaluate ECMAScript Script and Module source text. See clauses 10 and 15. |
| PromiseJobs | Jobs that are responses to the settlement of a Promise (see 25.4). |
单个 `Job Queue` 中的 `PendingJob` 总是按序（先进先出）执行，但多个 `Job Queue` 可能会交错执行。
跟随PromiseJobs到25.4章节，可以看到 [PerformPromiseThen ( promise, onFulfilled, onRejected, resultCapability )](http://ecma-international.org/ecma-262/6.0/index.html#sec-performpromisethen) ：

这里我们看到， `promise.then` 的执行其实是向 `PromiseJobs` 添加Job。
### task（macro-task）和micro-task
`micro-task` 在 ES6 规范中称为 `Job`。 其次，`macro-task` 代指 `task`。

有一个事件循环，但是任务队列可以有多个。
整个 script 代码，放在了 `macro-task queue` 中，`setTimeout` 也放入`macro-task queue`。
但是，`promise.then` 放到了另一个任务队列 `micro-task queue`中。
这两个任务队列执行顺序如下，取1个`macro-task queue` 中的 `task`，执行之。
然后把所有 `micro-task queue` 顺序执行完，再取 `macro-task queue` 中的下一个任务。

## 解释
代码一开始执行时，所有这些代码在`macro-task queue` 中，取出来执行之。
后面遇到了 `setTimeout` ，又加入到`macro-task queue` 中，然后，遇到了 `Promise`新建后立即执行输出`2`, 然后，遇到了 `promise.then`，放入到了另一个队列`micro-task queue`，然后代码继续执行，输出 `4` 。
等整个 `execution context stack` 执行完后，下一步该取的是`micro-task queue` 中的任务了。
取出 `promise.then`，执行，输出`3`，`micro-task queue`为空，下一步取出 `marco-task queue` 中的 `setTimeout`，执行，输出 `1`
所以最终的结果是 `2 4 3 1`。



推荐一个视频[what-the-heck-is-the-event-loop-anyway](https://2014.jsconf.eu/speakers/philip-roberts-what-the-heck-is-the-event-loop-anyway.html)

里面讲的很详细，从同步讲到异步，还有图文讲解，英文的，要翻墙的。



>参考：
>[Promise的队列与setTimeout的队列有何关联？](https://www.zhihu.com/question/36972010)
>[promise和setTimeout执行顺序的疑惑](https://segmentfault.com/q/1010000009811652)
>[从Promise来看JavaScript中的Event Loop、Tasks和Microtasks](https://www.tuicool.com/articles/MnY7N3a)
