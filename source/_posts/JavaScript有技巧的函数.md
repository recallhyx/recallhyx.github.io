---
title: JavaScript有技巧的函数
date: 2018-04-27 20:56:10
tags: javascript
categories: 前端
---
# 前言
JavaScript 有很多有技巧的函数，这篇文章是我自己的一个记录。

# 链式调用
看一个例子吧，我们要实现一个函数 `fun` ，它有以下功能：
```
fun(1).add(1).min(2).num // 0
```
这是一个很典型的例子，用到了链式调用，其核心就是返回自己，即 `return this`，那么我们来实现这个函数吧：
```
function fun(num){
    this.num = num;
    this.add = function(num){
        this.num+=num;
        return this; // 核心
    }
    this.min = function (num){
        this.num-=num;
        return this; // 核心
    }
}
```
然后我们要怎么用这个函数呢？如果我们直接链式调用：
```
fun(1).add(1).min(2);
// Uncaught TypeError: Cannot read property 'add' of undefined
```
只有对象才能链式调用，jquery的 `$` 其实是返回一个对象：
```
var obj = {
  a:{
    b:{
      fun :function(){console.log('b');}
    }
  }
}
obj.a.b.fun(); // b
```
既然如此，那么我们就 `new` 一下：
```
function fun(num){
    this.num = num;
    this.add = function(num){
        this.num+=num;
        return this; // 核心
    }
    this.min = function (num){
        this.num-=num;
        return this; // 核心
    }
}
var fun = new fun(1);
fun.add(1).min(2).num; // 0
```
这样和我们一开始 `fun(1).add(1).min(2).num` 的调用方式差了一点，既然是要用对象，那我们用一个函数包起来不就行了嘛：
```
function Fun(num){
    this.num = num;
    this.add = function(num){
        this.num+=num;
        return this; // 核心
    }
    this.min = function (num){
        this.num-=num;
        return this; // 核心
    }
}
function fun(num){
  return new Fun(num);
}
fun(1).add(1).min(2).num; // 0
```
这就是完整版的代码了。

# compose
还是来看一道题 ：
```
实现一个 compose 函数，它接受任意多个函数作为参数（这些函数都只接受一个参数），然后 compose 返回的也是一个函数，达到以下的效果：
const operate = compose(div2, mul3, add1, add1)
operate(0) // => 相当于 div2(mul3(add1(add1(0))))
operate(2) // => 相当于 div2(mul3(add1(add1(2))))
```
我们直接看怎么实现的：
```
const compose = (...fns) => fns.reduce((f, g) => (...args) => f(g(...args)));
var add1 = x => x+1;
var mul3  = x=> x*3;
var div2 = x=> x/2;
var operate = compose(div2,mul3,add1,add1);
operate(0); // 3
operate(2); // 6
```
使用 `Array.reduce()` 和 ES6 的 `rest`参数，执行从右向左的函数组合。最后一个 (最右边) 的函数可以接受一个或多个参数，其余的函数必须是一元的。
如果觉得很难看懂的话，我们把箭头函数改成 `function`：
```
var compose2 = function (...fns){
    return fns.reduce(function (f,g){
        return function(...args){
            return f(g(...args));
        }
    })
}
var add1 = x => x+1;
var mul3  = x=> x*3;
var div2 = x=> x/2;
var operate = compose(div2,mul3,add1,add1);
operate(0); // 3
```