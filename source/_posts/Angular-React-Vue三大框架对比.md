---
title: 'Angular,React,Vue三大框架对比'
date: 2018-03-04 21:56:38
tags: Angular-React-Vue
categories: 前端
---
最近的面试遇到面试官问前端三大框架的区别，我实际开发只用过Angular2，没有用过其他框架，所以感觉很吃亏，作为一个前端，怎么能拘泥于一个框架呢？
# 历史
**Angular** 是基于 TypeScript 的 Javascript 框架。由 Google 进行开发和维护，它被描述为“超级厉害的 JavaScript MVW框架”。Angular（也被称为 “Angular 2+”，“Angular 2” 或者 “ng2”）已被重写，是与 AngularJS（也被称为 “Angular.js” 或 “AngularJS 1.x”）不兼容的后续版本。当 AngularJS（旧版本）最初于2010年10月发布时，仍然在修复 bug，等等 —— 新的 Angular（sans JS）于 2016 年 9 月推出版本 2。最新的主版本是 4，因为版本 3 被跳过了。Google，Vine，Wix，Udemy，weather.com，healthcare.gov 和 Forbes 都使用 Angular。

**React** 被描述为 “用于构建用户界面的 JavaScript 库”。React 最初于 2013 年 3 月发布，由 Facebook 进行开发和维护，Facebook 在多个页面上使用 React 组件（但不是作为单页应用程序）。根据 [Chris Cordle](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2F%40chriscordle) [这篇文章](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2F%40chriscordle%2Fwhy-angular-2-4-is-too-little-too-late-ea86d7fa0bae)的统计，React 在 Facebook 上的使用远远多于 Angular 在 Google 上的使用。React 还被 Airbnb，Uber，Netflix，Twitter，Pinterest，Reddit，Udemy，Wix，Paypal，Imgur，Feedly，Stripe，Tumblr，Walmart 等使用

**Vue** 是 2016 年发展最为迅速的 JS 框架之一。Vue 将自己描述为一款“用于构建直观，快速和组件化交互式界面的 [MVVM](https://link.juejin.im?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FModel%25E2%2580%2593view%25E2%2580%2593viewmodel) 框架”。它于 2014 年 2 月首次由 Google 前员工 [Evan You](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fyyx990803) 发布（顺便说一句：尤雨溪那时候发表了一篇 [vue 发布首周的营销活动和数据](https://link.juejin.im?target=http%3A%2F%2Fblog.evanyou.me%2F2014%2F02%2F11%2Ffirst-week-of-launching-an-oss-project%2F)的博客文章）。尤其是考虑到 Vue 在没有大公司的支持的情况下，作为一个人开发的框架还能获得这么多的吸引力，这无疑是非常成功的。尤雨溪目前有一个包含数十名核心开发者的团队。2016 年，版本 2 发布。Vue 被阿里巴巴，百度，Expedia，任天堂，GitLab 使用 — 可以在 [madewithvuejs.com](https://link.juejin.im?target=https%3A%2F%2Fmadewithvuejs.com%2F) 找到一些小型项目的列表。

这三大框架都有大公司在用，而且社区非常强大，所以不用担心短时间会被淘汰。

# 使用语言
React 专注于使用 Javascript ES6。Vue 使用 Javascript ES5 或 ES6。Angular 依赖于 TypeScript。
TypeScript其实是 JavaScript 的超集，如果对 JavaScript 熟练的话，上手 TypeScript也不是什么很大的问题，而且 TypeScript 会最终编译成 JavaScript ES5，也能在其他浏览器正常运行 

# JSX 和 HTML
React 打破了长期以来的最佳实践。几十年来，开发人员试图分离 UI 模板和内联的 Javascript 逻辑，但是使用 JSX，这些又被混合了。这听起来很糟糕，但是你应该听彼得·亨特（Peter Hunt）的演讲 “[React：反思最佳实践](https://link.juejin.im?target=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3Dx7cQ3mrcKaY)”（2013 年 10 月）。他指出，分离模板和逻辑仅仅是技术的分离，而不是关注的分离。你应该构建组件而不是模板。组件是可重用的、可组合的、可单元测试的。

在 React 中，所有的组件的渲染功能都依靠 JSX。JSX 是使用 XML 语法编写 JavaScript 的一种语法糖。

JSX 是一个类似 HTML 语法的可选预处理器，并随后在 JavaScript 中进行编译。JSX 有一些怪癖 —— 例如，你需要使用 className 而不是 class，因为后者是 Javascript 的保留字。JSX 对于开发来说是一个很大的优势，因为代码写在同一个地方，可以在代码完成和编译时更好地检查工作成果。当你在 JSX 中输入错误时，React 将不会编译，并打印输出错误的行号。Angular 2 在运行时静默失败（如果使用 Angular 中的预编译，这个参数可能是无效的）。

JSX 意味着 React 中的所有内容都是 Javascript -- 用于JSX模板和逻辑。[Cory House](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2F%40housecor) 在 [2016 年 1 月的文章](https://link.juejin.im?target=https%3A%2F%2Fmedium.freecodecamp.org%2Fangular-2-versus-react-there-will-be-blood-66595faafd51) 中指出：“Angular 2 继续把 'JS' 放到 HTML 中。React 把 'HTML' 放到 JS 中。“这是一件好事，因为 Javascript 比 HTML 更强大。

Angular 模板使用特殊的 Angular 语法（比如 ngIf 或 ngFor）来增强 HTML。虽然 React 需要 JavaScript 的知识，但 Angular 会迫使你学习 [Angular 特有的语法](https://link.juejin.im?target=https%3A%2F%2Fangular.io%2Fguide%2Fcheatsheet)。

Vue 具有“[单个文件组件](https://link.juejin.im?target=https%3A%2F%2Fvuejs.org%2Fv2%2Fguide%2Fsingle-file-components.html)”。这似乎是对于关注分离的权衡 - 模板，脚本和样式在一个文件中，但在三个不同的有序部分中。这意味着你可以获得语法高亮，CSS 支持以及更容易使用预处理器（如 Jade 或 SCSS）。JSX 更容易调试，因为 Vue 不会显示不规范 HTML 的语法错误。这是不正确的，因为 Vue [转换 HTML 来渲染函数](https://link.juejin.im?target=https%3A%2F%2Fvuejs.org%2Fv2%2Fguide%2Frender-function.html) - 所以错误显示没有问题。

更抽象一点来看，我们可以把组件区分为两类：一类是偏视图表现的 (presentational)，一类则是偏逻辑的 (logical)。我们推荐在前者中使用模板，在后者中使用 JSX 或渲染函数。这两类组件的比例会根据应用类型的不同有所变化，但整体来说我们发现表现类的组件远远多于逻辑类组件。

# 状态管理和数据绑定
构建用户界面很困难，因为处处都有状态 - 随着时间的推移而变化的数据带来了复杂性。定义的状态工作流程对于应用程序的增长和复杂性有很大的帮助。对于复杂度不大的应用程序，就不必定义的状态流了，像原生 JS 就足够了。

它是如何工作的？组件在任何时间点描述 UI。当数据改变时，框架重新渲染整个 UI 组件 - 显示的数据始终是最新的。我们可以把这个概念称为“ UI 即功能”。

React 经常与 Redux 在一起使用。**Redux** 以三个[基本原则](https://link.juejin.im?target=http%3A%2F%2Fredux.js.org%2Fdocs%2Fintroduction%2FThreePrinciples.html)来自述：

*   单一数据源（Single source of truth）
*   State 是只读的（State is read-only）
*   使用纯函数执行修改（Changes are made with pure functions）

换句话说：整个应用程序的状态存储在单个 store 的状态树中。这有助于调试应用程序，一些功能更容易实现。状态是只读的，只能通过 action 来改变，以避免竞争条件（这也有助于调试）。编写 Reducer 来指定如何通过 action 来转换 state。

大多数教程和样板文件都已经集成了 Redux，但是如果没有它，你可以使用 React（你可能不需要在你的项目中使用 Redux）。Redux 在代码中引入了复杂性和相当强的约束。如果你正在学习React，那么在你要使用 Redux 之前，你应该考虑学习纯粹的 React。你绝对应该阅读 [Dan Abramov](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2F%40dan_abramov) 的“[你可能不需要 Redux](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2F%40dan_abramov%2Fyou-might-not-need-redux-be46360cf367)”。

[有些开发人员](https://link.juejin.im?target=https%3A%2F%2Fnews.ycombinator.com%2Fitem%3Fid%3D13151577) 建议使用 **[Mobx](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fmobxjs%2Fmobx) 代替 Redux**。你可以把它看作是一个 “自动的 Redux”，这使得事情一开始就更容易使用和理解。如果你想了解，你应该从[介绍](https://link.juejin.im?target=https%3A%2F%2Fmobxjs.github.io%2Fmobx%2Fgetting-started.html)开始。你也可以阅读 Robin 的 [Redux 和 MobX 的比较](https://link.juejin.im?target=https%3A%2F%2Fwww.robinwieruch.de%2Fredux-mobx-confusion%2F)。他还提供了有关[从 Redux 迁移到 MobX](https://link.juejin.im?target=https%3A%2F%2Fwww.robinwieruch.de%2Fmobx-react%2F) 的信息。如果你想查找其他 Flux 库，[这个列表](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fvoronianski%2Fflux-comparison)非常有用。如果你是来自 MVC 的世界，那么你应该阅读 [Mikhail Levkovsky](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2F%40mlovekovsky) 的文章“[Redux 中的思考（当你所知道的是 MVC）](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2Fp%2Fthinking-in-redux-when-all-youve-known-is-mvc-c78a74d35133%3Fsource%3Duser_popover)”。

Vue 可以使用 Redux，但它提供了 [Vuex](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvuex) 作为自己的解决方案。

React 和 Angular 之间的巨大差异是 **单向与双向绑定**。当 UI 元素（例如，用户输入）被更新时，Angular 的双向绑定改变 model 状态。React 只有一种方法：先更新 model，然后渲染 UI 元素。Angular 的方式实现起来代码更干净，开发人员更容易实现。React 的方式会有更好的数据总览，因为数据只能在一个方向上流动（这使得调试更容易）。

这两个概念各有优劣。你需要了解这些概念，并确定这是否会影响你选择框架。文章“[双向数据绑定：Angular 2 和 React](https://link.juejin.im?target=https%3A%2F%2Fwww.accelebrate.com%2Fblog%2Ftwo-way-data-binding-angular-2-and-react%2F)”和[这个 Stackoverflow 上的问题](https://link.juejin.im?target=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F34519889%2Fcan-anyone-explain-the-difference-between-reacts-one-way-data-binding-and-angula)都提供了一个很好的解释。[在这里](https://link.juejin.im?target=http%3A%2F%2Fn12v.com%2F2-way-data-binding%2F)你可以找到一些交互式的代码示例（3 年前的示例（，只适用于 Angular 1 和 React）。最后，Vue 支持[单向绑定和双向绑定](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2Fjs-dojo%2Fexploring-vue-js-reactive-two-way-data-binding-da533d0c4554)（默认为单向绑定）。

如果你想进一步阅读，这有一篇长文，是有关状态的不同类型和 [Angular 应用程序中的状态管理](https://link.juejin.im?target=https%3A%2F%2Fblog.nrwl.io%2Fmanaging-state-in-angular-applications-22b75ef5625f)（[Victor Savkin](https://link.juejin.im?target=https%3A%2F%2Fmedium.com%2F%40vsavkin)）。

# 学习曲线
要学习 Vue，你只需要有良好的 HTML 和 JavaScript 基础。有了这些基本的技能，你就可以非常快速地通过阅读 [指南](https://cn.vuejs.org/v2/guide/) 投入开发。

Angular 的学习曲线是非常陡峭的——作为一个框架，它的 API 面积比起 Vue 要大得多，你也因此需要理解更多的概念才能开始有效率地工作。当然，Angular 本身的复杂度是因为它的设计目标就是只针对大型的复杂应用；但不可否认的是，这也使得它对于经验不甚丰富的开发者相当的不友好。

在调试方面，React 和 Vue 的黑魔法更少是一个加分项。找出 bug 更容易，因为需要看的地方少了，堆栈跟踪的时候，自己的代码和那些库之间有更明显的区别。使用 React 的人员报告说，他们永远不必阅读库的源代码。但是，在调试 Angular 应用程序时，通常需要调试 Angular 的内部来理解底层模型。从好的一面来看，从 Angular 4 开始，错误信息应该更清晰，更具信息性。



# 总结
Vue 更像是 Angular2 和 React 的集合，这并不奇怪，Vue充分吸收了 Angular2 和 React 这两位前辈的优点和长处，并不断运用在自身，也难怪 Vue 在 Github 上 star数越来越多

>参考
[[译] 2017 年比较 Angular、React、Vue 三剑客](https://juejin.im/post/5a0d5df1f265da43062a542f)
[对比其他框架](https://cn.vuejs.org/v2/guide/comparison.html)
