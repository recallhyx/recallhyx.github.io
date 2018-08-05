---
title: 解决iPhone拍照旋转
date: 2018-06-22 10:11:34
tags: javascript
categories: 前端
---

# 问题
一般我们上传图片，可以选择拍照或者从图库中选择，如果用 iPhone ，选择拍照，并且是竖着拍照，则会出现图片是逆时针旋转90°。但是安卓手机却没问题。
![](https://upload-images.jianshu.io/upload_images/5834506-75114f8bc12ae289.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面是测试图，分别是竖着拍，顺时针旋转90°拍，顺时针旋转180°拍，顺时针旋转270°拍：
![](https://upload-images.jianshu.io/upload_images/5834506-55da8f2c44e481ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，只有顺时针旋转270°，即逆时针旋转90°，拍出来的图片才是正常的。
# 解决方法
既然 iPhone 会自动旋转拍出来的图片，那我们把它旋转回去不就得了，看下面这张图：
![](https://upload-images.jianshu.io/upload_images/5834506-5d20d6e5c782e367.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们先不管数字是什么意思，可以看到，数字标1的图片就是逆时针旋转90°拍出来的照片，是一个正的 ‘F’，而数字标6的图片就是我们竖着拍照拍出来的照片，数字标3的图片是我们顺时针旋转90°拍出来的照片，数字标8的图片就是我们倒着拍出来的照片，和我们实际拍出来的照片是一样的。

从上面我们知道了如何去矫正图片，如果是数字标6，那我们就顺时针旋转图片90°，如果是数字标8，那我们就逆时针旋转图片90°，那么怎么知道拍摄方向呢？这个时候就要介绍一下 `Exif.js` 这个库
>Exif.js 提供了 JavaScript 读取图像的原始数据的功能扩展，例如：拍照方向、相机设备型号、拍摄时间、ISO 感光度、GPS 地理位置等数据。

Gayhub地址：[Exif.js](https://github.com/exif-js/exif-js)

主要是用到了 `Exif.getData` 和 `Exif.getTag` 两个方法：
```
<template>
  <input ref="inputBoxRef" type="file" accept="Image/jpeg" @change="rotate">
</template>
<script>
import Exif from 'exif-js';
const file = this.inputRef.files[0];
Exif.getData(file, function () {
   let Orientation = Exif.getTag(this, 'Orientation');
   console.log('Ori', Orientation);
});
</script>
```
注意：这个 `Oreientation` 变量只有在用手机拍照的时候才会有值，除此之外都是 `undefined`，而且 `Exif.getData` 是个异步函数，之前就是把 `console.log` 放到外面了所以一直是 `undefined`。

现在通过 `Exif.getTag(this,'Orientation')` 就可以得到拍摄照片时的方向啦，它的值就是我们上面那张图的数字，我们关注左边那些数字，就是 1、8、3、6，接下来我们就要把它旋转。
# 旋转
我们可以用 `canvas` 的 `totate()` 方法：
```
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');
```
先创建一个 `canvas` 并获取上下文赋值给 `ctx`，让我们先看一下原理：
```
ctx.rotate(angle);
```
`rotate` 方法的参数为旋转弧度。需要将角度转为弧度：`degrees * Math.PI / 180`
旋转的中心点默认都在 `canvas` 的起点，即 ( 0, 0 )。旋转的原理如下图：
![](https://upload-images.jianshu.io/upload_images/5834506-a7a17c9648957187.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



旋转之后，如果从 ( 0, 0 ) 点进行 `drawImage()` ，那么画出来的位置就是在左图中的旋转 90° 后的位置，不在可视区域呢。旋转之后，坐标轴也跟着旋转了，想要显示在可视区域呢，需要将 ( 0, 0 ) 点往 `y` 轴的反方向移 `y` 个单位，此时的起始点则为 ( 0, -y )。

同理，可以获得旋转 -90° 后的起始点为 ( -x, 0 )，旋转 180° 后的起始点为 ( -x, -y )。
先看一下全部代码：
```
export default function rotateImage(img) {
  const width = img.width;
  const height = img.height;

  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');

  let newImg = new Image();
  let imageData;
  Exif.getData(img, function () {
    const orientation = Exif.getTag(this, 'Orientation');
    console.log('orientation', orientation);
    switch (orientation) {
      case 1:
        newImg = img;
        break;
      case 6:

        canvas.height = width;
        canvas.width = height;
        ctx.rotate(Math.PI / 2);
        ctx.translate(0, -height);

        ctx.drawImage(img, 0, 0);
        imageData = canvas.toDataURL('Image/jpeg', 1);
        newImg.src = imageData;

        break;
      case 3:
        canvas.height = height;
        canvas.width = width;
        ctx.rotate(Math.PI);
        ctx.translate(-width, -height);

        ctx.drawImage(img, 0, 0);
        imageData = canvas.toDataURL('Image/jpeg', 1);
        newImg.src = imageData;

        break;
      case 8:
        canvas.height = width;
        canvas.width = height;
        ctx.rotate(-Math.PI / 2);
        ctx.translate(-height, 0);

        ctx.drawImage(img, 0, 0);
        imageData = canvas.toDataURL('Image/jpeg', 1);
        newImg.src = imageData;

        break;
      case undefined:
        newImg = img;
        break;
      default:
        console.error('wrong orientation');
    }
  });
  return newImg;
}
```


我们看一下 `case 6`，因为 iPhone 竖着拍照时 `Orientation` 的值就是6：



```
        canvas.height = width;
        canvas.width = height;
        ctx.rotate(Math.PI / 2);
        ctx.translate(0, -height);

        ctx.drawImage(img, 0, 0);
        imageData = canvas.toDataURL('Image/jpeg', 1);
        newImg.src = imageData;
```


首先，`width` 和 `height` 都是拍摄图片的宽高，把他们分别赋值给 `canvas.height` 和 `canvas.width`，就能达到这种效果：

![](https://upload-images.jianshu.io/upload_images/5834506-40dfe9c2e6d90f3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第一步矫正已经完成，接下来就是要将渲染上下文 `context` 旋转，由上图可知，如果我们 `rotate` 之后，坐标轴会旋转，而且 `context` 都在不可见区域，所以我们要往 `y` 轴方向平移 `-height` 个单位，然后我们调用 `drawImage` 方法把我们拍的照片画上去，下面两行代码只是为了显示矫正完的图片，依次类推，我们就可以把 iPhone 各个方向拍的照片都矫正过来了：



![](https://upload-images.jianshu.io/upload_images/5834506-c808e95944f43ef5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



注意：只有 `jpg` 格式的图片才有 `exif` 信息，才能够获取 `orientation` ，所以第一步应该判断是不是 `jpg` 格式的图片。
![](https://upload-images.jianshu.io/upload_images/5834506-6e7c20750fff1166.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>参考：
>[js上传图片处理：压缩，旋转校正图片](https://blog.csdn.net/qq_25237107/article/details/69292374)
>[移动端图片上传旋转、压缩的解决方案](https://zhuanlan.zhihu.com/p/27627436)
>[stackoverflow](https://stackoverflow.com/questions/7584794/accessing-jpeg-exif-rotation-data-in-javascript-on-the-client-side)