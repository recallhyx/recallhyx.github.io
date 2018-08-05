---
title: JavaScript关于判断的一些奇怪结果
date: 2018-03-30 18:36:37
tags: javascript
categories: 前端
---
# 前言
JavaScript 的 `==` 和 `===` 两种判断相等的方法行为不一致，很让人头晕，而且还有 `typeof` 和 `instanceof`所以记录一下，解决这个问题
# == 、 === 和 Object.is()
## ==
双等号 `==` ，也叫 宽松相等，相等操作符比较两个值是否相等，在比较前将两个被比较的值转换为相同类型。在转换后（等式的一边或两边都可能被转换），最终的比较方式等同于全等操作符 `===` 的比较方式。 相等操作符满足交换律。
### 经典例子
```
var num = 0;
var obj = new String('0');
var str = "0";
var b = false;

console.log(num == obj); // true
console.log(num == str); // true
console.log(obj == str); // true
console.log(null == undefined); // true

console.log(0 == false);// true
console.log('' == false);// true
console.log('' == 0);// true

console.log(null == 0);// false;
console.log(undefined == 0);// false;

console.log(1 == true);// true
console.log(2 == true);// false
```
### 内部原理
标准的相等性操作符（== 与 !=）使用了[Abstract Equality Comparison Algorithm](https://link.zhihu.com/?target=http%3A//www.ecma-international.org/ecma-262/5.1/%23sec-11.9.3)来比较操作符两侧的操作对象（x == y），该算法流程要点提取如下：

*   如果 x 或 y 中有一个为 `NaN`，则返回 `false`；
*   如果 x 与 y 皆为 `null` 或 `undefined` 中的一种类型，则返回 `true`（`null == undefined // true`）；否则返回 `false`（`null == 0 // false`）；
*   如果 x,y 类型不一致，且 x,y 为 `String`、`Number`、`Boolean` 中的某一类型，则将 x,y 使用 `toNumber` 函数转化为 `Number` 类型再进行比较；
*   如果 x，y 中有一个为 `Object`，则首先使用 `ToPrimitive` 函数将其转化为原始类型，再进行比较。

```
ToPrimitive(obj,preferredType)

JS引擎内部转换为原始值ToPrimitive(obj,preferredType)函数接受两个参数，
第一个obj为被转换的对象，
第二个preferredType为希望转换成的类型（默认为空，接受的值为Number或String）
在执行ToPrimitive(obj,preferredType)时如果第二个参数为空并且obj为Date的事例时，
此时preferredType会被设置为String，其他情况下preferredType都会被设置为Number如果preferredType为Number，
ToPrimitive执行过程如下：

1. 如果obj为原始值，直接返回；
2. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
3. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
4. 否则抛异常。

如果preferredType为String，将上面的第2步和第3步调换，即：
1. 如果obj为原始值，直接返回；
2. 否则调用 obj.toString()，如果执行结果是原始值，返回之；
3. 否则调用 obj.valueOf()，如果执行结果是原始值，返回之；
4. 否则抛异常。
```
还有一种情况：
```
console.log([]==![]);// true
```
具体涉及到了运算符优先级，可以看看[这篇文章](https://github.com/jawil/blog/issues/1)，讲的非常详细
## ===
三等号 `===`，叫 严格相等比较，三等号将进行相同的比较，而不进行类型转换 (如果类型不同, 只是总会返回 `false` )
### 经典例子
用 `===` 就不会出现 `==` 的一些奇怪的结果
```
var num = 0;
var obj = new String("0");
var str = "0";
var b = false;

console.log(num === num); // true
console.log(obj === obj); // true
console.log(str === str); // true

console.log(num === obj); // false
console.log(num === str); // false
console.log(obj === str); // false
console.log(null === undefined); // false
console.log(obj === null); // false
console.log(obj === undefined); // false
```
在日常中使用全等操作符几乎总是正确的选择，但还是有几个特殊的例子
```
console.log(NaN === NaN);// false
console.log(Symbol(1) === Symbol(1));// false
console.log({x:1}==={x:1});// false
console.log([]===[]);// false
```
对于 `NaN` ，是 IEEE 754 规范上写的，而 `Symbol` 是 ES6 新增的原始数据类型，代表的意思就是独一无二的，所以当然不会相等，`{x:1}` 是一个对象，即使再声明一个一样的对象，它的内存地址不一样，也当然不一样了，`[]` 也是同样的道理，内存地址不同运用在 `==` 也是一样的。

## Object.is()
由于 `===` 也会出现一些奇怪的判断，所以在 ES6 加入了 `Object.is()` 对于 `NaN`和 `-0` 和 `+0` 进行特殊处理。
```
console.log(Object.is(NaN,NaN));// true
```
## 全部比较
![](https://upload-images.jianshu.io/upload_images/5834506-7bf7a148c550b241.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# typeof 和 instanceof
另外两个比较混乱的判断就是 `typeof` 和 `instanceof`
## typeof
`typeof` 操作符返回一个字符串，表示未经计算的操作数的类型。
### 经典例子
```
// Numbers
typeof 37 === 'number';
typeof 3.14 === 'number';
typeof Math.LN2 === 'number';
typeof Infinity === 'number';
typeof NaN === 'number'; // 尽管NaN是"Not-A-Number"的缩写
typeof Number(1) === 'number'; // 但不要使用这种形式!

// Strings
typeof "" === 'string';
typeof "bla" === 'string';
typeof (typeof 1) === 'string'; // typeof总是返回一个字符串
typeof String("abc") === 'string'; // 但不要使用这种形式!

// Booleans
typeof true === 'boolean';
typeof false === 'boolean';
typeof Boolean(true) === 'boolean'; // 但不要使用这种形式!

// Symbols
typeof Symbol() === 'symbol';
typeof Symbol('foo') === 'symbol';
typeof Symbol.iterator === 'symbol';

// Undefined
typeof undefined === 'undefined';
typeof declaredButUndefinedVariable === 'undefined';
typeof undeclaredVariable === 'undefined'; 

// Objects
typeof {a:1} === 'object';

// 使用Array.isArray 或者 Object.prototype.toString.call
// 区分数组,普通对象
typeof [1, 2, 4] === 'object';

typeof new Date() === 'object';

// 下面的容易令人迷惑，不要使用！
typeof new Boolean(true) === 'object';
typeof new Number(1) === 'object';
typeof new String("abc") === 'object';

// 函数
typeof function(){} === 'function';
typeof class C{} === 'function'
typeof Math.sin === 'function';
typeof new Function() === 'function';
```
### typeof null
还有一个特别的例子：
```
typeof null === 'object';
```
官方解释是：
>在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 0。由于 null 代表的是空指针（大多数平台下值为 0x00），因此，null的类型标签也成为了 0，typeof null就错误的返回了"object"。

### new 操作符
```
// 所有的构造函数在实例化一个对象的时候 typeof 都会返回 'object'
var str = new String('String');
var num = new Number(100);

typeof str; //  'object'
typeof num; // 'object'

// 只有一个例外：构造函数

var func = new Function();

typeof func; //  'function'
```
## instanceof
有时候我们需要判断一个对象的类型，如果我们用 `typeof` 只会得到 `"object"`：
```
var arr = [1,2,3,4]
typeof arr;// "object"

var date = new Date();
typeof date;// "object"
```
这个时候就要用 `instanceof` 。
`instanceof` 运算符用来测试一个对象在其原型链中是否存在一个构造函数的 `prototype` 属性。
```
var arr = [1,2,3,4]
arr instanceof Array// true

var date = new Date();
date instanceof Date// true
```
但这并不是 `instanceof` 原来的用法，`instanceof` 运算符用来检测 `constructor.prototype` 是否存在于参数 `object` 的原型链上。
```
// 定义构造函数
function C(){} 
function D(){} 

var o = new C();


o instanceof C; // true，因为 Object.getPrototypeOf(o) === C.prototype


o instanceof D; // false，因为 D.prototype不在o的原型链上

o instanceof Object; // true,因为Object.prototype.isPrototypeOf(o)返回true
C.prototype instanceof Object // true,同上

C.prototype = {};
var o2 = new C();

o2 instanceof C; // true

o instanceof C; // false,C.prototype指向了一个空对象,这个空对象不在o的原型链上.

D.prototype = new C(); // 继承
var o3 = new D();
o3 instanceof D; // true
o3 instanceof C; // true
```
需要注意的是，如果表达式 `obj instanceof Foo` 返回 `true`，则并不意味着该表达式会永远返回 `true`，因为 `Foo.prototype` 属性的值有可能会改变，改变之后的值很有可能不存在于 `obj` 的原型链上，这时原表达式的值就会成为 `false` 。另外一种情况下，原表达式的值也会改变，就是改变对象 `obj` 的原型链的情况，虽然在目前的 ES 规范中，我们只能读取对象的原型而不能改变它，但借助于非标准的`__proto__`魔法属性，是可以实现的。比如执行 `obj.__proto__ = {}` 之后，`obj instanceof Foo` 就会返回 `false` 了。
### 神奇的例子
```
var simpleStr = "This is a simple string"; 
var myString  = new String();
var newStr    = new String("String created with constructor");
var myDate    = new Date();
var myObj     = {};

simpleStr instanceof String; // returns false, 检查原型链会找到 undefined
myString  instanceof String; // returns true
newStr    instanceof String; // returns true
myString  instanceof Object; // returns true

myObj instanceof Object;    // returns true, despite an undefined prototype
({})  instanceof Object;    // returns true, 同上

myString instanceof Date;   // returns false

myDate instanceof Date;     // returns true
myDate instanceof Object;   // returns true
myDate instanceof String;   // returns false
```
所以我们判断是否是数组还是用这种方法`Object.prototype.toString.call(myObj) === "[object Array]"` 或者 ES6 新增的 `Array.isArray(myObj)`。

>参考
[MDN-typeof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)
[MDN-instanceof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)
[JavaScript 中的相等性判断](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness)