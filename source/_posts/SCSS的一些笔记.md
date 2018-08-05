---
title: SCSS的一些笔记
date: 2018-03-16 14:34:56
tags: css
categories: 前端
---

## 安装
### 基于webpack
如果是在 vue-cli 项目中，可以使用以下命令安装
```
npm install node-sass --save-dev
npm install sass-loader --save-dev
```
vue-cli生成的项目，已经默认加入了处理sass的loader
只需要在需要的地方加入`lang=scss`即可
```
<style lang='scss' scope>
...
</style>
```
### 不是基于webpack
按照(官网教程)[https://www.sass.hk/install/]，手动把 `scss` 编译成 `css` 文件，然后自己引入。
## SCSS 介绍
Sass 有两种语法规则(`syntaxes`),目前新的语法规则（从 Sass 3开始）被称为 “SCSS”( 时髦的css（Sassy CSS）),它是 css3 语法的的拓展级，就是说每一个语法正确的 CSS3 文件也是合法的 SCSS 文件，SCSS 文件使用 `.scss` 作为拓展名。第二种语法别成为缩进语法（或者 Sass），它受到了 Haml 的简洁精炼的启发，它是为了人们可以用和 css 相近的但是更精简的方式来书写 css 而诞生的。它没有括号，分号，它使用 行缩进 的方式来指定 css 块，虽然 sass 不是最原始的语法，但是缩进语法将继续被支持，在缩进语法的文件以 `.sass` 为拓展名。
所以 SCSS 是 Sass 的其中一种语法规则而已，即 SCSS 是 Sass 的子集
## SCSS 特性
SCSS 加入了很多特性，例如，变量，嵌套，类似函数，继承，混合器，一切都是为了在写 css 的时候，不大量重复代码，注意，这只是在写的时候省去了重复的工作，滥用 SCSS ，会导致编译后的 css 文件过大，降低网站加载速度。下面介绍一下 SCSS 的特性。

### 变量
声明一个变量：`$div-color:#F90` 以`$`符开头，后面是变量名，之后跟一个 `:` ，然后赋值，我们甚至可以用空格隔开多个值，就像我们原本写 css 那样：`$div-border:1px solid black`。然后我们就可以使用这个变量了
```
$div-color:#F90
div {
  color: $div-color;
}
```
与 CSS 属性不同，变量可以在 css 规则块定义之外存在。当变量定义在 css 规则块内，那么该变量只能在此规则块内使用。
```
$nav-color: #F90;
nav {
  $width: 100px;
  width: $width;
  color: $nav-color;
}

//编译后
nav {
  width: 100px;
  color: #F90;
}
```
在这段代码中，`$nav-color` 这个变量定义在了规则块外边，所以在这个样式表中都可以像 `nav` 规则块那样引用它。`$width` 这个变量定义在了 `nav` 的 `{ }` 规则块内，所以它只能在 `nav` 规则块 内使用。这意味着是你可以在样式表的其他地方定义和使用 `$width` 变量，不会对这里造成影响。
我们完全可以嵌套定义一个变量：
```
$highlight-color: #F90;
$highlight-border: 1px solid $highlight-color;
.selected {
  border: $highlight-border;
}

//编译后
.selected {
  border: 1px solid #F90;
}
```
这里，`$highlight-border` 变量的声明中使用了 `$highlight-color` 这个变量。产生的效 果就跟你直接为 `border` 属性设置了一个 `1px $highlight-color solid` 的值是一样的。 
对于变量名，推荐用 `-` 连接符而不是 `_`，虽然这两个东西 SCSS 都会认为是同一个东西：
```
$link-color: blue;
a {
  color: $link_color;
}

//编译后
a {
  color: blue;
}
```
但是用 `-` 连接符连接更接近我们原本的 CSS 写法。
### 嵌套
有了嵌套，我们可以不用重复大段代码，而且可读性更高了：
```
#content {
  article {
    h1 { color: #333 }
    p { margin-bottom: 1.4em }
  }
  aside { background-color: #EEE }
}
 /* 编译后 */
#content article h1 { color: #333 }
#content article p { margin-bottom: 1.4em }
#content aside { background-color: #EEE }
```
SCSS 会一层一层解开嵌套，首先，把 `#content`（父级）这个 `id` 放到 `article` 选择器（子级）和 `aside` 选择器（子级）的前边：
```
#content {
  article {
    h1 { color: #333 }
    p { margin-bottom: 1.4em }
  }
  #content aside { background-color: #EEE }
}
```
然后，`#content article` 里边还有嵌套的规则，SCSS 重复一遍上边的步骤，把新的选择器添加到内嵌的选择器前边。
```
 /* 编译后 */
#content article h1 { color: #333 }
#content article p { margin-bottom: 1.4em }
#content aside { background-color: #EEE }
```
我们可以嵌套任意的 CSS，但是当我们想用伪类例如`:hover`时，就会出现一种情况
```
div a {
  color: blue;
  :hover { color: red }
}
```
这种嵌套，让 SCSS 解析后，会得到这样的结果：`color: red` 这条规则将会被应用到选择器 `div a :hover`，`div` 元素内链接的所有子元素在被 `hover` 时都会变成红色。
![鼠标悬浮在span标签](https://upload-images.jianshu.io/upload_images/5834506-e887e3e9ca227aa9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![鼠标悬浮在a标签](https://upload-images.jianshu.io/upload_images/5834506-9d0c210043e1aaaf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这不是我们想要的结果，怎么办呢？解决之道为使用一个特殊的 SCSS 选择器，即父选择器。在使用嵌套规则时，父选择器能对于嵌套规则如何解开提供更好的控制。它就是一个简单的 `&` 符号，且可以放在任何一个选择器可出现的地方。
```
div a {
  color: blue;
  &:hover { color: red }
}
```
当包含父选择器标识符的嵌套规则被打开时，它不会像后代选择器那样进行拼接，而是 `&` 被父选择器直接替换：
```
div a { color: blue }
div a:hover { color: red }
```
![](https://upload-images.jianshu.io/upload_images/5834506-919d412a5ab58a0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样，整个链接都会匹配到规则。
父选择器标识符还有另外一种用法，你可以在父选择器之前添加选择器。举例来说，当用户在使用IE浏览器时，你会通过JavaScript在 `<body>` 标签上添加一个`ie`的类名，为这种情况编写特殊的样式如下：
```
#content aside {
  color: red;
  body.ie & { color: green }
}
/*编译后*/
#content aside {color: red};
body.ie #content aside { color: green }
```

当我们使用群组选择器的时候，用 css 语法，重复的代码就会变多
```
.container h1, .container h2, .container h3 { margin-bottom: .8em }
```
这时我们可以用 SCSS 的嵌套：
```
.container {
  h1, h2, h3 {margin-bottom: .8em}
}
```

当我们使用子组合选择器和同层组合选择器：`>` 、`+` 和 `~`的时候，也可以用嵌套的方式：
```
article {
  ~ article { border-top: 1px dashed #ccc }
  > section { background: #eee }
  dl > {
    dt { color: #333 }
    dd { color: #555 }
  }
  nav + & { margin-top: 0 }
}
/*编译后*/
article ~ article { border-top: 1px dashed #ccc }
article > footer { background: #eee }
article dl > dt { color: #333 }
article dl > dd { color: #555 }
nav + article { margin-top: 0 }
```
在 SCSS 中，除了 CSS 选择器，属性也可以进行嵌套。尽管编写属性涉及的重复不像编写选择器那么糟糕，但是要反复写`border-style` `border-width` `border-color` 以及`border-*` 等也是非常烦人的。在 SCSS 中，你只需敲写一遍`border`：
```
nav {
  border: {
  style: solid;
  width: 1px;
  color: #ccc;
  }
}
```
嵌套属性的规则是这样的：把属性名从中划线-的地方断开，在根属性后边添加一个冒号`:`，紧跟一个 `{ }` 块，把子属性部分写在这个 `{ }` 块中。就像 css 选择器嵌套一样，SCSS 会把你的子属性一一解开，把根属性和子属性部分通过中划线 `-` 连接起来，最后生成的效果与你手动一遍遍写的css样式一样：
```
nav {
  border-style: solid;
  border-width: 1px;
  border-color: #ccc;
}
```
### 混合器
如果你的整个网站中有几处小小的样式类似（例如一致的颜色和字体），那么使用变量来统一处理这种情况是非常不错的选择。但是当你的样式变得越来越复杂，你需要大段大段的重用样式的代码，独立的变量就没办法应付这种情况了。你可以通过 SCSS 的混合器实现大段样式的重用。
混合器使用 `@mixin` 标识符定义。看上去很像其他的CSS `@`标识符，比如说`@media` 或者 `@font-face` 。这个标识符给一大段样式赋予一个名字，这样你就可以轻易地通过引用这个名字重用这段样式。下边的这段 SCSS 代码，定义了一个非常简单的混合器，目的是添加跨浏览器的圆角边框。
```
@mixin rounded-corners {
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}
```
然后就可以在你的样式表中通过 `@include` 来使用这个混合器，放在你希望的任何地方。`@include` 调用会把混合器中的所有样式提取出来放在 `@include` 被调用的地方。如果像下边这样写：
```
notice {
  background-color: green;
  border: 2px solid #00aa00;
  @include rounded-corners;
}

//sass最终生成：
.notice {
  background-color: green;
  border: 2px solid #00aa00;
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}
```
在`.notice` 中的属性 `border-radius` `-moz-border-radius` 和` -webkit-border-radius`全部来自 `rounded-corners` 这个混合器。
正如我们一开始所说的，使用 SCSS 虽然会减少我们写代码的重复性工作，但是滥用 SCSS 会造成编译后的 CSS 文件过大，混合器就是一个很好的例子，它只是将代码引入，如果我们在哪里都用混合器，看起来我们写的文件小，其实编译后的文件比我们想象的还要大，这个时候我们就要知道何时使用混合器。
>判断一组属性是否应该组合成一个混合器，一条经验法则就是你能否为这个混合器想出一个好的名字。如果你能找到一个很好的短名字来描述这些属性修饰的样式，比如`rounded-corners` `fancy-font`或者`no-bullets`，那么往往能够构造一个合适的混合器。如果你找不到，这时候构造一个混合器可能并不合适。

混合器并不一定总得生成相同的样式。可以通过在 `@include` 混合器时给混合器传参，来定制混合器生成的精确样式。当 `@include` 混合器时，参数其实就是可以赋值给css属性值的变量。如果你写过 `JavaScript` ，这种方式跟 `JavaScript` 的`function` 很像：
```
@mixin link-colors($normal, $hover, $visited) {
  color: $normal;
  &:hover { color: $hover; }
  &:visited { color: $visited; }
}
```
当混合器被 `@include` 时，你可以把它当作一个 css 函数来传参。如果你像下边这样写：
```
a {
  @include link-colors(blue, red, green);
}

//Sass最终生成的是：
a { color: blue; }
a:hover { color: red; }
a:visited { color: green; }
```
当你 `@include` 混合器时，有时候可能会很难区分每个参数是什么意思，参数之间是一个什么样的顺序。为了解决这个问题，SCSS 允许通过语法 `$name: value`的形式指定每个参数的值。这种形式的传参，参数顺序就不必再在乎了，只需要保证没有漏掉参数即可：
```
a {
    @include link-colors(
      $normal: blue,
      $visited: green,
      $hover: red
  );
}
```
尽管给混合器加参数来实现定制很好，但是有时有些参数我们没有定制的需要，这时候也需要赋值一个变量就变成很痛苦的事情了。所以 SCSS 允许混合器声明时给参数赋默认值。
为了在 `@include` 混合器时不必传入所有的参数，我们可以给参数指定一个默认值。参数默认值使用 `$name: default-value`的声明形式，默认值可以是任何有效的css 属性值，甚至是其他参数的引用，如下代码：
```
@mixin link-colors(
    $normal,
    $hover: $normal,
    $visited: $normal
  )
{
  color: $normal;
  &:hover { color: $hover; }
  &:visited { color: $visited; }
}
```
如果像下边这样调用：`@include link-colors(red)` ， `$hover` 和 `$visited` 也会被自动赋值为`red`。
### 继承
使用 SCSS 的时候，最后一个减少重复的主要特性就是选择器继承。基于Nicole Sullivan 面向对象的 css 的理念，选择器继承是说一个选择器可以继承为另一个选择器定义的所有样式。这个通过 `@extend` 语法实现，如下代码：
```
//通过选择器继承继承样式
.error {
  border: 1px solid red;
  background-color: #fdd;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}
```
在上边的代码中，`.seriousError` 将会继承样式表中任何位置处为 `.error` 定义的所有样式。以 `class="seriousError"`  修饰的 html元素最终的展示效果就好像是`class="seriousError error"`。相关元素不仅会拥有一个 `3px` 宽的边框，而且这个边框将变成红色的，这个元素同时还会有一个浅红色的背景，因为这些都是在 `.error` 里边定义的样式。

`.seriousError` 不仅会继承 `.error` 自身的所有样式，任何跟 `.error` 有关的组合选择器样式也会被 `.seriousError`以组合选择器的形式继承，如下代码:
```
//.seriousError从.error继承样式
.error a{  //应用到.seriousError a
  color: red;
  font-weight: 100;
}
h1.error { //应用到hl.seriousError
  font-size: 1.2rem;
}
```
如上所示，在`class="seriousError"`的 `html` 元素内的超链接也会变成红色和粗体。

那我们应该在什么时候使用继承呢？
>混合器主要用于展示性样式的重用，而类名用于语义化样式的重用。因为继承是基于类的（有时是基于其他类型的选择器），所以继承应该是建立在语义化的关系上。当一个元素拥有的类（比如说 `.seriousError`）表明它属于另一个类（比如说 `.error`），这时使用继承再合适不过了。

关于 `@extend` 有两个要点你应该知道。
+ 跟混合器相比，继承生成的 css 代码相对更少。因为继承仅仅是重复选择器，而不会重复属性，所以使用继承往往比混合器生成的 css 体积更小。如果你非常关心你站点的速度，请牢记这一点。
+ 继承遵从 css 层叠的规则。当两个不同的 css 规则应用到同一个 html 元素上时，并且这两个不同的 css 规则对同一属性的修饰存在不同的值， css 层叠规则会决定应用哪个样式。相当直观：通常权重更高的选择器胜出，如果权重相同，定义在后边的规则胜出。

混合器本身不会引起 css 层叠的问题，因为混合器把样式直接放到了 css 规则中，而继承存在样式层叠的问题。被继承的样式会保持原有定义位置和选择器权重不变。通常来说这并不会引起什么问题，但是知道这点总没有坏处。

即使我们说继承相比混合器生成的 css 代码相对更少，但是我们也要小心一种情况：
```
.foo .bar { @extend .baz; }
.bip .baz { a: b; }
```
在上边的例子中，SCSS 必须保证应用到 `.baz` 的样式同时也要应用到 `.foo .bar`（位于 `class="foo"` 的元素内的 `class="bar"` 的元素）。例子中有一条应用到`.bip .baz`（位于 `class="bip"` 的元素内的 `class="baz"` 的元素）的 css 规则。当这条规则应用到 `.foo .bar`时，可能存在三种情况，如下代码:
```
<!-- 继承可能迅速变复杂 -->
<!-- Case 1 -->
<div class="foo">
  <div class="bip">
    <div class="bar">...</div>
  </div>
</div>
<!-- Case 2 -->
<div class="bip">
  <div class="foo">
    <div class="bar">...</div>
  </div>
</div>
<!-- Case 3 -->
<div class="foo bip">
  <div class="bar">...</div>
</div>
```
为了应付这些情况，SCSS 必须生成三种选择器组合（仅仅是`.bip` `.foo` `.bar` 不能覆盖所有情况）。如果任何一条规则里边的后代选择器再长一点，sass需要考虑的情况就会更多。实际上sass并不总是会生成所有可能的选择器组合，即使是这样，选择器的个数依然可能会变得相当大，所以如果允许，尽可能避免这种用法。

值得一提的是，只要你想，你完全可以放心地继承有后代选择器修饰规则的选择器，不管后代选择器多长，但有一个前提就是，不要用后代选择器去继承。

### 导入SCSS
css有一个特别不常用的特性，即 `@import`规则，它允许在一个 css 文件中导入其他 css 文件。然而，后果是只有执行到 `@import` 时，浏览器才会去下载其他 css 文件，这导致页面加载起来特别慢。

sass也有一个 `@import` 规则，但不同的是，sass的`@import` 规则在生成 css 文件时就把相关文件导入进来。这意味着所有相关的样式被归纳到了同一个 css 文件中，而无需发起额外的下载请求。另外，所有在被导入文件中定义的变量和混合器均可在导入文件中使用。

当通过 `@import` 把 sass 样式分散到多个文件时，你通常只想生成少数几个 css 文件。那些专门为 `@import` 命令而编写的 sass 文件，并不需要生成对应的独立css文件，这样的 sass 文件称为局部文件。对此， sass 有一个特殊的约定来命名这些文件。

此约定即，sass 局部文件的文件名以下划线开头。这样，sass 就不会在编译时单独编译这个文件输出 css ，而只把这个文件用作导入。当你 `@import` 一个局部文件时，还可以不写文件的全名，即省略文件名开头的下划线。举例来说，你想导入`themes/_night-sky.scss` 这个局部文件里的变量，你只需在样式表中写 `@import "themes/night-sky";` 。

局部文件可以被多个不同的文件引用。当一些样式需要在多个页面甚至多个项目中使用时，这非常有用。

由于sass兼容原生的css，所以它也支持原生的CSS `@import` 。尽管通常在 sass 中使用 `@import` 时，sass 会尝试找到对应的 sass 文件并导入进来，但在下列三种情况下会生成原生的 CSS `@import`，尽管这会造成浏览器解析 css 时的额外下载：

+ 被导入文件的名字以 `.css` 结尾；
+ 被导入文件的名字是一个URL地址（比如http://www.sass.hk/css/css.css），由此可用谷歌字体API提供的相应服务；
+ 被导入文件的名字是CSS的url()值。
  这就是说，你不能用sass的 `@import` 直接导入一个原始的 css 文件，因为 sass 会认为你想用css原生的 `@import` 。但是，因为sass的语法完全兼容css，所以你可以把原始的css文件改名为.scss后缀，即可直接导入了。

更多用法可以参考[官网](https://www.sass.hk/docs/)



> 参考：[Sass 中文网](https://www.sass.hk/guide/)