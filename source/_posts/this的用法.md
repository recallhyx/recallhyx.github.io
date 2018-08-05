---
title: JavaScript this的用法
date: 2018-02-14 21:25:45
categories: 前端
tags: javascript
---
## this介绍

`this` 是 JavaScript 的关键字之一，用法灵活，也是很多 web 前端面试题会出的，不过总结起来就一句话：__this指的是，调用函数的那个对象__

## this 取值的四种情况
### 构造函数的调用
所谓构造函数，就是通过这个函数生成一个新对象（`object`）。这时，`this`就指这个新对象。
```
var name = "hyx";
function Name(){
    this.name = "Recall";
}
var person = new Name();
person.name;// "Recall"
```
可以看到，`this` 指向新对象 `person` 而不是全局变量 `window` 这与我们的总结是一致的
如果不是使用 `new Name()`的方法而是直接调用，即 `Name()`，结果会不一样，因为这不是构造函数调用

### 普通函数的调用
这是函数的最通常用法，属于全局性调用，因此`this`就代表全局对象 `window`。
```
var name = "hyx";
function Name(){
    this.name = "Recall";
}
Name();
console.log(name); // Recall
```
上面就是直接调用，可以看到全局变量 `name` 被改变了（从`hyx`->`Recall`），说明`this`指向的是全局变量 `window·`

### call 和 apply 的调用
当一个函数被 `call` 和 `apply` 调用时，`this` 的值就取传入的对象的值。`call` 和 `apply` 的用法类似，只不过传入参数不一样。
`fun.call(thisObj[, arg1[, arg2[, ...]]])`
`call() `可以接收任意个数的参数，其中第一个必须是一个 `this` 对象，其余依次是所有的参数。如果没有提供 `thisObj` 参数，那么全局对象被用作 `thisObj`。

`fun.apply(thisObj, [argsArray])`
`apply()` 方法接受两个参数第一个是函数运行的作用域，另外一个是一个参数数组(arguments)。
如果没有提供 `thisObj` 和 `argsArray` 任何一个参数，那么全局对象将被用作 `thisObj`， 并且无法被传递任何参数。
```
　　var x = 10;
　　function test(){
　　　　alert(this.x);
　　}
　　var o={};
　　o.x = 1;
　　o.m = test;
　　o.m.apply(); //10
```
这段代码，对象 `o` 有两个属性，`x` 和 `m`，其中，`x` 的值为 `1`，`m` 的值为函数 `test`，`o.m.apply()` 不传任何参数，则默认`this` 为全局对象 `window`，所以 `x` 的值就是代码第一行声明赋值的 `10`
如果把最后一行代码修改为
```
o.m.apply(o); //1
```
运行结果就变成了`1`，证明了这时 `this` 代表的是对象 `o`。
### 作为对象方法的调用
这才是重中之重，如果函数作为对象的一个属性时，并且作为对象的一个属性被调用时，函数中的 `this` 指向该对象。我们先看个例子
```
var name = 'hyx';
var person = {
  name : 'Recall',
  fn : function(){
    console.log(this);
    console.log(this.name);
  }
}
person.fn()
//{name: "Recall", fn: ƒ}
//Recall
```
上述代码中，是 `person` 调用的 `fn` 方法，所以 `this` 指向 `person`
注意：以下两种情况，`this` 指向全局对象 `window`
第一种，函数不作为对象的属性而被调用
```
var name = 'hyx';
var person = {
  name : 'Recall',
  fn : function(){
    console.log(this);
    console.log(this.name);
  }
}
var fn = person.fn;
fn();
// window {...}
// hyx
```
第二种，在一个对象的函数内再声明的这个函数
```
var name = 'hyx';
var person = {
  name : 'Recall',
  fn : function(){
     function fn2(){
         console.log(this);
         console.log(this.name);
    }
    fn2();
  }
}
person.fn();
// window {...}
// hyx
```
正常来说，按照我们的总结，即 __this指的是，调用函数的那个对象__，来说，`this.name` 应该输出 `Recall` ，但却输出的是 `hyx` ，而且我们可以看到 `this` 的值是全局对象 `window`。这是语言设计上的一个错误。倘若语言设计正确，那么当内部函数被调用时，`this` 应该仍然绑定到外部函数的 `this` 变量。
这个设计错误的后果是方法不能利用内部函数来帮助它工作。
ECMAScript6 的箭头函数（注意只是箭头函数）基本纠正了这个设计上的错误（注意只是基本上，但不是彻底地纠正了错误）
```
var name = 'hyx';
var person = {
  name : 'Recall',
  fn : function(){
      fn2 = ()=>{
         console.log(this);
         console.log(this.name);
    }
    fn2();
  }
}
person.fn();
// {name: "Recall", fn: ƒ}
// hyx
```
我们可以看一下箭头函数转换成 ES5 是怎样做的
```
var name = 'hyx';
var person = {
  name : 'Recall',
  fn : function(){
      var _this = this;
      fn2 = function(){
         console.log(_this);
         console.log(_this.name);
    }
    fn2();
  }
}
person.fn();
// {name: "Recall", fn: ƒ}
// Recall
```
### 总结
`this` 的用法是我刷面试题的时候遇到的，一开始挺头疼的，不知道什么时候指向全局对象，什么时候指向调用对象，加上 JavaScript 设计上的缺陷就更头晕了，所以花了一个下午专门做了笔记，希望能有效果。
__注意：全局对象，在浏览器是 window ，在 node 是 Global __


>参考
[深入理解javascript原型和闭包（10）——this](http://www.cnblogs.com/wangfupeng1988/p/3988422.html)
[JavaScript this 总结（含 ES6）](http://www.cnblogs.com/libin-1/p/5814792.html)
[Javascript的this用法](http://www.ruanyifeng.com/blog/2010/04/using_this_keyword_in_javascript.html)
