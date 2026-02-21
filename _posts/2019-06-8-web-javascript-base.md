---
title: JavaScript 快速入门
date: 2019-06-8 10:01:47
tags: [Web,JavaScript]
categories: Web
---

# JavaScript
概念：运行在客户端浏览器中的一门客户端脚本语言。每一个浏览器都有 JavaScript 的解析引擎，不需要编译，直接就可以被浏览器解析执行。
功能：可以来增强用户和 html 页面的交互过程，可以来控制 html 元素，让页面有一些动态的效果，增强用户的体验。


## JavaScript 发展史
1. 1992年，Nombase公司，开发出第一门客户端脚本语言，专门用于表单的校验。命名为 ： C- -	，后来更名为：ScriptEase
2. 1995年，Netscape(网景)公司，开发了一门客户端脚本语言：LiveScript。后来，请来SUN公司的专家，修改LiveScript，命名为JavaScript
3. 1996年，微软抄袭JavaScript开发出jscript语言
4. 1997年，ECMA(欧洲计算机制造商协会)，制定出客户端脚本语言的标准：ECMAScript，就是统一了所有客户端脚本语言的编码方式。

「注」：JavaScript = ECMAScript + JavaScript自己特有的东西(BOM+DOM)

# ECMAScript
客户端脚本语言的标准
## 基本语法
### 与 html 结合方式
1. 内部js： 
    定义`<script>` 标签体内容就是js代码
2. 外部js：
    定义`<script>` 通过src属性引入外部的js文件
3. 注意：
    1. `<script>` 可以定义在html页面的任何地方。但是定义的位置会影响执行顺序。
    2. `<script>` 可以定义多个。

### 注释
1. 单行注释：//注释内容
2. 多行注释：`/*注释内容*/`

### 数据类型
1. 原始数据类型(基本数据类型)：
`number`：数字。 整数/小数/ NaN (not a number 一个不是数字的数字类型)
`string`：字符串。 字符串  "abc" "a" 'abc'
`boolean`： true 和 false
`null`：一个对象为空的占位符
`undefined`：未定义。如果一个变量没有给初始化值，则会被默认赋值为 undefined
    
2. 引用数据类型：对象

### 变量
* 变量：一小块存储数据的内存空间
* Java 语言是强类型语言，而 JavaScript 是弱类型语言。
    强类型：在开辟变量存储空间时，定义了空间将来存储的数据的数据类型。只能存储固定类型的数据
    弱类型：在开辟变量存储空间时，不定义空间将来的存储数据类型，可以存放任意类型的数据。
* 语法：
```javascript 
    var 变量名 = 初始化值;
```
* `typeof` 运算符：获取变量的类型。
    注：`null`运算后得到的是`object`

### 运算符
1. 一元运算符：只有一个运算数的运算符，`++`，`--` ， `+`(正号)  
    * `++` `--`： 自增(自减)
    * `+(-)`：正负号
    * 注意：在 js 中，如果运算数不是运算符所要求的类型，那么 js 引擎会自动的将运算数进行类型转换

2. 算数运算符：`+` `-` `*` `/` `%` ...
3. 赋值运算符：`=` `+=` `-+` ...
4. 比较运算符：`>` `<` `>=` `<=` `==`  `===(全等于)`
    1. 类型相同：按照字典顺序直接比较。按位逐一比较，直到得出大小为止。
    2. 类型不同：先进行类型转换，再比较
    3. `===`：全等于。在比较之前，先判断类型，如果类型不一样，则直接返回 false
5. 逻辑运算符：`&&`  `||`  `!`
6. 三元运算符：`表达式? 值1 : 值2;`
```javascript
var a = 3;
var b = 4;
var c = a > b ? 1 : 0;
```
6. 其他类型转 `number`：
    * string 转 number：按照字面值转换。如果字面值不是数字，则转为NaN（不是数字的数字）
    * boolean 转 number：true转为1，false转为0

7. 其他类型转 `boolean`：
    1. number：0或NaN为假，其他为真
    2. string：除了空字符串("")，其他都是true
    3. null&undefined：都是false
    4. 对象：所有对象都为true
    
### 流程控制语句
1. if...else...
2. switch：在java中，switch语句可以接受的数据类型： byte int shor char。在js中,switch语句可以接受`任意`的`原始`数据类型
```javascript
    switch(变量)：
     case 值：
```
「注」：Java Switch中：枚举(1.5) ,String(1.7)
3. while
4. do...while
5. for

### js特殊语法
1. 语句以`;`结尾，如果一行只有一条语句则 `;`可以省略 (不建议)
2. 变量的定义使用var关键字，也可以不使用
    **用**： 定义的变量是`局部`变量
    **不用**：定义的变量是`全局`变量(不建议)

### 练习：99乘法表
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>99乘法表</title>
    <style>
        td{
            border： 1px solid;
        }
    </style>

    <script>
        document.write("<table  align='center'>");
        //1.完成基本的for循环嵌套，展示乘法表
        for (var i = 1; i <= 9 ; i++) {
            document.write("<tr>");
            for (var j = 1; j <=i ; j++) {
                document.write("<td>");
                //输出  1 * 1 = 1
                document.write(i + " * " + j + " = " + ( i*j) +"&nbsp;&nbsp;&nbsp;");
                document.write("</td>");
            }
            /*//输出换行
            document.write("<br>");*/
            document.write("</tr>");
        }
        //2.完成表格嵌套
        document.write("</table>");
    </script>
</head>
<body>
</body>
</html>
```

## 基本对象
### Function
函数(方法)对象
1. 创建：
```javascript
    //第一种方式
    var fun = new Function(形式参数列表,方法体);  //不建议使用
    //第二种方式
    function 方法名称(形式参数列表){
            方法体
    }
    //第三种方式
    var 方法名 = function(形式参数列表){
            方法体
}
```
2. 属性：
    `length`： 代表形参的个数
3. 特点：
    1. 方法定义是，形参的类型不用写,返回值类型也不写。
    2. 方法是一个对象，如果定义名称相同的方法，会覆盖
    3. 在js中，方法的调用只与方法的名称有关，和参数列表无关
    4. 在方法声明中有一个隐藏的内置对象（数组）：`arguments`，封装所有的实际参数
4. 调用：
    方法名称(实际参数列表);

### Array
数组对象
1. 创建：
```javascript
    var arr = new Array(元素列表);
    var arr = new Array(默认长度);
    var arr = [元素列表];
```
2. 方法
    `join(参数)`：将数组中的元素按照指定的分隔符拼接为字符串
    `push()`：向数组的末尾添加一个或更多元素，并返回新的长度。
3. 属性
    `length`：数组的长度
4. 特点：js中，数组`元素的类型`是可变的,`长度`是可变的。

### Boolean
### Date
日期对象
1. 创建：
```javascript
    var date = new Date();
```

2. 方法：
    `toLocaleString()`：返回当前date对象对应的时间本地字符串格式
    `getTime()`：获取毫秒值。返回当前如期对象描述的时间到1970年1月1日零点的毫秒值差

### Math
数学对象
1. 创建：Math对象不用创建，直接使用。  `Math.方法名();`
2. 方法：
`random()`：返回 0 ~ 1 之间的随机数。 含0不含1
`ceil(x)`：对数进行上舍入。
`floor(x)`：对数进行下舍入。
`round(x)`：把数四舍五入为最接近的整数。
3. 属性： PI , ...

### Number
### String
### RegExp
- 正则表达式，定义字符串的组成规则。
    1. 单个字符：`[]`
        如： [a] [ab] [a-zA-Z0-9_]
        特殊符号代表特殊含义的单个字符：
        `\d`：单个数字字符 [0-9]
        `\w`：单个单词字符[a-zA-Z0-9_]
    2. 量词符号：
        `?`：表示出现0次或1次
        `*`：表示出现0次或多次
        `+`：出现1次或多次
        `{m,n}`：表示 m<= 数量 <= n
            * m如果缺省： {,n}：最多n次
            * n如果缺省：{m,} 最少m次
    3. 开始结束符号
        `^`：开始
        `$`：结束
- 正则对象：
    1. 创建
    ```javascript
       //第一种方式
       var reg = new RegExp("正则表达式");
       //第二种方式
       var reg = /正则表达式/;
    ```
    2. 方法	
    `test(参数)`：验证指定的字符串是否符合正则定义的规范	

### Global
1. 特点：全局对象，这个 Global 中封装的方法不需要对象就可以直接调用。  `方法名();`
2. 方法：
    `encodeURI()`：url编码
    `decodeURI(`)：url解码

    `encodeURIComponent()`：url编码,编码的字符更多
    `decodeURIComponent()`：url解码

    `parseInt()`：将字符串转为数字
        * 逐一判断每一个字符是否是数字，直到不是数字为止，将前边数字部分转为number
    `isNaN()`：判断一个值是否是NaN
        * NaN六亲不认，连自己都不认。NaN参与的==比较全部为false

    `eval()`：将 JavaScript 字符串，并把它作为脚本代码来执行。
3. URL编码：XXXX =  %E4%BX%A2%Ea%95%BA%E5%92%AD%E3%A2%A3

# 事件
**概念**：某些组件被执行了某些操作后，触发某些代码的执行。	
* 事件：某些操作。如： 单击，双击，键盘按下了，鼠标移动了
* 事件源：组件。如： 按钮 文本输入框...
* 监听器：代码。
* 注册监听：将事件，事件源，监听器结合在一起。 当事件源上发生了某个事件，则触发执行某个监听器代码。

## 常见的事件
### 点击事件
1. `onclick` ：单击事件
2. `ondblclick` ：双击事件
eg：通过 js 获取元素对象，指定事件属性，设置一个函数
```javascript
<body>
    <img id="light" src="img/off.gif"  onclick="fun();">
    <img id="light2" src="img/off.gif">
    
    <script>
        function fun(){
            alert('被点击');
        }
    
        function fun2(){
            alert('被点击2');
        }
        //1.获取light2对象
        var light2 = document.getElementById("light2");
        //2.绑定事件
        light2.onclick = fun2;
    </script>
</body>
```
### 焦点事件
1. `onblur` ：失去焦点
2. `onfocus` ：元素获得焦点。

### 加载事件
1. `onload` ：一张页面或一幅图像完成加载。

### 鼠标事件
1. `onmousedown`	鼠标按钮被按下。
2. `onmouseup`	鼠标按键被松开。
3. `onmousemove`	鼠标被移动。
4. `onmouseover`	鼠标移到某元素之上。
5. `onmouseout`	鼠标从某元素移开。

    
### 键盘事件
1. `onkeydown`	某个键盘按键被按下。	
2. `onkeyup`		某个键盘按键被松开。
3. `onkeypress`	某个键盘按键被按下并松开。

### 选择和改变
1. `onchange`	域的内容被改变。
2. `onselect`	文本被选中。

### 表单事件
1. `onsubmit`	确认按钮被点击。
2. `onreset`	重置按钮被点击。

# BOM
## 概念
Browser Object Model 浏览器对象模型，将浏览器的各个组成部分封装成对象。
## 组成
* `Window` ：窗口对象
* `Navigator` ：浏览器对象
* `Screen` ：显示器屏幕对象
* `History` ：历史记录对象
* `Location` ：地址栏对象

### Window
1. 创建：直接可以`window.方法名()`，window 关键字可省略
2. 方法
    1. 与弹出框有关的方法：
    `alert()`	显示带有一段消息和一个确认按钮的警告框。
    `confirm()`	显示带有一段消息以及确认按钮和取消按钮的对话框，有返回值
    `prompt()`	显示可提示用户输入的对话框。返回值：获取用户输入的值
    2. 与打开关闭有关的方法：
    `close()`	关闭浏览器窗口。谁调用我 ，我关谁
    `open()`	打开一个新的浏览器窗口，返回新的Window对象
    3. 与定时器有关的方式
    `setTimeout()`	在指定的毫秒数后调用函数或计算表达式。
    ```bash
    参数：
        1. js代码或者方法对象
        2. 毫秒值
    返回值：唯一标识，用于取消定时器
    ```
    `clearTimeout()`	取消由 setTimeout() 方法设置的 timeout。
    `setInterval()`	按照指定的周期（以毫秒计）来调用函数或计算表达式。
    `clearInterval()`	取消由 setInterval() 设置的 timeout。

3. 属性
- 获取其他BOM对象：
    `history`
    `location`
    `Navigator`
    `Screen`
- 获取DOM对象
    `document`
4. 特点
Window 对象不需要创建可以直接使用 `window` 使用。 `window.方法名();`
window 引用可以省略。 `方法名();`

### Location
1. 创建(获取)
    1. `window.location`
    2. `location`

2. 方法：
`reload()`	重新加载当前文档。刷新
3. 属性
`href`	设置或返回完整的 URL。

### History
1. 创建(获取)
    1. `window.history`
    2. `history`

2. 方法
`back()`	加载 history 列表中的前一个 URL。
`forward()`	加载 history 列表中的下一个 URL。
`go(参数)`	加载 history 列表中的某个具体页面。
```bash
    参数：
        正数：前进几个历史记录
        负数：后退几个历史记录
```
3. 属性
    `length` 返回当前窗口历史列表中的 URL 数量。

### Screen
### Navigator

# DOM
## 概念
Document Object Model 文档对象模型，将标记语言文档的各个组成部分，封装为对象。可以使用这些对象，对标记语言文档进行 CRUD 的动态操作。
![dom-tree-elem](/assets/img/tech/web-dom-tree-elem.png)

## DOM分类
W3C DOM 标准被分为 3 个不同的部分

### 核心 DOM 
针对任何结构化文档的标准模型

#### Document：文档对象
1. 创建(获取)：在html dom 模型中可以使用 window 对象来获取
    1. `window.document`
    2. `document`
2. 方法：
- 获取 Element 对象：
    1. `getElementById()`	： 根据id属性值获取元素对象。id属性值一般唯一
    2. `getElementsByTagName()`：根据元素名称获取元素对象们。返回值是一个数组
    3. `getElementsByClassName()`：根据Class属性值获取元素对象们。返回值是一个数组
    4. `getElementsByName()`： 根据name属性值获取元素对象们。返回值是一个数组
- 创建其他DOM对象：
    `createAttribute(name)`
    `createComment()`
    `createElement()`
   ` createTextNode()`

#### Element：元素对象
1. 获取/创建：通过 document 来获取和创建
2. 方法：
    1. `removeAttribute()`：删除属性
    2. `setAttribute()`：设置属性

#### Node：节点对象
Node节点对象是其他5个的父对象。
* 特点：所有 dom 对象都可以被认为是一个节点
* 方法：
    * `appendChild()`：向节点的子节点列表的结尾添加新的子节点。
    * `removeChild()`	：删除（并返回）当前节点的指定子节点。
    * `replaceChild()`：用新节点替换一个子节点。
* 属性：
    * `parentNode` 返回节点的父节点。

#### Attribute：属性对象
#### Text：文本对象
#### Comment:注释对象

### HTML DOM 
针对 HTML 文档的标准模型
1. 标签体的设置和获取： `innerHTML`
2. 使用 html 元素对象的属性
3. 控制元素样式
    1. 使用元素的 `style` 属性来设置
    ```javascript
    //修改样式方式1
    div1.style.border = "1px solid red";
    div1.style.width = "200px";
    //font-size--> fontSize
    div1.style.fontSize = "20px";
    ```
    2. 提前定义好类选择器的样式，通过元素的 `className` 属性来设置其class属性值。

### XML DOM 
- 针对 XML 文档的标准模型

# 小结
js语法的使用基本类似java，只需在使用时随手查阅文档即可。要把握好 DOM 和 BOM 的区别，DOM管理浏览器的内容对象，BOM管理浏览器的控件对象。而 BOM 控制(获取) DOM。