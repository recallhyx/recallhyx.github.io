---
title: Angular2动态加载组件
date: 2017-07-01 11:21:23
categories: 前端
tags: angular2
---
（~~日常水一下~~，由于6月要复习应付考试啥的，没时间写博客，本来想写一篇关于自动化测试的博文，最近又在学 Angular2，解决了一个技术问题，就先写这篇文章）
***
好的，进入正文，关于Angular2动态加载组件，之前在网上搜到的是用 __DynamicComponentLoader()__这个函数，但是这个函数在 Angular2就弃用了，于是再搜了一下，找到了这个函数__ComponentFactoryResolver()__接下来就来看一下怎么使用
这篇博文借鉴了->[这篇文章](https://dotblogs.com.tw/wellwind/2017/06/21/dynamic-component-with-component-factory-resolver)，上面他的实例的完整代码，但是我有自己的demo。
__第一步__
创建一个名为__dynamic-component.directive.ts__的文件
```
import {Directive,ViewContainerRef} from '@angular/core';

@Directive({
  selector: '[appDynamicComponent]'
})

export class DynamicComponentDirective{
  constructor(public viewContainerRef:ViewContainerRef){}
}
```
代码在上面，首先，这个文件用了@Directive这个装饰器，简单来说就是找到html中带有 __selector__中属性的那个标签，在这里就是找到 标签中带有 __appDybanucComponent__这个属性的标签。
然后我们export 一个类，这个类注入了__ViewContainerRef__，它可以让我们得知目前所在的HTML元素中包含的View內容，也可以通过它來改变View的结果(ex: 动态的产生Component、移除某个Component等等)。
也就是说，这个文件做了两件事情，第一件事情就是找到要改变的标签，第二件事情就是对它进行改变。
这里我们还导出了 DynamicComponentDirective 这个类，下面会用到
__第二步__
既然我们是对带有 __appDynamicComponent__的标签进行操作，那么我们的html里面某个标签有这个属性，在 app.component.html中写以下代码
```
<div id="wrapper">
  <div id="body" class="clearfix">
    <div id="left">
      <div class="inner">
        left
        <!--看这里-------------------------------------------------------------------------->
        <ng-template appDynamicComponent></ng-template>
        <br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
        <br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
      </div>
    </div>
    <div id="right">
      <div class="inner">
        <button class="button" (click)="CreateGroup()">添加组</button>
        <br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
        <br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
      </div>
    </div>
  </div>
</div>
```
上面的<ng-template>标签里面就有__appDynamicComponent__属性，于是就找到了对应的标签。
我们还写了一个__button__，点击之后就会进入__CreateGroup()__这个函数，这个函数就是我们要加载组件的函数
__第三步__
我们找到了这个标签，接下来就要动态的加载组件了，那么我们应该就要有另外一个组件，创建一个名为__text.component.ts__的文件
```
import {Component} from '@angular/core'

@Component({
  selector: 'text',
  template:'<label>text</label>'
})

export class TextComponent{}

```
这个文件就只有一行文本 __text__而已，并且导出一个 TextComponent 这个类，我们之后会用到
__第四步__
接下来就开始进行加载了
```
import {Component, ComponentFactoryResolver, ViewChild, } from '@angular/core';
import {TextComponent} from "./text.component";
import {DynamicComponentDirective} from './dynamic-component.directive'

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
  entryComponents: [TextComponent]
})
export class AppComponent {

  @ViewChild(DynamicComponentDirective) componentHost: DynamicComponentDirective;
  constructor(private componentFactoryResolver: ComponentFactoryResolver){}

  CreateGroup(){
    const componentFactory = this.componentFactoryResolver.resolveComponentFactory(TextComponent)

    const viewContainerRef = this.componentHost.viewContainerRef;

    viewContainerRef.clear();

    const componentRef = viewContainerRef.createComponent(componentFactory);

  }
}
```
首先我们看到在@Component这个装饰器里面多了一个__entryComponents__，意思是当加载了当前组件之后，告诉编译器去编译__entryComponents__数组里面的组件，之后就能给下一步使用了。
然后我们看 __AppComponent__这个类
它有一个__@ViewChild__装饰器，这个装饰器是用来访问一个类和这个类的方法的，访问的是__DynamicComponentDIrective__这个类，我们在第一步就创建了
然后就是注入一个__ComponentFactoryResolver__这个类，好像是对组件进行处理，__viewContainerRef__这个变量赋值为 我们在DynamicComponentDirective里面的viewContainerRef。
最后调用方法 __viewContainerRef.createComponent(componentFactory)__实例化组件，到这里还没完，我们要把用到的组件在 __app.module.ts__里面声明
__最后__
```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';


import { AppComponent } from './app.component';
import { FormsModule } from '@angular/forms'

import { WeUIModule} from 'angular-weui'
import {TextComponent} from "./text.component";
import {DynamicComponentDirective} from "./dynamic-component.directive";
@NgModule({
  declarations: [
    AppComponent,
    TextComponent,
    DynamicComponentDirective
  ],
  imports: [
    BrowserModule,
    FormsModule,
    WeUIModule
  ],
  entryComponents: [
    TextComponent
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```
__顺带一提__
我用的css样式
```
.clearfix { *zoom: 1;}
.clearfix:after{ content: ""; display: block; height: 0; visibility:hidden; clear: both;}
#header{ width: 96%; height: 90px; margin: 0 auto; background: #f60;}
#body{ width: 96%; margin: 0 auto;
  clear: both;}
#left{ float: left; width: 50%; background: #ccc;float: left}
#right{ margin-left: 50%; background: orange;
  overflow: hidden;
  position: fixed;}
```
__效果图__
在没点击按钮的时候

![之前](http://upload-images.jianshu.io/upload_images/5834506-8a44d88d1663a9d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击按钮之后

![之后](http://upload-images.jianshu.io/upload_images/5834506-6521e8dadb26b1a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__总结__
其实一开始我是摸不着头脑的，不知道要怎么动态加载组件，上网找了好久也找不到，之后也是看别人的代码一步一步试才懂得，这个动态加载组件只是入门，我再挖掘一下其他用法






