---
title: 前端导出excel
date: 2018-07-11 23:07:56
tags: javascript
categories: 前端
---

# 前言
之前导出 Excel 的工作是交给后端来做的，后来才发现前端也可以导出 Excel，还是太年轻了，接下来一起看一下怎么做的。
# 前期准备
我们用 Vue，可以很容易生成一个表格：
```
<template>
  <div>
    <table border="1" ref="table">
      <thead>
        <tr>
          <th v-for="(head, index) in thead" :key="index">{{head}}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in rows" :key="index">
          <td v-for="(item, idx) in row" :key="idx">{{item}}</td>
        </tr>
      </tbody>
    </table>
    <a id="hrefToExportTable" style="postion: absolute;left: -10px;top: -10px;width: 0px;height: 0px;"></a>
    <button @click="download">下载</button>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      thead: ['姓名','学号','性别','年龄'],
      rows: [
        ['小明','1','男','18'],
        ['小红','2','女','17'],
        ['小白','3','男','19'],
      ],
    };
  }
};
</script>

<style>
</style>

```
注意：表格的结构和我们之后要导出 Excel 之前的解析有关，所以要按照格式来。
效果：
![](https://upload-images.jianshu.io/upload_images/5834506-9682a45af7fc7797.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

就是一个表格，一个按钮还有一个看不见的链接。当我们点击按钮，其实就是点击链接，我们只要把链接的 `href` 换成base64，就可以下载 Excel，功能我们之后再补充

# 导出 Excel
## 分辨浏览器
我们在 `src` 下新建文件夹 `libs`，然后新建文件 `utils.js` （个人习惯），然后我们一步一步分析。
首先，不同的浏览器导出 Excel 的方式不一样， IE 是用 `ActiveXObject` 用于启动创建对象的应用程序，在创建某个对象后，可在代码中使用已定义的对象变量引用该对象。但是其他主流浏览器就是用 `base64` 的方式下载的，所以第一步就是要判断现在的浏览器是什么。
```
function getExplorer () {
    var explorer = window.navigator.userAgent;
    if (explorer.indexOf('MSIE') >= 0) {
        // ie
        return 'ie';
    } else if (explorer.indexOf('Firefox') >= 0) {
        // firefox
        return 'Firefox';
    } else if (explorer.indexOf('Chrome') >= 0) {
        // Chrome
        return 'Chrome';
    } else if (explorer.indexOf('Opera') >= 0) {
        // Opera
        return 'Opera';
    } else if (explorer.indexOf('Safari') >= 0) {
        // Safari
        return 'Safari';
    };
};
```
我们用 `window.navigator.userAgent` 来识别浏览器，
IE11的 `userAgent` 是 `Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E)`，
Chrome 的 `userAgent` 是 `Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36`
特定的浏览器有特定的标识符，所以第一步判断浏览器就完成了，其实，只是为了分成两类：IE 和 非 IE 。。。
## 转换成 Excel
```
function tranform (table, aId, name) {
    let tableHead = table.children[0];
    let tableBody = table.children[1];
    let tableInnerHTML = '<thead><tr>';
    if (table.children.length !== 1) {
        let len = tableBody.rows.length;
        let i = -1;
        while (i < len) {
            if (i === -1) {
                Array.from(tableHead.rows[0].children).forEach((th) => {
                    tableInnerHTML = tableInnerHTML + '<th>' + th.innerHTML + '</th>';
                });
                tableInnerHTML += '</tr><thead><tbody>';
            } else {
                tableInnerHTML += '<tr>';
                Array.from(tableBody.rows[i].children).forEach((td) => {
                    tableInnerHTML = tableInnerHTML + '<td>' + td.innerHTML + '</td>';
                });
                tableInnerHTML += '</tr>';
            }
            i++;
        }
        tableInnerHTML += '</tbody>';
    }

    if (getExplorer() !== 'Safari' && name.substr(-1, 4) !== '.xls') {
        name += '.xls';
    }

    if (getExplorer() === 'ie') {
        var curTbl = table;
        var oXL = new ActiveXObject('Excel.Application');
        var oWB = oXL.Workbooks.Add();
        var xlsheet = oWB.Worksheets(1);
        var sel = document.body.createTextRange();
        sel.moveToElementText(curTbl);
        sel.select();
        sel.execCommand('Copy');
        xlsheet.Paste();
        oXL.Visible = true;

        try {
            var fname = oXL.Application.GetSaveAsFilename('Excel.xls', 'Excel Spreadsheets (*.xls), *.xls');
        } catch (e) {
            print('Nested catch caught ' + e);
        } finally {
            oWB.SaveAs(fname);
            // oWB.Close(savechanges = false);
            oXL.Quit();
            oXL = null;
            idTmr = setInterval(Cleanup(), 1);
        }
    } else {
        tableToExcel(tableInnerHTML, aId, name);
    }
}
function Cleanup () {
    window.clearInterval(idTmr);
    // CollectGarbage();
}
let tableToExcel = (function () {
    let uri = 'data:application/vnd.ms-excel;base64,';
    let template = '<html><head><meta charset="UTF-8"></head><body><table>{table}</table></body></html>';
    let base64 = function (s) { return window.btoa(unescape(encodeURIComponent(s))); };
    let format = function (s, c) {
        return s.replace(/{(\w+)}/g, function (m, p) { return c[p]; });
    };
    return function (table, aId, name) {
        let ctx = {worksheet: name || 'Worksheet', table: table};
        document.getElementById(aId).href = uri + base64(format(template, ctx));
        document.getElementById(aId).download = name;
        document.getElementById(aId).click();
    };
})();
```
我们一段一段讲：
```
function tranform (table, aId, name) {
    let tableHead = table.children[0];
    let tableBody = table.children[1];
    let tableInnerHTML = '<thead><tr>';
    if (table.children.length !== 1) {
        let len = tableBody.rows.length;
        let i = -1;
        while (i < len) {
            if (i === -1) {
                Array.from(tableHead.rows[0].children).forEach((th) => {
                    tableInnerHTML = tableInnerHTML + '<th>' + th.innerHTML + '</th>';
                });
                tableInnerHTML += '</tr><thead><tbody>';
            } else {
                tableInnerHTML += '<tr>';
                Array.from(tableBody.rows[i].children).forEach((td) => {
                    tableInnerHTML = tableInnerHTML + '<td>' + td.innerHTML + '</td>';
                });
                tableInnerHTML += '</tr>';
            }
            i++;
        }
        tableInnerHTML += '</tbody>';
    }
}
```
函数 `transform` 传入三个参数，第一个就是 `table` 的 DOM 节点，第二个是下载按钮的 id，第三个是 下载的文件名。
由变量名称可以知道是在获取表头和表格的元素，然后循环，将表格的 html 存到字符串 `tableInnerHTML` 里面，之后要用到，其实，在循环那里完全可以一把梭这么写：
```
    let tableInnerHTML = tableHead.outerHTML+tableBody.outerHTML;
```
效果是一样的，如果我们一开始表格的结构变了，那我们也要改变这条语句，而第一种方式就比较通用一点。

## IE处理
```
var idTmr;
if (getExplorer() === 'ie') {
        var curTbl = table;
        var oXL = new ActiveXObject('Excel.Application');
        var oWB = oXL.Workbooks.Add();
        var xlsheet = oWB.Worksheets(1);
        var sel = document.body.createTextRange();
        sel.moveToElementText(curTbl);
        sel.select();
        sel.execCommand('Copy');
        xlsheet.Paste();
        oXL.Visible = true;

        try {
            var fname = oXL.Application.GetSaveAsFilename('Excel.xls', 'Excel Spreadsheets (*.xls), *.xls');
        } catch (e) {
            print('Nested catch caught ' + e);
        } finally {
            oWB.SaveAs(fname);
            // oWB.Close(savechanges = false);
            oXL.Quit();
            oXL = null;
            idTmr = setInterval(Cleanup(), 1);
        }
    } else {
        tableToExcel(tableInnerHTML, aId, name);
    }
function Cleanup () {
    window.clearInterval(idTmr);
    // CollectGarbage();
}
```
其实就是调用 IE 的方法，生成 IE，这就不说了
## 其他浏览器处理
```
let tableToExcel = (function () {
    let uri = 'data:application/vnd.ms-excel;base64,';
    let template = '<html><head><meta charset="UTF-8"></head><body><table>{table}</table></body></html>';
    let base64 = function (s) { return window.btoa(unescape(encodeURIComponent(s))); };
    let format = function (s, c) {
        return s.replace(/{(\w+)}/g, function (m, p) { return c[p]; });
    };
    return function (table, aId, name) {
        let ctx = {worksheet: name || 'Worksheet', table: table};
        document.getElementById(aId).href = uri + base64(format(template, ctx));
        document.getElementById(aId).download = name;
        document.getElementById(aId).click();
    };
})();
```
一个立即执行函数，首先就是 `uri`，可以看到是 base64 的格式，然后 `template` 就是一个页面，里面的 `{table}` 就是我们上面生成的 `tableInnerHTML` ，用字符串的 `replace` 方法替换，然后把 `uri` 和 base64 连接，赋值给我们的链接，手动点击，就能下载了。
# 效果
 ![](https://upload-images.jianshu.io/upload_images/5834506-d769e5d4f16bb52a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5834506-338383b9468ef671.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，我们的文件已经下载下来了，由于是直接保存 html，所以 Excel 会警告，而且没有样式，如果想自己加样式，就在 `template` 那里自己写 `<style>` 样式。
# 拓展
有了 `table2excel` 那有没有 `table2img` 呢？当然有了，思路也是差不多，就是利用 `html2canvas` 这个插件，将 table 的 html 转成 canvas，然后 canvas 转成 图片。