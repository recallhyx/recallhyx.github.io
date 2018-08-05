---
title: JavaScript一些思考
date: 2018-04-19 09:39:55
tags: javascript
categories: 前端
---
# 前言
最近走马观花的了解了很多东西，感觉web前端的趋势和发展很快，但是唯一不变的就是基础，所以需要把基础打好，把 JavaScript 学好 ，才能以不变应万变。
# 全局变量
我们都很熟悉全局变量了，看一道面试题
```
(function (){
  var a = b = 1;
})()
console.log(b);// 1
console.log(a);// Uncaught ReferenceError: a is not defined
```
实际上，JavaScript 是从右到左赋值的，相当于
```
(function (){
  var a = (b = 1);
})()
```
先给 `b` 赋值，然后再把 `b` 的值赋值给 `a`。`b` 没有用 `var` 声明就赋值，所以是一个全局变量，因此能够在函数体外访问。
所以如果是想用这种连续赋值的方式赋值又不想暴露全局变量的话，可以用以下的方式赋值：
```
(function (){
  var a,b;
  a = b = 1;
})()
console.log(b);// Uncaught ReferenceError: b is not defined
console.log(a);// Uncaught ReferenceError: a is not defined
```
其实全局变量分为两种，显式全局变量和隐式全局变量。
隐式全局变量和明确定义的全局变量间有些小的差异，判断方法就是能否通过 `delete` 操作符让变量未定义的能力。
+ 通过 `var` 创建的全局变量（任何函数之外的程序中创建）是不能被删除的。
+ 无 `var` 创建的隐式全局变量（无视是否在函数中创建）是能被删除的。（即上例的 `b`）

让我们看个例子：
```
// 定义三个全局变量
var global_var = 1;
global_novar = 2; // 反面教材
(function () {
   global_fromfunc = 3; // 反面教材
}());

// 试图删除
console.log(delete global_var); // false
console.log(delete global_novar); // true
console.log(delete global_fromfunc); // true

// 测试该删除
typeof global_var; // "number"
typeof global_novar; // "undefined"
typeof global_fromfunc; // "undefined"
```
这下我们就明白了，隐式全局变量并不是真正的全局变量，它们是全局对象的属性。属性是可以通过 `delete` 操作符删除的，而变量是不能的。所以隐式全局变量我们也可以通过全局对象来访问：
```
(function (){
  var a = b = 1;
})()
console.log(window.b);// 1
```
当然，这种隐式全局变量是不安全的，因为是全局变量的属性，所以很可能被其他变量所覆盖导致代码出错：
```
// 文件1定义了 id 函数
id = function(){
    console.log('id');
}
// 文件2定义了 id 变量并赋值
id = 1;
// 文件1 并不知道文件2定义了 id，并覆盖了id 函数，当执行的时候就会出错
id();// Uncaught TypeError: id is not a function
```
所以为了团队开发，不要使用隐式全局变量。
# for 循环
很多人需要遍历一个数组的时候都是这样子的：
```
for(var i = 0; i < arr.length; i++){
  
}
```
这样做每次循环都会获取数组的长度，这样会降低性能，特别是如果我们对 DOM 进行操作的时候：
```
for(var i = 0;i < document.getElementsByTagName('div').length; i++){

}
```
每次都要获取 `div` ，会极大的降低性能，所以我们一般都会用一个变量保存：
```
for(var i = 0, len = arr.length; i < len; i++){

}
```
# for-in 循环
for-in 循环应该用在非数组对象的遍历上，而且 for-in 循环要小心使用，因为它会遍历原型链上的属性，我们先看一个例子：
```
var div = document.getElementsByTagName('div')
var count = 0;
for(var value in div[0]){
    count++;
}
console.log(count)// 232 
```
结果不一定就是正好 `232` 但是可以从这个例子看出来，一个 DOM 结点是有很多属性的，那如果我们恰巧在一个对象的原型链上添加了一个 DOM 结点，没有过滤掉的话就会白白浪费性能又得不到我们想要的结果。
`Object.hasOwnProperty()` 能够帮我们过滤原型链上的属性：
```
// 对象
var man = {
   hands: 2,
   legs: 2,
   heads: 1
};

// 在代码的某个地方
// 一个方法添加给了所有对象
if (typeof Object.prototype.clone === "undefined") {
   Object.prototype.clone = function () {};
}

// 1.
// for-in 循环
for (var i in man) {
   if (man.hasOwnProperty(i)) { // 过滤
      console.log(i, ":", man[i]);
   }
}
/* 控制台显示结果
hands : 2
legs : 2
heads : 1
*/
// 2.
// 反面例子:
// for-in loop without checking hasOwnProperty()
for (var i in man) {
   console.log(i, ":", man[i]);
}
/*
控制台显示结果
hands : 2
legs : 2
heads : 1
clone: function()
*/
```
让我们看另外一个例子：
```
var arr = [1,2,3];
Array.prototype.a = 4;
for(var i in arr){
    console.log(arr[i]);
}
console.log('arr.length',arr.length);
// 1
// 2
// 3
// 4
// arr.length 3
```
突然出现了 `4` ，与我们实际的结果不一致，所以 for-in 不要用在数组上，还是用普通的 for 循环。

# 自定义 log
很多时候我们都要用 `console.log` 来 debug，但是一堆 `console.log` 也不知道对应哪个文件的输出，所以我们需要自己自定义 log，在 log 加上前缀。
```
function log() {
    var args = Array.from(arguments); // 用 ES6 的 Array.from 将 arguments 对象转为数组 
    args.unshift('[app.js]');       // 在数组头部添加前缀
    console.log.apply(console, args);     // 调用console.log()
}
log('hello','world')
```

# 个人不推荐用 && 和 || 代替 if
```
var foo = 10;  
foo === 10 && doSomething(); // 等效 if (foo == 10) doSomething(); 
foo === 5 || doSomething(); // 等效 if (foo != 5) doSomething();
```
确实有人这么写，说是代码简洁了，但是如果不懂短路原理的人看这个就会混乱，毕竟一个团队技术水平参差不齐，而且代码以后也可能交给别人维护，可读性要比较好。


>参考：
[深入理解JavaScript系列（1）：编写高质量JavaScript代码的基本要点](http://www.cnblogs.com/TomXu/archive/2011/12/28/2286877.html)
[JS里的进阶技巧](https://rainzhaojy.github.io/2015/js_tips.html)
