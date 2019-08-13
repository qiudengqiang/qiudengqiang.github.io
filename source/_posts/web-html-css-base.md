---
title: HTML 和 CSS 基础
date: 2019-05-27 10:01:47
tags: [Web,HTML,CSS]
categories: Web
---

# HTML
hyper text markup language 超文本标记语言，是最基础的网页开发语言。网页文件后缀名以`.html`/`.htm`结束。
## 基本标签
### 文件标签
`<!DOCTYPE html>`：html5 中定义该文档类型是html文档
`<html>`：html 文档的根标签
`<head>`：头标签，用于指定 html 文档的一些属性，引入外部的资源。
`<title>`：标题标签
`<body>`：体标签

### 文本标签
`注释`：`<!-- 注释内容 -->`
`<h1> to <h6>`：标题标签，字体大小逐渐递增
`<p>`：段落标签
`<br>`：换行标签
`<hr>`：水平线,属性：color,width,size,align(center,left,right)
`<b>`：字体加粗
`<i>`：字体斜体
`<font>`：字体标签,属性：color,size,face
`<center>`：文本居中

### 属性定义
`color`:
    1. 英文单词：red,green,blue
    2. rgb(值1,值2,值3)：值的取值范围：0~255。 如rgb(0,0,255)
    3. #值1值2值3：值的范围：00~FF之间。如 #00FF00
`width`:
    1. 数值 width = '20'，数值的单位默认是px像素
    2. 数值%：占比相对于父元素的比例。

### 图片标签
`<img>`：图片标签，属性：src,alt,align,width,height
`相对路径`：以`.`开头的路径，eg：`./`代表当前路径，`../`代表上一级目录

### 列表标签
`<ol>`：有序列表外层标签
`<ul>`：无序列表外层标签
`<li>`：条目标签，包含属性 type

### 链接标签
`<a>`：超链接标签
属性：`href`：访问资源的 URL ,`target`：打开资源的方式(`_self`默认值,`_blank`)

### div和span
`<div>`：每一个 div 占满一行，块级标签
`<span>`：文本信息在一行展示，行内标签

### 语义化标签
语义化标签是 html5 之后出现的新特性，目的是为了提高程序的可读性
`<header>`：页眉
`<footer>`：页脚

### 表格标签
`<table>`：定义表格标签
属性：
width,border,bgcolor,align
cellpadding：单元格内容与单元格的距离
cellspacing：定义单元格之间的距离，如果指定为0，则单元格线会合并为一条
`<tr>`：定义行
`<td>`：定义单元格
`<th>`：定义表头单元格
`<caption>`：表格标题
`<thead>`：表格中表头内容，类似语义化标签，目的为增强代码可读性
`<tbody>`：表格表体内容
`<tfoot>`：表格脚注

## 表单标签
用户采集用户输入的数据，和服务器进行交互。

### 表单体标签
`<form>`：可以定义一个范围，范围代表采集用户数据的范围。
属性：
     action
     method：请求方式有7种，一般使用 get，post
     name：(不指定无法提交表单项中的数据)
### 表单项标签
`<input>`：可以通过 `type` 属性改变元素展示的样式。
`type`属性值：
    text：文本
    password：密码
    radio(`value`属性指定提交的值,`checked`指定默认值),
    checkbox(`value`属性指定提交的值,`checke`d指定默认值),
    placeholder：提示文字
    file：选择文件框
    hidden：隐藏域，用于提交一些信息。
    submit：提交按钮
    button：普通按钮
    image：图片提交按钮，`src`属性选择图片路径
    color：取色器
    date：日期选择
    datetime-local：带时分日期选择
    email：邮箱
    number：数字选择
`<label>`：指定输入项的文字描述信息，其`for`属性一般会和 input 的 id 属性对应。这样点击 label 后 input 则会获取到焦点。
`<select>`：下拉列表
`<option>`：下拉列表中的子选项,`value`属性用于指定提交的值
`<textarea>`：多行输入框，属性`rows`，`cols`用于指定行数和每行显示的字符数。

# CSS
## 概念
Cascading Style Sheets 层叠样式表，多个样式可作用在同一个 html 元素上，同时生效。
## 使用
### 内联样式
在标签内使用`style`属性指定 css 代码
eg：
```html
<div style="color:red;">hello css</div>
```
### 内部样式
在`head`标签内，定义`style`标签 style 标签体内容就是 css 代码
eg：
```html
<style>
    div{
        color:blue;
    }
</style>
<div>hello css</div>
```
### 外部样式
1.定义 css 资源文件
2.在`head`标签内，定义`link`标签，引入外部的资源文件
eg：
.css文件
```css
div{
 color:green;
}
```
.html文件
```html
<link rel="stylesheet" href="css/a.css">
<div>hello css</div>
```
### CSS语法
格式：
```css
选择器 {
    属性名1:属性值1;
    属性名1:属性值1;
    ...
}
```
**注意**：每一对属性需要使用`;`隔开，最后一对属性可以不加`;`

### 选择器
筛选具有相似特征的元素
- 基础选择器
1. id选择器：选择具体的 id 属性值的元素，建议在 html 页面中 id 值唯一；语法：`#id属性值{}`
2. 元素选择器：选择具有相同标签名称的元素；语法：`标签名称{}`，注意 id 选择器优先级高于元素选择器
3. 类选择器：选择具有相同的 class 属性值的元素`.class属性值{}`，注意类选择器优先级高于元素选择器，低于 id 选择器

- 扩展选择器
1. 选择素有元素；语法：`*{}`
2. 并集选择器；语法：`选择器1,选择器2{}`
3. 子选择器：筛选选择器1元素下的选择器2元素；语法：`选择器1 选择器2{}`
4. 父选择器：筛选选择器2的父元素选择器1；语法：`选择器1 > 选择器2{}`
5. 属性选择器：选择元素名称,属性名=属性值的元素；语法：`元素名称[属性名="属性值"]{}`
6. 伪类选择器：选择一些元素具有的状态；语法：`元素:状态{}`

### 属性
1. 字体、文本
`font-size`：字体大小
`color`：文本颜色
`text-align`：对齐方式
`line-height`：行高
2. 背景
`background`：背景，属性`url`可以指定图片路径
3. 边框
`border`：设置边框，复合属性
4. 尺寸
`width`：宽度
`height`：高度
5. 盒子模型：控制布局
`margin`：外边距
`padding`：内边距，默认情况下调整内边距会影响整个盒子的大小，这时可设置`box-sizing:border-box;`确定盒子的指定宽高为最终大小
`float`：left,right