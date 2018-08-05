---
title: Angular Material2 常用组件用法
date: 2018-01-13 10:42:31
categories: 前端
tags: angular2
---
（~~时隔2个多月终于有时间来写博文了~~）
先总结一下，11月接到一个项目，做到12月然后停工准备考试，到1月才重新开始工作，把一些收尾工作做了，这个项目也是第一次用 Angular Material2 组件，其中还是有一些坑的，就记录一下
__这是[Angular Material官网](https://material.angular.io/)__
一、安装Angular Material
-----------------------
首先根据官网的指导`Get Start`安装Material 
__坑1：__ 安装完的 Angular Material，有时候不能 `AOT预编译`，这个时候要看依赖包文件，我是用 [webpack](https://github.com/gdi2290/angular-starter) 这个框架，所以看`package.json`就可以看到所有依赖包
![依赖包](http://upload-images.jianshu.io/upload_images/5834506-d205620390122a95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，除了`"@angular/material": "^5.0.0-rc0"`为5.0.0之外，还有其他包也要升级，至少要升级到5.0.0
__坑2：__即使你已经升级到最新版，还有可能不能`AOT编译`，当初这个问题搞了我好久，（~~也不久，就一个晚上~~），然后就有种直觉是框架的问题，第二天果断换框架，其实也就是升级框架，之前用的是 `webpack6`，看了`github`发现 `webpack` 已经有7了，果断换成`webpack7`，然后就能 `AOT预编译`了
__总结：__一开始遇到问题是很正常的，这还是第一步，所以第一步不好好解决的话，后面的工作会很难开展的，幸好是项目一开始就发现不能`AOT预编译`的问题，不然到后期大改就很麻烦了
二、使用Angular Material
-----------------------
关于怎么使用 Angular Material 官网并没有说明怎么用，一开始我以为引入组件就能用的，其实并不是，看了官网的例子也没有说，之后通过官网`在线编辑`，才发现玄机
```
import {
  MatAutocompleteModule,
  MatButtonModule,
  MatButtonToggleModule,
  MatCardModule,
  MatCheckboxModule,
  MatChipsModule,
  MatDatepickerModule,
  MatDialogModule,
  MatExpansionModule,
  MatGridListModule,
  MatIconModule,
  MatInputModule,
  MatListModule,
  MatMenuModule,
  MatNativeDateModule,
  MatPaginatorModule,
  MatProgressBarModule,
  MatProgressSpinnerModule,
  MatRadioModule,
  MatRippleModule,
  MatSelectModule,
  MatSidenavModule,
  MatSliderModule,
  MatSlideToggleModule,
  MatSnackBarModule,
  MatSortModule,
  MatTableModule,
  MatTabsModule,
  MatToolbarModule,
  MatTooltipModule,
  MatStepperModule,
} from '@angular/material';
```
```
import {CdkTableModule} from '@angular/cdk/table';

@NgModule({
  exports: [
    CdkTableModule,
    MatAutocompleteModule,
    MatButtonModule,
    MatButtonToggleModule,
    MatCardModule,
    MatCheckboxModule,
    MatChipsModule,
    MatStepperModule,
    MatDatepickerModule,
    MatDialogModule,
    MatExpansionModule,
    MatGridListModule,
    MatIconModule,
    MatInputModule,
    MatListModule,
    MatMenuModule,
    MatNativeDateModule,
    MatPaginatorModule,
    MatProgressBarModule,
    MatProgressSpinnerModule,
    MatRadioModule,
    MatRippleModule,
    MatSelectModule,
    MatSidenavModule,
    MatSliderModule,
    MatSlideToggleModule,
    MatSnackBarModule,
    MatSortModule,
    MatTableModule,
    MatTabsModule,
    MatToolbarModule,
    MatTooltipModule,
  ]
})
export class DemoMaterialModule {}
```
在官网的`在线编辑`的`main.ts（或者app.module.ts）`里面，是有以上两段代码的，这就是把所有的material组件作为一个`module`，然后再把这个`module`放到整个项目内唯一一个AppModule里面，就像官网做的那样
![](http://upload-images.jianshu.io/upload_images/5834506-ff28bdb21ec7ed75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样就能成功引用组件了
三、常用组件用法
-----------------------
__1、button__
button是最基础的组件，最常用的就是提交表单的时候，这里用一个验证码的例子来说明button怎么用
![](http://upload-images.jianshu.io/upload_images/5834506-7a7ce56b8de53d0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里点击`获取验证码`后就`disable`使按钮不可点击，然后再根据后端返回值，来进行倒数![](http://upload-images.jianshu.io/upload_images/5834506-01a1eb82bcad1d38.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/5834506-8211570a66aa25e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
倒计时代码如下：
```
  //倒计时
  countdown = 30;
  setTime(obj){
    if(this.countdown==0){
      this.disablebtn = false;
      this.tipstring = "获取验证码";
      this.countdown = 30;
      return;
    }else{
      this.disablebtn = true;
      this.tipstring = "（"+this.countdown+"s）";
      this.countdown--;
    }
    setTimeout(()=>{
      this.setTime(obj);
    },1000)
  }
```
这样，要一个变量控制是否`disable`按钮，然后根据业务逻辑自己调整即可
__2、tabs__
![tabs](http://upload-images.jianshu.io/upload_images/5834506-cf6564f179a8f4fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里主要讲 tabs的事件怎么用
```
<!-- html -->
<mat-tab-group (selectedTabChange)="select($event)">
  <mat-tab label="Tab 1">Content 1</mat-tab>
  <mat-tab label="Tab 2">Content 2</mat-tab>
</mat-tab-group>

```
```
//ts
select = (tabChangeEvent: MatTabChangeEvent): void => {
    console.log(tabChangeEvent.index);
    switch (tabChangeEvent.index) {
      case 0: {
          ...
      } break;
      case 1: {
          ...
      } break;
    }
  };
```
上面的 `tabChangeEvent.index`就是你现在选中的tab，根据不同的tab可以做不同的事情
__3、snackbar__
![snackbar](http://upload-images.jianshu.io/upload_images/5834506-ff0a5a3de829f200.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
snackbar 就相当于 Android 的 toast，为了提醒用户一些信息的
snackbar 不难，重点在于它可以复用 ，它有一个属性`data`可以用来传值
![传值](http://upload-images.jianshu.io/upload_images/5834506-963e263c65ee9bcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
之后只需要在snackbar组件中就可以用 `data`属性了
__4、dialog__
对话框是一个很重要的交互方式，它可以用于提醒用户信息，也可以用来进行一些输入，这里给出一个复用的例子，用于通知用户一些信息
```
<!-- html -->
<h1 mat-dialog-title>{{data.title}}</h1>
<div mat-dialog-content>
  <p>{{data.content}}</p>
</div>
<div mat-dialog-actions align="center">
  <button *ngIf="data.isOkButton" mat-button (click)="data.OkEvent()" tabindex="2">{{data.OkButtonContent}}</button>
  <button *ngIf="data.isNoButton" mat-button (click)="data.NoEvent()" tabindex="1">{{data.NoButtonContent}}</button>
</div>
```
```
export class CustomDialogNoticeComponent{
  constructor(
    public dialogRef: MatDialogRef<CustomDialogNoticeComponent>,
    @Inject(MAT_DIALOG_DATA) public data: any) {
  }

}
export interface CustomDialogData{
  title:string;
  content:string;
  isOkButton:boolean;
  isNoButton:boolean;
  OkEvent?:Function;
  NoEvent?:Function;
  OkButtonContent?:string;
  NoButtonContent?:string;
}

```
这样，就可以在打开这个对话框的时候，传一个 `CustomDialogData`的对象，就能根据这些信息配置对话框
__5、datepicker__
这里主要讲怎么换成中文，因为默认是按照老美那边的标识的，所以要用在我们这边，就需要做一些改动
首先，安装 [momentjs](http://momentjs.cn/)
>momentjs: JavaScript 日期处理类库

`npm install moment --save`
然后按照官网的例子
```
@Component({
  ...
  providers: [
    // The locale would typically be provided on the root module of your application. We do it at
    // the component level here, due to limitations of our example generation script.
    { provide: MAT_DATE_LOCALE, useValue: 'zh-cn' },

    // `MomentDateAdapter` and `MAT_MOMENT_DATE_FORMATS` can be automatically provided by importing
    // `MatMomentDateModule` in your applications root module. We provide it at the component level
    // here, due to limitations of our example generation script.
    { provide: DateAdapter, useClass: MomentDateAdapter, deps: [MAT_DATE_LOCALE] },
    { provide: MAT_DATE_FORMATS, useValue: MAT_MOMENT_DATE_FORMATS },
  ],
})
  ...
```
将`{ provide: MAT_DATE_LOCALE, useValue: 'zh-cn' },`的`useValue`改成`'zh-cn`就是中文了，[具体时区代号参考](https://github.com/moment/moment/tree/develop/locale)
##四、总结
总的来说 Angular Material 组件集成了很多东西，也方便了我们的使用，适合敏捷开发，但是会带来一些依赖性的问题，比如你要为了使用 Angular Material 而改变接口的格式或者和后端的一些对接问题。但是 Angular Material 可以更加复用，我只是很低程度的复用而已