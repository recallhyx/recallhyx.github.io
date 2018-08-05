---
title: 理解nextTick
date: 2018-05-14 21:07:06
tags: node vue
categories: 前端
---
# 前言
一开始看到 `nextTick` 是在理解 JavaScript 事件循环的时候看到的，那个时候认为是 `node` 的东西，所以没怎么在意，但是后来看到 Vue 也有个地方用到了 `nextTick`，所以还是要花时间理解一下这个东西，看看它们的区别与联系。
# node 中的 process.nextTick()
在之前 [事件循环](https://recallhyx.github.io/2018/03/14/JavaScript%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF/) 这一篇博客中可以知道有两个东西，一个叫 `marcotask`，另外一个叫 `microtask`，而 `process.nextTick` 是属于 `microtask` 的，也就是说是和 `promise.then` 一样，会比 `setTimeout` 等 `marcotask` 事件更先执行，但是 `process.nextTick` 和 `promise.then` 谁更先执行呢，我们看以下代码：
```
console.log(1);
const promise = new Promise((resolve,reject) => {
    console.log(2);
    resolve();
})
promise.then(() => {
    console.log(3);
});
process.nextTick(() => console.log(4));
```
可以看到结果：
![](https://upload-images.jianshu.io/upload_images/5834506-3363ab10ad9df969.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
从这里我们就可以看出来了，`process.nextTick` 比 `promise.then` 更快执行。
其实，在 node，一次事件循环包括以下三个队列：
![](https://upload-images.jianshu.io/upload_images/5834506-4dfaa1faa61aaba7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，多了一个 `nextTickQueue` ，它在 `marcotask` 之后在 `microtask` 之前执行，`process.nextTick` 就是在这个 `nextTickQueue` 里面，所以它会比 `promise.then` 先执行。

这里提一下 node 的特有的函数 `setImmediate`，它和 `setTimeout` 还有 `setInterval` 都是会添加到次轮循环的 `marcotask`，但是 `process.nextTick` 是添加到本轮循环的 `nextTickQueue` 里面。

## 事件循环六个阶段
我们上图看到的三个队列，其实具体对应到事件循环六个阶段的几个阶段：
1. timers
2. I/O callbacks
3. idle, prepare
4. poll
5. check
6. close callbacks

每个阶段都有一个先进先出的回调函数队列。只有一个阶段的回调函数队列清空了，该执行的回调函数都执行了，事件循环才会进入下一个阶段。
![](https://upload-images.jianshu.io/upload_images/5834506-021889547efeb107.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（1）timers
这个是定时器阶段，处理 `setTimeout()` 和 `setInterval()` 的回调函数。进入这个阶段后，主线程会检查一下当前时间，是否满足定时器的条件。如果满足就执行回调函数，否则就离开这个阶段。
（2）I/O callbacks
除了以下操作的回调函数，其他的回调函数都在这个阶段执行。
>`setTimeout()` 和 `setInterval()` 的回调函数
`setImmediate()`的回调函数
用于关闭请求的回调函数，比如 `socket.on('close', ...)`

（3）idle, prepare
该阶段只供 `libuv` 内部调用，这里可以忽略。
（4）Poll
这个阶段是轮询时间，用于等待还未返回的 I/O 事件，比如服务器的回应、用户移动鼠标等等。
这个阶段的时间会比较长。如果没有其他异步任务要处理（比如到期的定时器），会一直停留在这个阶段，等待 I/O 请求返回结果。
（5）check
该阶段执行 `setImmediate()` 的回调函数。
（6）close callbacks
该阶段执行关闭请求的回调函数，比如 `socket.on('close', ...)` 。

需要注意的是，Node 开始执行脚本时，会先进行事件循环的初始化，但是这时事件循环还没有开始，会先完成下面的事情。而且 `process.nextTick` 会在当前阶段结束前执行，`microtask` 会在 `process.nextTick` 执行完后全部执行。即每个阶段都会先执行 `process.nextTick`，然后执行 `microtask`。
>同步任务
发出异步请求
规划定时器生效的时间
执行process.nextTick()等等

最后，上面这些事情都干完了，事件循环就正式开始了。
## 加深印象
让我们看一下一段代码：
```
setImmediate(function () {
    console.log(1);
    process.nextTick(function () {
        console.log(2);
    });
});
process.nextTick(function () {
    console.log(3);
    setImmediate(function () {
        console.log(4);
    })
});
// 3 1 4 2
```
让我们理一下过程：
首先遇到 `setImmediate` ，把它的回调函数放到下一轮事件循环的 `marcotask`。
然后遇到 `process.nextTick`，由于先进行事件循环的初始化，会执行 `process.nextTick` ，输出 `3` 并把 `setImmediate` 放到下一轮事件循环的 `marcotask` 队列。 
这样，开始第一轮事件循环，由于没有 `setTimeout` 或者 `setInterval` ，所以跳过。
进入下一个阶段 `I/O callbacks`，`nextTickQuque` 队列为空，继续下面的阶段直至 `check` 阶段，从 `marcotask` 队列拿出 `setImmediate` ，执行，输出 `1`，遇到 `process.nextTick` 放入`nextTickQuque` 队列。
`marcotask` 队列不为空，继续拿出队头，执行，输出 `4` ，`marcotask` 队列为空，在 `check` 阶段结束前，`nextTickQuque` 不为空，拿出队头，执行，输出 `2` 。
# Vue 中的 nextTick
Vue 中的 `nextTick` 是在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。
一个普遍的常识是 DOM Tree 的修改是实时的，而修改的 Render 到 DOM 上才是异步的。根本不存在什么所谓的等待 DOM 修改完成，任何时候我在上一行代码里往DOM中添加了一个元素、修改了一个 DOM 的 `textContent` ，你在下一行代码里一定能立马就读取到新的 DOM。
虽然我们能读取到新的 DOM，但是浏览器并未渲染出来，如果我们马上在 `document` 获取修改的 DOM的结点，会发现并不是刚刚修改的值。这就是为什么在 Vue 中，如果在 `created` 事件中操作元素，会报错说提示不存在，因为 Vue 在 `created` 的时候还没有渲染出 UI，所以将需要的操作用 `nextTick` 执行或者直接在 `mounted` 生命周期中操作元素，这个时候 UI 就已经渲染出来了。

`nextTick` 使用的是 `microtask` ，在 `marcotask` 执行完毕之后就会立即执行所有`microtask` ，那么 `flushBatcherQueue`（真正修改 DOM）便得以在此时立即完成，而后，当前轮次的 `microtask` 全部清理完成时，执行UI rendering，把重排重绘等操作真正更新到DOM上。
真正的去遍历 watcher，批处理更新是在 `microtask` 中执行的，而且用户在修改数据后自己执行的 `nextTick(cb)` 也会在此时执行 `cb`，他们都是在同一个 `microtask` 中执行。
之所以要这样，是因为用户的代码当中是可能多次修改数据的，而每次修改都会同步通知到所有订阅该数据的 watcher，而立马执行将数据写到 DOM 上是肯定不行的，那就只是把 watcher 加入数组。等到当前 `task` 执行完毕，所有的同步代码已经完成，那么这一轮次的数据修改就已经结束了，这个时候我可以安安心心的去将对监听到依赖变动的 watcher 完成数据真正写入到 DOM 上的操作，这样即使你在之前的 task 里改了一个 watcher 的依赖100次，我最终只会计算一次 value、改 DOM 一次。一方面省去了不必要的 DOM 修改，另一方面将DOM操作聚集，可以提升 DOM Render 效率。

# 区别与联系
在 node 里面，有一个专门的队列来存放 `nextTick` 中的回调函数，而 Vue 中的 `nextTick` 则是用 `microtask` 来实现，浏览器只有 `marcotask` 和 `microtask` 两种队列。 但是他们执行的时间是一样的，即是在 `marcotask` 执行完之后执行。

>参考：
[Node.js的event loop及timer/setImmediate/nextTick](https://github.com/creeperyang/blog/issues/26)
[Node 定时器详解](www.ruanyifeng.com/blog/2018/02/node-event-loop.html)
[Vue源码详解之nextTick](https://github.com/Ma63d/vue-analysis/issues/6)