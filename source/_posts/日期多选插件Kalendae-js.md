---
title: 日期多选插件Kalendae.js
tags: 
 - 博客迁移
 - JavaScript
urlname: use-Kalendaejs
date: 2016-10-09 16:07:17
---

>在项目中要实现日期多选的功能，于是在网上找到Kalendae.js，此文主要记录本人对于Kalendae.js的一些用法，以便以后查阅，希望对读者也有所帮助

## Kalendae.js一句话介绍
>[Kalendae.js](https://github.com/ChiperSoft/Kalendae)是一款强大的日期多控件（插件），支持日期的单选、**日期的多选**、**日期的范围选择**
>[案例](http://chipersoft.github.com/Kalendae/)

----------
## Kalendae.js如何使用

### 下载
（[Github下载](https://github.com/ChiperSoft/Kalendae/archive/master.zip)|[csdn下载](http://download.csdn.net/detail/mrhuangxiutao/9648268)）下载`Kalendae.js`相关的资源，解压后将`build`目录下的`js`和`css`拷贝到项目相应的资源文件夹下面，在需要使用日期多选的页面引入`js`和`css`就行了：
![解压文件目录](http://img.blog.csdn.net/20161009111814543)

```HTML
<link rel="stylesheet" href="./build/kalendae.css" type="text/css">
<script type='text/javascript' src='./build/kalendae.standalone.js'></script>
<!-- 这里不引入min.js是因为在后面要修改js -->
```
### 在页面使用：新建一个demo.html
①直接使用：
```
<!-- 单选 -->
<div class="auto-kal"></div>
<!-- 多选 -->
<div class="auto-kal" data-kal="mode:'multiple'"></div>
```

完整代码：

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<link rel="stylesheet" href="./build/kalendae.css" type="text/css">
    <script type='text/javascript' src='./build/kalendae.standalone.js'></script>
	<title>KalendaeDemo</title>
</head>
<body>
	<p>1直接展示（单选）</p>
	<div class="auto-kal"></div>
	<p>1.1直接展示（多选）</p>
	<div class="auto-kal" data-kal="mode:'multiple'"></div>
</body>
</html>
```
②输入框使用

```
<!-- 单选 -->
<input type="text" class="auto-kal">
<!-- 多选 -->
<input type="text" class="auto-kal" data-kal="mode:'multiple'">
```
完整代码：

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<link rel="stylesheet" href="./build/kalendae.css" type="text/css">
    <script type='text/javascript' src='./build/kalendae.standalone.js'></script>
	<title>KalendaeDemo</title>
</head>
<body>
	<p>2输入框使用（单选）：</p>
	<input type="text" class="auto-kal">
	<p>2输入框使用（多选）：</p>
	<input type="text" class="auto-kal" data-kal="mode:'multiple'" style="width: 100%;">
</body>
</html>
```
demo.html完整代码：

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<link rel="stylesheet" href="./build/kalendae.css" type="text/css">
    <script type='text/javascript' src='./build/kalendae.standalone.js'></script>
	<title>KalendaeDemo</title>
</head>
<body>
	<p>1直接展示（单选）</p>
	<div class="auto-kal"></div>
	<p>1.1直接展示（多选）</p>
	<div class="auto-kal" data-kal="mode:'multiple'"></div>
	<p>2输入框使用（单选）：</p>
	<input type="text" class="auto-kal">
	<p>2输入框使用（多选）：</p>
	<input type="text" class="auto-kal" data-kal="mode:'multiple'" style="width: 100%;">
</body>
</html>
```
显示效果：
![案例展示1](http://img.blog.csdn.net/20161009140916941)
![案例展示2](http://img.blog.csdn.net/20161009140945161)

----------


## Kalendae.js的个性化配置
### 日期中文显示
默认显示的样式是英文的，用户不友好，可以在`kalendae.standalone.js`里面进行编辑，设置`Locale.prototype`
#### 修改月份显示效果：

```
/*修改_months属性和_monthsShort属性*/
_months : '1月_2月_3月_4月_5月_6月_7月_8月_9月_10月_11月_12月'.split('_')
_monthsShort : '1月_2月_3月_4月_5月_6月_7月_8月_9月_10月_11月_12月'.split('_')
```
最终是这样子的：

```
/*_months : 'January_February_March_April_May_June_July_August_September_October_November_December'.split('_'),*/
_months : '1月_2月_3月_4月_5月_6月_7月_8月_9月_10月_11月_12月'.split('_'),
months : function (m) {
    return this._months[m.month()];
},
/*_monthsShort : 'Jan_Feb_Mar_Apr_May_Jun_Jul_Aug_Sep_Oct_Nov_Dec'.split('_'),*/
_monthsShort : '1月_2月_3月_4月_5月_6月_7月_8月_9月_10月_11月_12月'.split('_'),
monthsShort : function (m) {
    return this._monthsShort[m.month()];
},
```
#### 修改“星期”显示效果：
修改`_weekdays` 、`_weekdaysShort` 和`_weekdaysMin `
最终代码：

```
//星期显示样式
_weekdays : '星期日_星期一_星期二_星期三_星期四_星期五_星期六'.split('_'),
weekdays : function (m) {
    return this._weekdays[m.day()];
},

_weekdaysShort : '周日_周一_周二_周三_周四_周五_周六'.split('_'),
weekdaysShort : function (m) {
    return this._weekdaysShort[m.day()];
},

_weekdaysMin : '日_一_二_三_四_五_六'.split('_'),
weekdaysMin : function (m) {
    return this._weekdaysMin[m.day()];
},
```
#### 修改年月显示效果：
修改`Kalendae.prototype`的`titleFormat`
```
titleFormat:'MMMM, YYYY年',
```
最终效果：
![最终效果](http://img.blog.csdn.net/20161009144029611)
### 指定的div使用Kalendae
前面都是通过指定`class`来使用`kalendae`，可以通过`js`代码指定容器使用`kalendae`。

```
<div  id="datepk"></div>
<script type="text/javascript">
	/*使用方式：
	new Kalendae(指定容器,配置);
	配置属性注解：
	months属性表示日历显示几个月，值：1、2、3.....；默认值：1
	mode属性表示显示的是单选还是多选还是范围，值：'single'、'multiple'、'range'；默认值：'single'
	subscribe属性表示绑定kalendea指定的事件，支持的事件有change、date-clicked、view-changed
	*/
	new Kalendae(document.getElementById("datepk"), {
			months:1,
			mode:'multiple',
			subscribe: {
			       'date-clicked': function (date) {
			           console.log(date._i, this.getSelected());
			       }
			   }
		});
</script>
```
完整代码：

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<link rel="stylesheet" href="./build/kalendae.css" type="text/css">
    <script type='text/javascript' src='./build/kalendae.standalone.js'></script>
	<title>KalendaeDemo</title>
</head>
<body>
	<p>3指定div使用（多选）：</p>
	<div  id="datepk" style="width: 100%;"></div>
</body>
<script type="text/javascript">
	new Kalendae(document.getElementById("datepk"), {
			months:1,
			mode:'multiple',
			subscribe: {
			       'date-clicked': function (date) {
			           console.log(date._i, this.getSelected());
			       }
			   }
		});
</script>
</html>
```
### 修改控件显示的大小
（问题：当页面很小的时候布局会乱......）
修改`kalendae.css`
设置`.kalendae` ` .k-calendar`的`width`为100%;

```
.kalendae .k-calendar {
	display: inline-block;
	zoom:1;
	*display:inline;
	/* width改为100% width:155px;*/ 
	width: 100%; 
	vertical-align:top;
}
```

设置`.kalendae` ` .k-title`,
`.kalendae`  `.k-header`,
`.kalendae`  `.k-days` 的`width`为100%；

```
.kalendae .k-title,
.kalendae .k-header,
.kalendae .k-days {
	/* width改为100% */
	/* width:154px; */
	width: 100%;
	display:block;
	overflow:hidden;
}
```
`.kalendae` `.k-header` `span` 的`width`为12.8%；

```
.kalendae .k-header span {
	text-align:center;
	font-weight:bold;
	/* 这里的width要和.kalendae .k-days span 里面的相等 */
	width:12.8%;
	padding:1px 0;
	color:#666;
}
```

`.kalendae` `.k-days` `span` 的`width`为12.8%；

```
.kalendae .k-days span {
	/* 水平居中 */
	text-align:center;
	width:12.8%;
	/* 高度 4.5em效果比较好 height等于line-height就能垂直居中了 */
	height:4.5em;
	line-height:4.5em;
	padding:2px 3px 2px 2px;
	border:1px solid transparent;
	border-radius:3px;
	color:#999;
}
```
`.kalendae` `.k-header` `span` 和`.kalendae` `.k-days` `span`的`width`要相等

设置文字显示的样式：
```
.kalendae .k-days span {
	/* 水平居中 */
	text-align:center;
	width:12.8%;
	/* 高度 4.5em效果比较好 height等于line-height就能垂直居中了 */
	height:4.5em;
	line-height:4.5em;
	padding:2px 3px 2px 2px;
	border:1px solid transparent;
	border-radius:3px;
	color:#999;
}
```
最终效果：
![最终效果1](http://img.blog.csdn.net/20161009155341300)
![最终效果2](http://img.blog.csdn.net/20161009155403303)

个性化配置的`css`和`js` demo[代码下载](http://download.csdn.net/detail/mrhuangxiutao/9648670)

<hr/>

很老的文章了。。。:older_man: