---
title: 从Webpack的Output看模块规范
date: 2018-06-13 21:55:27
tags: webpack
categories: 前端
---

# libraryTarget
从上一篇[Vue组件开发流程](https://recallhyx.github.io/2018/06/12/Vue%E7%BB%84%E4%BB%B6%E5%BC%80%E5%8F%91%E6%B5%81%E7%A8%8B/)中，我们可以看到 webpack 里面的一个设置 `output` ：
```
  output: {
    path: config.build.assetsRoot,
    filename: 'vue-crop-avatar.min.js', // 输出的名字
    publicPath: '/dist/', // 输出的目录
    library: 'vue-crop-avatar', // 引入组件时的名字
    libraryTarget: 'umd',   // 输出格式
    umdNamedDefine: true    // 是否将模块名称作为 AMD 输出的命名空间
  },
```
其中，`libraryTarget` 可选的值有以下几个：
- `var`
- ` this`
- `commonjs`
- `commonjs2`
- `amd`
- `umd`

`libraryTarget` 就是选择怎么暴露 `library` ，即打包后的代码以什么格式输出

# var-暴露为一个变量
`var` 是 `libraryTarget` 的默认值，如果要用 `var` ，就一定要设置 `library` 的值，因为打包后会是这种形式：
```
var MyLibrary = _entry_return_;
```
其中，`MyLibrary` 就是 `library` 的值，如果没有这个值，就会出错。`_entry_return_` 是由 webpack 生成的，是一个函数。
使用方法：
```
// 另外的脚本里
MyLibrary.doSomething()
```
# this-通过在对象上赋值暴露
如果要使用 `this`，推荐设置 `library` 的值，因为打包后是这种形式：
```
this["MyLibrary"] = _entry_return_;
```
如果没有 `library` 那么所有 `_entry_return_` 的东西都会赋值给 `this`
使用方法：
```
// 另外的脚本里
this.MyLibrary.doSomething();
MyLibrary.doSomething(); // 如果 this 是 window
```
# 模块定义系统
## commonjs
`libraryTarget: "commonjs"` 入口起点的返回值将使用 `output.library` 中定义的值，分配给 exports 对象。这个名称也意味着，模块用于 CommonJS 环境，打包后的形式：
```
exports["MyLibrary"] = _entry_return_;
```
使用方法：
```
// 另外的脚本里
require("MyLibrary").doSomething();
```
## commonjs2
`libraryTarget: "commonjs2"`  入口起点的返回值将分配给 `module.exports` 对象。这个名称也意味着模块用于 CommonJS 环境，打包后的形式：
```
module.exports = _entry_return_;
```
使用方法：
```
// 另外的脚本里
require("MyLibrary").doSomething();
```
## commonjs 和 commonjs2
两者在 webpack 里面没有区别，用还是那么用，从定义上讲，Commonjs 规范仅仅定义了 `exports` ，`module.exports` 是 `node` 对 Commonjs 规范的具体实现 。
## amd
`libraryTarget: "amd"`  将你的 library 暴露为 AMD 模块。
生成的 output 将会使用 "MyLibrary" 作为模块名定义，即：
```
define("MyLibrary", [], function() {
  return _entry_return_; // 此模块返回值，是入口 chunk 返回的值
});
```
使用方法：
可以在 `script` 标签中引入脚本，也需要 `requirejs` 
```
require(['MyLibrary'], function(MyLibrary) {
  // 使用 library 做一些事……
});
```
## amd 和 commonjs
AMD 和 CommonJS 都是模块规范，但是这两种是竞争关系。CommonJS 规范加载模块是同步的，也就是说，只有加载完成，才能执行后面的操作。AMD规范则是非同步加载模块，允许指定回调函数。由于 Node 主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以在服务端 CommonJS 规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用 AMD 规范。 
## umd

`libraryTarget: "umd"`  将你的 library 暴露为所有的模块定义下都可运行的方式。它将在 CommonJS, AMD 环境下运行，或将模块导出到 global 下的变量。打包后的形式：
```
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory();
  else if(typeof define === 'function' && define.amd)
    define([], factory);
  else if(typeof exports === 'object')
    exports["MyLibrary"] = factory();
  else
    root["MyLibrary"] = factory();
})(typeof self !== 'undefined' ? self : this, function() {
  return _entry_return_; // 此模块返回值，是入口 chunk 返回的值
});
```
这里的 `root` ，就是 `library` 的值，不设置的话，也会像不设置 `this` 一样，暴露到全局。
既然 CommonJs 和 AMD 风格一样流行，似乎缺少一个统一的规范。所以人们产生了这样的需求，希望有支持两种风格的“通用”模式，于是通用模块规范（UMD）诞生了。
不得不承认，这个模式略难看，但是它兼容了 AMD 和 CommonJS ，同时还支持老式的“全局”变量规范。
# 其他规范
## ES6 Module 和 CommonJS
ES6 Module 与 CommonJS 模块完全不同。它们有两个重大差异。

- CommonJS 模块输出的是一个值的拷贝，ES6 Module 输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 Module 是编译时输出接口。

CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。 
```
    // lib.js
    var counter = 3;
    function incCounter() {
      counter++;
    }
    module.exports = {
      counter: counter,
      incCounter: incCounter,
    };
```
上面代码输出内部变量 `counter` 和改写这个变量的内部方法 `incCounter`。然后，在 `main.js` 里面加载这个模块。 
```
    var mod = require('./lib');
    
    console.log(mod.counter);  // 3
    mod.incCounter();
    console.log(mod.counter); // 3
```
上面代码说明，lib.js 模块加载以后，它的内部变化就影响不到输出的 `mod.counter` 了。这是因为`mod.counter` 是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。 
```
    // lib.js
    var counter = 3;
    function incCounter() {
      counter++;
    }
    module.exports = {
      get counter() {
        return counter
      },
      incCounter: incCounter,
    };
```
上面代码中，输出的 `counter` 属性实际上是一个取值器函数。现在再执行 `main.js` ，就可以正确读取内部变量 counter 的变动了。 
```
    $ node main.js
    3
    4
```
ES6 Module 的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令 import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。换句话说，ES6 的 `import` 有点像 Unix 系统的“符号连接”，原始值变了， `import` 加载的值也会跟着变。因此，ES6 Module 是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。 
```
    // lib.js
    export let counter = 3;
    export function incCounter() {
      counter++;
    }
    
    // main.js
    import { counter, incCounter } from './lib';
    console.log(counter); // 3
    incCounter();
    console.log(counter); // 4
    
```
上面代码说明，ES6 Module 输入的变量 `counter` 是活的，完全反应其所在模块 `lib.js` 内部的变化。 
ES6 Module 不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。
由于 ES6 输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错。

但是，commonjs 输出的是一个值的拷贝，仅仅在这个值是原始类型的值，即 `Boolean`，`Number`，`String`，`Undefined`，`Null`，`Symbol` 这 6 中原始类型

值为 `Boolean`：

```
// lib.js
var counter = true;
function change() {
  counter = false;
}
module.exports = {
  counter: counter,
  change: change,
};
```

```
//main.js
var mod = require('./lib');

console.log(mod.counter);  // true
mod.change();
console.log(mod.counter); // true
```

值为 `String`：

```
// lib.js
var counter = 'hello';
function change() {
  counter = 'world';
}
module.exports = {
  counter: counter,
  change: change,
};
```

```
var mod = require('./lib');

console.log(mod.counter);  // hello
mod.change();
console.log(mod.counter);  // hello
```

值为 `undefined`：

```
// lib.js
var counter = undefined;
function change() {
  counter = 1;
}
module.exports = {
  counter: counter,
  change: change,
};
```

```
// main.js
var mod = require('./lib');

console.log(mod.counter); // undefined
mod.change();
console.log(mod.counter); // undefined
```

值为 `Symbol` ：

```
// lib.js
var counter = Symbol('foo');
function change() {
  counter = Symbol('bar');
}
module.exports = {
  counter: counter,
  change: change,
};
```

```
// main.js
var mod = require('./lib');

console.log(mod.counter);  // Symbol(foo)
mod.change();
console.log(mod.counter);  // Symbol(bar)
```



其他的就不做演示了，如果这个值是 `Object` 类型，那结果就会变：

```
// lib.js
var counter = {};
function change() {
  counter.a = 1;
}
module.exports = {
  counter: counter,
  change: change,
};
```

```
// main.js
var mod = require('./lib');

console.log(mod.counter);  // {}
mod.change();
console.log(mod.counter);  // {a:1}

```



>参考：
>[Webpack 中文指南](http://zhaoda.net/webpack-handbook/commonjs.html)
>[Module 的加载实现](http://es6.ruanyifeng.com/#docs/module-loader)
>[[译]神马是AMD, CommonJS, UMD?](https://75team.com/post/%E8%AF%91%E7%A5%9E%E9%A9%AC%E6%98%AFamd-commonjs-umd.html)
