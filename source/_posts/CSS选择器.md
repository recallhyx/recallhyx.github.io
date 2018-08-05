---
title: CSS 都有哪些选择器？
date: 2018-02-20 11:55:46
categories: 前端
tags: css
---
## 什么是选择器
CSS有一套用于描述其语言的术语。
```
div {
  color: red;
}
```
在CSS的术语中，上面这段代码被称为一条规则（`rule`）。这条规则以选择器 `div` 开始，它选择要在DOM中哪些元素上使用这条规则。
## 选择器介绍
### 标签选择器（类型选择器）
```
div {
  color: red;
}
```
上述代码就是标签选择器，选择所有`div`标签。因为`div`是一个标签，在CSS规范中称为类型选择器。
### 类选择器
```
.name {
  color: red;
}
```
通过 `.` 来指定一个类
上述代码就是选择所有标签内有`class="name"`的标签，注意，如果被选择的标签内还有子标签，且没有被其他选择器选中，则样式会保持默认。
```
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
<meta charset="utf-8" />
<title>类型选择器</title>
<style>
.title {
    font-size: 30px;
}
</style>
</head>
<body>
<p class="title">标题<p>正文内容</p><p>
<p>多类选择符的使用</p>
</body>
</html>
```
效果如下
![](http://upload-images.jianshu.io/upload_images/5834506-fd4540c9874b6931.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来我们把 `title`的 `font-size`改为`20px`，然后加上
```
.content {
    font-size: 30px;
}
```
再在`<p>正文内容</p>`内加上类`content`，即下面代码
```
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
<meta charset="utf-8" />
<title>类型选择器</title>
<style>
.title {
    font-size: 20px;
}
.content {
    font-size: 30px;
}

</style>
</head>
<body>
<p class="title">标题<p class="content">正文内容</p></p>
<p>多类选择符的使用</p>
</body>
</html>
```
效果如下
![](http://upload-images.jianshu.io/upload_images/5834506-c002d14b7c33a9f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，嵌套在子标签里面的类优先匹配了`content`规则
我们再改一下
```
.content.note {
    color:red;
}
```
```
<p class="content note">多类选择符的使用</p>
```
![](http://upload-images.jianshu.io/upload_images/5834506-6e8d6eb089f740c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这时，两个规则同时作用在`多类选择符的使用`上，即匹配同时具有`content`和`note`的元素，如果元素只有其中一个类，那这个规则不会匹配到这个元素，这就是多类选择器
如果不同的类选择器作用在同一个元素上，则样式会叠加
```
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
<meta charset="utf-8" />
<title>类型选择器</title>
<style>
.title {
    font-size: 20px;
}
.content {
    color: red;
}

</style>
</head>
<body>
<p class="title">标题<p >正文内容</p></p>
<p class="title content">多类选择符的使用</p>
</body>
</html>
```
![](http://upload-images.jianshu.io/upload_images/5834506-5053c8d0166f6219.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中，`多类选择符的使用`同时具有`title`和`content`的样式。
注意，如果定义了的规则，CSS改变的是同一个样式，则后定义的规则会覆盖前面的规则。
```
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
<meta charset="utf-8" />
<title>类型选择器</title>
<style>
.title {
    color: green;
}
.content {
    color: red;
}

</style>
</head>
<body>
<p class="title">标题<p >正文内容</p></p>
<p class="title content">多类选择符的使用</p>
</body>
</html>
```
![](http://upload-images.jianshu.io/upload_images/5834506-4523178f869bf09c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里，虽然`多类选择符的使用`这个p标签同时有`title`和`content`两个规则，但是由于`content`是在之后定义的，所以`content`生效。
### id选择器
```
#name{
  color: red;
}
```
通过`#`来指定一个id
>通过设置元素的 `id` 属性为该元素制定ID。ID名由开发者指定。每个ID在文档中必须是唯一的。

上述只是规范，因为你可以设置多个一样的id，而且浏览器不报错，但是，你只能通过`getElementById()`的方法获得第一个符合id的元素，而不能获得全部符合id的元素，这是不推荐的。
```
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
<meta charset="utf-8" />
<title>id选择器</title>
<style>
#subtitle {
    color: red;
}
#content {
    color: blue;
}
</style>
</head>
<body>
<h1 id="subtitle">标题</h1>
<p id="content">正文内容</p>
</body>
</html>
```
![](http://upload-images.jianshu.io/upload_images/5834506-dd78e1cd6e4475c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意，id选择器不能在同个元素上定义多个，比如`id="a b"`就是错误的写法。
### 伪类选择器和伪元素选择器（伪对象选择器）
CSS伪类（`pseudo-class`）是加在选择器后面的用来指定元素状态的关键字。比如，`:hover`。CSS伪类适用于用户使用指示设备虚指一个元素（没有激活它）的情况。这个样式会被任何与链接相关的伪类重写，像`:link`, `:visited`, 和 `:active`等。为了确保生效，`:hover`规则需要放在`:link`和`:visited`规则之后，但是在`:active`规则之前，按照LVHA的循顺序声明`:link`－`:visited`－`:hover`－`:active`。") 会在鼠标悬停在选中元素上时应用相应的样式。



#### 伪类选择器列表：

| 选择器 | 版本 | 描述 |
| --- | --- | --- |
| [E:link](http://css.doyoe.com/selectors/pseudo-classes/link.htm) | CSS1 | 设置超链接a在未被访问前的样式。 |
| [E:visited](http://css.doyoe.com/selectors/pseudo-classes/visited.htm) | CSS1 | 设置超链接a在其链接地址已被访问过时的样式。 |
| [E:hover](http://css.doyoe.com/selectors/pseudo-classes/hover.htm) | CSS1/2 | 设置元素在其鼠标悬停时的样式。 |
| [E:active](http://css.doyoe.com/selectors/pseudo-classes/active.htm) | CSS1/2 | 设置元素在被用户激活（在鼠标点击与释放之间发生的事件）时的样式。 |
| [E:focus](http://css.doyoe.com/selectors/pseudo-classes/focus.htm) | CSS1/2 | 设置元素在成为输入焦点（该元素的onfocus事件发生）时的样式。 |
| [E:lang(fr)](http://css.doyoe.com/selectors/pseudo-classes/lang(fr).htm) | CSS2 | 匹配使用特殊语言的E元素。 |
| [E:not(s)](http://css.doyoe.com/selectors/pseudo-classes/not(s).htm) | CSS3 | 匹配不含有s选择符的元素E。 |
| [E:root](http://css.doyoe.com/selectors/pseudo-classes/root.htm) | CSS3 | 匹配E元素在文档的根元素。 |
| [E:first-child](http://css.doyoe.com/selectors/pseudo-classes/first-child.htm) | CSS2 | 匹配父元素的第一个子元素E。 |
| [E:last-child](http://css.doyoe.com/selectors/pseudo-classes/last-child.htm) | CSS3 | 匹配父元素的最后一个子元素E。 |
| [E:only-child](http://css.doyoe.com/selectors/pseudo-classes/only-child.htm) | CSS3 | 匹配父元素仅有的一个子元素E。 |
| [E:nth-child(n)](http://css.doyoe.com/selectors/pseudo-classes/nth-child(n).htm) | CSS3 | 匹配父元素的第n个子元素E。 |
| [E:nth-last-child(n)](http://css.doyoe.com/selectors/pseudo-classes/nth-last-child(n).htm) | CSS3 | 匹配父元素的倒数第n个子元素E。 |
| [E:first-of-type](http://css.doyoe.com/selectors/pseudo-classes/first-of-type.htm) | CSS3 | 匹配同类型中的第一个同级兄弟元素E。 |
| [E:last-of-type](http://css.doyoe.com/selectors/pseudo-classes/last-of-type.htm) | CSS3 | 匹配同类型中的最后一个同级兄弟元素E。 |
| [E:only-of-type](http://css.doyoe.com/selectors/pseudo-classes/only-of-type.htm) | CSS3 | 匹配同类型中的唯一的一个同级兄弟元素E。 |
| [E:nth-of-type(n)](http://css.doyoe.com/selectors/pseudo-classes/nth-of-type(n).htm) | CSS3 | 匹配同类型中的第n个同级兄弟元素E。 |
| [E:nth-last-of-type(n)](http://css.doyoe.com/selectors/pseudo-classes/nth-last-of-type(n).htm) | CSS3 | 匹配同类型中的倒数第n个同级兄弟元素E。 |
| [E:empty](http://css.doyoe.com/selectors/pseudo-classes/empty.htm) | CSS3 | 匹配没有任何子元素（包括text节点）的元素E。 |
| [E:checked](http://css.doyoe.com/selectors/pseudo-classes/checked.htm) | CSS3 | 匹配用户界面上处于选中状态的元素E。(用于input type为radio与checkbox时) |
| [E:enabled](http://css.doyoe.com/selectors/pseudo-classes/enabled.htm) | CSS3 | 匹配用户界面上处于可用状态的元素E。 |
| [E:disabled](http://css.doyoe.com/selectors/pseudo-classes/disabled.htm) | CSS3 | 匹配用户界面上处于禁用状态的元素E。 |
| [E:target](http://css.doyoe.com/selectors/pseudo-classes/target.htm) | CSS3 | 匹配相关URL指向的E元素。 |
| [@page:first](http://css.doyoe.com/selectors/pseudo-classes/@page-first.htm) | CSS2 | 设置页面容器第一页使用的样式。仅用于[@page](http://css.doyoe.com/rules/@page.htm)规则 |
| [@page:left](http://css.doyoe.com/selectors/pseudo-classes/@page-left.htm) | CSS2 | 设置页面容器位于装订线左边的所有页面使用的样式。仅用于[@page](http://css.doyoe.com/rules/@page.htm)规则 |
| [@page:right](http://css.doyoe.com/selectors/pseudo-classes/@page-right.htm) | CSS2 | 设置页面容器位于装订线右边的所有页面使用的样式。仅用于[@page](http://css.doyoe.com/rules/@page.htm)规则 |
#### 伪元素选择器列表：
| 选择符 | 版本 | 描述 |
| --- | --- | --- |
| [E:first-letter/E::first-letter](http://css.doyoe.com/selectors/pseudo-element/first-letter.htm) | CSS1/3 | 设置对象内的第一个字符的样式。 |
| [E:first-line/E::first-line](http://css.doyoe.com/selectors/pseudo-element/first-line.htm) | CSS1/3 | 设置对象内的第一行的样式。 |
| [E:before/E::before](http://css.doyoe.com/selectors/pseudo-element/before.htm) | CSS2/3 | 设置在对象前（依据对象树的逻辑结构）发生的内容。用来和content属性一起使用 |
| [E:after/E::after](http://css.doyoe.com/selectors/pseudo-element/after.htm) | CSS2/3 | 设置在对象后（依据对象树的逻辑结构）发生的内容。用来和content属性一起使用 |
| [E::placeholder](http://css.doyoe.com/selectors/pseudo-element/placeholder.htm) | CSS3 | 设置对象文字占位符的样式。 |
| [E::selection](http://css.doyoe.com/selectors/pseudo-element/selection.htm) | CSS3 | 设置对象被选择时的颜色。 |

* CSS3将伪元素选择器(Pseudo-Element Selectors)前面的单个冒号(:)修改为双冒号(::)用以区别伪类选择器(Pseudo-Classes Selectors)，但以前的写法仍然有效。


伪类和伪元素（`pseudo-elements`）不仅可以让你为符合某种文档树结构的元素指定样式，还可以为符合某些外部条件的元素指定样式：浏览历史(比如是否访问过 (`:visited`)， 内容状态(如 `:checked` CSS 伪类选择器表示任何处于选中状态的`radio`(<input type="radio">), `checkbox` (<input type="checkbox">) 或(`"select"`) 元素中的`option` HTML元素(`"option"`)) 。用户通过点击元素或选择其他的值，可以改变该元素的 `:checked` 状态，并`:checked`属性赋给一个新的对象(例如其他的`option`值)。") ), 鼠标位置 (如`:hover`).

### 关系选择器
| 选择器 | 名称 | 版本 | 描述 |
| --- | --- | --- | --- |
| [E F](http://css.doyoe.com/selectors/relationship/ef.htm) | 包含选择器(Descendant combinator) | CSS1 | 选择所有被E元素包含的F元素。 |
| [E>F](http://css.doyoe.com/selectors/relationship/e-child-f.htm) | 子选择器(Child combinator) | CSS2 | 选择所有作为E元素的子元素F。 |
| [E+F](http://css.doyoe.com/selectors/relationship/e-adjacent-f.htm) | 相邻选择器(Adjacent sibling combinator) | CSS2 | 选择紧贴在E元素之后F元素。 |
| [E~F](http://css.doyoe.com/selectors/relationship/e-brother-f.htm) | 兄弟选择器(General sibling combinator) | CSS3 | 选择E元素所有兄弟元素F。 |

### 属性选择器
| 选择器 | 版本 | 描述 |
| --- | --- | --- |
| [E[att]](http://css.doyoe.com/selectors/attribute/att.htm) | CSS2 | 选择具有att属性的E元素。 |
| [E[att="val"]](http://css.doyoe.com/selectors/attribute/att2.htm) | CSS2 | 选择具有att属性且属性值等于val的E元素。 |
| [E[att~="val"]](http://css.doyoe.com/selectors/attribute/att3.htm) | CSS2 | 选择具有att属性且属性值为一用空格分隔的字词列表，其中一个等于val的E元素。 |
| [E[att^="val"]](http://css.doyoe.com/selectors/attribute/att4.htm) | CSS3 | 选择具有att属性且属性值为以val开头的字符串的E元素。 |
| [E[att$="val"]](http://css.doyoe.com/selectors/attribute/att5.htm) | CSS3 | 选择具有att属性且属性值为以val结尾的字符串的E元素。 |
| [E[att*="val"]](http://css.doyoe.com/selectors/attribute/att6.htm) | CSS3 | 选择具有att属性且属性值为包含val的字符串的E元素。 |
| [E[att&#124;="val"]](http://css.doyoe.com/selectors/attribute/att7.htm) | CSS2 | 选择具有att属性且属性值为以val开头并用连接符"-"分隔的字符串的E元素，如果属性值仅为val，也将被选择。 |


## 选择器优先级算法
如果多于一个规则指定了相同的属性值都应用到一个元素上，CSS规定拥有更高确定度的选择器优先级更高。ID选择器比类选择器更具确定度, 而类选择器比标签选择器更具确定度。
下面是利用权重的方式来计算优先级：

>最高优先级是 (直接在标签中的设置样式，假设级别为1000)<div style="color:Red;"></div> 　　
次优先级是（ID选择器 ,假设级别为100） #myDiv{color:Red;} 　　
其次优先级是（类选择器，假设级别为10） .divClass{color:Red;} 　　
最后优先级是 （标签选择器，假设级别是 1） div{color:Red;}
全局选择器(*), 以及组合选择器(+, > ~),和:not选择器对权重没有影响。（但是，在 :not() 内部声明的选择器是会影响优先级）。

还有一种优先级算法：

>1.优先级就近原则，同权重情况下样式定义最近者为准;
2.载入样式以最后载入的定位为准;
3.!important >  id > class > tag
4.important 比 内联优先级高，但内联比 id 要高

使用 `!important` 是一个坏习惯，应该尽量避免，因为这破坏了样式表中的固有的级联规则 使得调试找bug变得更加困难了。当两条相互冲突的带有 `!important` 规则的声明被应用到相同的元素上时，拥有更大优先级的声明将会被采用。

一些经验法则：

+ __一定__要优化考虑使用样式规则的优先级来解决问题而不是 `!important`
+ __只有__在需要覆盖全站或外部 css（例如引用的 ExtJs 或者 YUI ）的特定页面中使用 `!important`
+ __永远不要__在全站范围的 css 上使用 `!important`
+ __永远不要__在你的插件中使用 `!important`


>参考:
[MDN web docs](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Getting_started/Selectors)
[css参考手册](http://css.doyoe.com/)