---
title: jQuery & Ajax 快速入门
date: 2019-08-12 10:01:47
tags: [Web，jQuery，Ajax]
categories: Web
---
# jQuery
jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（或JavaScript框架）。jQuery设计的宗旨	是“write Less，Do More”，即倡导写更少的代码，做更多的事情。它封装JavaScript常用的功能代码，提供一种简便的JavaScript设计模式，优化HTML文档操作、事件处理、动画设计和Ajax交互。

## JavaScript 回顾
JavaScript核心 --- JavaScript 语言的核心，定义了语言中基本语法
DOM  --- Document Object Model 文档对象模型 w3c
BOM  --- Broswer Object Model 浏览器对象模型

## jQuery 基础
### 快速入门
1. 下载JQuery
#### 目前jQuery有三个大版本：
        1.x：兼容ie678,使用最为广泛的，官方只做BUG维护，
                功能不再新增。因此一般项目来说，使用1.x版本就可以了，
                最终版本：1.12.4 (2016年5月20日)
        2.x：不兼容ie678，很少有人使用，官方只做BUG维护，
                功能不再新增。如果不考虑兼容低版本的浏览器可以使用2.x，
                最终版本：2.2.4 (2016年5月20日)
        3.x：不兼容ie678，只支持最新的浏览器。除非特殊要求，
                一般不会使用3.x版本的，很多老的jQuery插件不支持这个版本。
                目前该版本是官方主要更新维护的版本。最新版本：3.2.1（2017年3月20日）
#### jquery-xxx.js 与 jquery-xxx.min.js区别：
        1. jquery-xxx.js：开发版本。给程序员使用，有良好的缩进和注释。体积大一些
        2. jquery-xxx.min.js：生产版本。程序中使用，没有缩进。体积小一些。程序加载更快
2. 引入jQuery
直接将 jQuery 核心 js 文件加入到工程，一般放置在 web 应用的非 WEB-INF 文件夹，在 html 页面中使用以下代码引入就可以在这个页面中使用 jQuery 了。
```html
<script src="jquery.js文件所在的位置"/>
```
3. 使用
```js
    var div1 = $("#div1");
    alert(div1.html());
```

### 选择器

#### 基础选择器
1. `#id` 
    用法:` $("#myDiv")`  ;  返回值 : 单个元素的组成的集合
    说明: 这个就是直接选择 html 中的`id="myDiv"`
2. `Element`
    用法: `$("div")` ;    返回值 : 集合元素
    说明: element 的英文翻译过来是「元素」，所以 element 其实就是 html 已经定义的标签元素，例如 div， input， a等等
3. `class`         
    用法: `$(".myClass")` ;     返回值 : 集合元素
    说明: 这个标签是直接选择 html 代码中`class="myClass"`的元素或元素组（因为在同一html页面中class是可以存在多个同样值的）
4. `*` 
    用法: `$("*")` ;     返回值 : 集合元素
    说明: 匹配所有元素，多用于结合上下文来搜索
5. `selector1, selector2, selectorN`   
    用法: `$("div,span,p.myClass")`;    返回值 : 集合元素
    说明: 将每一个选择器匹配到的元素合并后一起返回。可以指定任意多个选择器， 并将匹配到的元素合并到一个结果内。其中`p.myClass`是表示匹配元素

#### 层次选择器
6. `ancestor descendant`
    用法: `$("form input")` ;   返回值 : 集合元素
    说明: 在给定的祖先元素下匹配所有后代元素。这个要和下面的`parent > child`区分开.
7. `parent > child`
    用法: `$("form > input")` ;    返回值 : 集合元素
    说明: 在给定的父元素下匹配所有子元素。注意:要区分好后代元素与子元素
8. `prev + next`
    用法: `$("label + input")` ;   返回值 : 集合元素
    说明: 匹配所有紧接在 prev 元素后的 next 元素
9. `prev ~ siblings`
    用法: `$("form ~ input")` ;    返回值 : 集合元素
    说明: 匹配 prev 元素之后的所有 siblings 元素。
    注意: 是匹配之后的元素，不包含该元素在内，并且 siblings 匹配的是和 prev 同辈的元素，其后辈元素不被匹配。

#### 基本操作
1. 事件绑定
```js
    //1.获取b1按钮
    $("#b1").click(function(){
        alert("abc");
    });
```
  
2. 入口函数
```js
    //以下方法等同于window.onload
    $(function () {
    });
```
**[注意]**：`window.onload`  和 `$(function)` 区别
    * window.onload 只能定义一次,如果定义多次，后边的会将前边的覆盖掉
    * $(function)可以定义多次的。

3. 样式控制：css方法
```js
    // $("#div1").css("background-color","red");
    $("#div1").css("backgroundColor","pink");
```
### 过滤器
#### 基础过滤器
1. `:first`
    用法: `$("tr:first")` ;   返回值:  单个元素的组成的集合
    说明: 匹配找到的第一个元素
2. `:last`
    用法: `$("tr:last")`;  返回值  集合元素
    说明: 匹配找到的最后一个元素，与 `:first` 相对应
3. `:not(selector)`
    用法: `$("input:not(:checked)")`; 返回值:  集合元素
    说明: 去除所有与给定选择器匹配的元素。有点类似于「非」，意思是没有被选中的 input (当 input 的`type="checkbox"`).
4. `:even`
    用法: `$("tr:even")` ;  返回值:  集合元素
    说明: 匹配所有索引值为偶数的元素，从 0 开始计数。js的数组都是从0开始计数的。例如要选择table中的行，因为是从0开始计数，所以 table 中的第一个 tr 就为偶数0.
5.  `:odd`
    用法: `$("tr:odd")`; 返回值 : 集合元素
    说明: 匹配所有索引值为奇数的元素，和`:even`对应，从 0 开始计数.
6. `:eq(index)`
    用法: `$("tr:eq(0)")` ;   返回值:  集合元素
    说明: 匹配一个给定索引值的元素`.eq(0)`就是获取第一个 tr 元素。括号里面的是索引值，不是元素排列数
7. `:gt(index)`
    用法: `$("tr:gt(0)")`;   返回值 : 集合元素
    说明: 匹配所有大于给定索引值的元素.
8. `:lt(index)`
    用法: `$("tr:lt(2)")` ;   返回值 : 集合元素 
    说明: 匹配所有小于给定索引值的元素.
9. `:header(固定写法)`
    用法: `$(":header").css("background"， “#EEE")`;    返回值 : 集合元素
    说明: 获得标题（h1~h6）元素，固定写法
10. `:animated(固定写法)` ;  返回值 : 集合元素
    说明: 匹配所有正在执行动画效果的元素

#### 内容过滤器
1. `:contains(text)`
    用法: `$("div:contains('John')")`  ;  返回值 : 集合元素
    说明: 匹配包含给定文本的元素。这个选择器比较有用，当我们要选择的不是dom标签元素时，它就派上了用场了，它的作用是查找被标签「围」起来的文本内容是否符合指定的内容
2. `:empty`
    用法: `$("td:empty")` ;  返回值 : 集合元素
    说明: 匹配所有不包含子元素或者文本的空元素
3. `:has(selector)`
    用法: `$("div:has(p)").addClass("test")` ;  返回值 : 集合元素
    说明: 匹配含有选择器所匹配的元素的元素。这个解释需要好好琢磨，但是一旦看了使用的例子就完全清楚了，就是给所有包含 p 元素的 div标签加上`class="test"`
4. `:parent` 
    用法:`$("td:parent")` ; 返回值 : 集合元素
    说明: 匹配含有子元素或者文本的元素。注意:这里是`:parent`，可不是`.parent`。与上面的`:empty`形成反义

#### 可见度过滤器
1. `:hidden`
    用法: `$("tr:hidden")` ; 返回值 : 集合元素
    说明: 匹配所有的不可见元素，input 元素的 type 属性为 `hidden` 的话也会被匹配到；意思是 CSS 中`display:none`和`input type="hidden"`的都会被匹配到。同样，要彻底分清楚冒号`:`点号`.`和逗号`，`的区别
2. `:visible`
    用法: `$("tr:visible")`;  返回值 : 集合元素
    说明: 匹配所有的可见元素

#### 属性过滤器
1. `[attribute]`
    用法: `$("div[id]")` ;  返回值 : 集合元素
    说明: 匹配包含给定属性的元素。例如：选取所有带 id 属性的 div 标签
2. `[attribute=value]`
    用法: `$("input[name='newsletter']“).attr("checked"， true);`   ; 返回值 : 集合元素
    说明: 匹配给定的属性是某个特定值的元素。例如：选取所有 name 属性是 newsletter 的 input 元素.
3. `[attribute!=value]`
    用法: `$("input[name!='newsletter']").attr("checked"， true);` ;   返回值 : 集合元素
    说明: 匹配所有不含有指定的属性，或者属性不等于特定值的元素。此选择器等价于`:not([attr=value])`，要匹配含有特定属性但不等于特定值的元素，请使用`[attr]:not([attr=value])`。之前看到的 `:not` 派上了用场
4. `[attribute^=value]`
    用法: `$("input[name^='news']")` ; 返回值 : 集合元素
    说明: 匹配给定的属性是以某些值开始的元素
5. `[attribute$=value]`
    用法: `$("input[name$='letter']")` ; 返回值 : 集合元素
    说明: 匹配给定的属性是以某些值结尾的元素
6. `[attribute*=value]`
    用法: `$("input[name*='man']")` ;  返回值 : 集合元素
    说明: 匹配给定的属性是以包含某些值的元素
7. `[attributeFilter1][attributeFilter2][attributeFilterN]`
    用法: `$("input[id][name$='man']")` ; 返回值 : 集合元素
    说明: 复合属性选择器，需要同时满足多个条件时使用；又是一个组合，这种情况实际使用的时候很常用。这个例子中选择的是所有含有 id 属性，并且它的 name 属性是以 man 结尾的元素。

#### 子元素过滤器
1. `:nth-child(index/even/odd/equation)`
    用法: `$("ul li:nth-child(2)")`   返回值 : 集合元素
    说明: 匹配其父元素下的第N个子或奇偶元素。这个选择器和之前说的基础过滤(Basic Filters)中的 `eq()` 有些类似，不同的地方就是：前者是从 0 开始，后者是从 1 开始。

2. `:first-child`
    用法: `$("ul li:first-child") `;   返回值 : 集合元素
    说明: 匹配第一个子元素；`:first` 只匹配一个元素，而此选择符将为每个父元素匹配一个子元素，这里需要**特别**的记忆下区别

3. `:last-child`
    用法: `$("ul li:last-child")`  ;    返回值 : 集合元素
    说明: 匹配最后一个子元素；`:last`只匹配一个元素，而此选择符将为每个父元素匹配一个子元素


4. `:only-child`
    用法: `$("ul li:only-child")` ;  返回值 : 集合元素
    说明: 如果某个元素是父元素中唯一的子元素，那将会被匹配；如果父元素中含有其他元素，那将不会被匹配；意思就是:只有一个子元素的才会被匹配

#### 表单对象属性过滤器
1. `:enabled`
    用法: `$("input:enabled")` ;   返回值 : 集合元素
    说明: 匹配所有可用元素；意思是查找所有input中不带有`disabled="disabled"`的input；不为disabled，当然就为enabled
2. `:disabled`
    用法: `$("input:disabled")`  ;  返回值 : 集合元素
    说明: 匹配所有不可用元素。与上面的那个是相对应的
3. `:checked`
    用法: `$("input:checked")` ;  返回值 : 集合元素
    说明: 匹配所有选中的被选中元素(复选框、单选框等，不包括select中的option)
4. `:selected`
    用法: `$("select option:selected")` ;  返回值 : 集合元素
    说明: 匹配所有选中的option元素

#### 表单过滤器
1. `:input`
    用法:` $(":input")` ;   返回值 : 集合元素
    说明:匹配所有 input， textarea， select 和 button 元素
2. `:text`
    用法: `$(":text")` ;  返回值 : 集合元素
    说明: 匹配所有的单行文本框
3. `:password`
    用法: `$(":password")` ; 返回值 : 集合元素
    说明: 匹配所有密码框
4. `:radio`
    用法: `$(":radio")` ; 返回值 : 集合元素
    说明: 匹配所有单选按钮
5. `:checkbox`
    用法: `$(":checkbox")` ; 返回值 : 集合元素
    说明: 匹配所有复选框
6. `:submit`
    用法: `$(":submit")` ;   返回值 : 集合元素
    说明: 匹配所有提交按钮

### DOM 操作
#### 外部插入
`after(content)` :在每个匹配的元素之后插入内容
`before(content)`:在每个匹配的元素之前插入内容
`insertAfter(content)`:把所有匹配的元素插入到另一个、指定的元素元素集合的后面
`insertBefore(content)` :把所有匹配的元素插入到另一个、指定的元素元素集合的前面
#### 内部插入
`append(content)` :向每个匹配的元素的内部的结尾处追加内容
`appendTo(content)` :将每个匹配的元素追加到指定的元素中的内部结尾处
`prepend(content)`:向每个匹配的元素的内部的开始处插入内容
`prependTo(content)` :将每个匹配的元素插入到指定的元素内部的开始处
#### 查找节点
选择器/过滤器 可以实现查找节点
`attr("name")`
`attr("name"，"value")`
`removeAttr("name")`
#### 创建节点
使用 jQuery 的工厂函数 `$()`:`$(html)`; 会根据传入的 html 标记字符串创建一个 DOM 对象， 并把这个 DOM 对象包装成一个 jQuery 对象返回
#### 删除节点
`remove()`: 从 DOM 中删除所有匹配的元素， 传入的参数用于根据 jQuery 表达式来筛选元素；当某个节点用 remove() 方法删除后， 该节点所包含的所有后代节点将被同时删，这个方法的返回值是一个指向已被删除的节点的引用
`empty()`: 清空节点 – 清空元素中的所有后代节点(不包含属性节点)
#### 内容操作
1. `html()`: 获取/设置元素的标签体内容   
```html
<a><font>内容</font></a>  --> <font>内容</font>
```
2. `text()`: 获取/设置元素的标签体纯文本内容  
```html
 <a><font>内容</font></a> --> 内容
```
3. `val()`： 获取/设置元素的value属性值

#### 属性操作
- 通用属性操作
1. `attr()`: 获取/设置元素的属性
2. `removeAttr()`:删除属性
3. `prop()`:获取/设置元素的属性
4. `removeProp()`:删除属性

**[注意]**：attr 和 prop 区别？
    1. 如果操作的是元素的固有属性，则建议使用prop
    2. 如果操作的是元素自定义的属性，则建议使用attr
- 对class属性操作
1. `addClass()`:添加class属性值
2. `removeClass()`:删除class属性值
3. `toggleClass()`:切换class属性
    `toggleClass("one")`: 判断如果元素对象上存在class="one"，则将属性值one删除掉。  如果元素对象上不存在 class="one"，则添加
4. `css()`:获取css样式


## JQuery对象和JS对象区别与转换
1. JQuery对象在操作时，更加方便。
2. JQuery对象和js对象方法不通用的。
3. 两者相互转换
    * jq -- > js : `jq对象[索引]` 或者 `jq对象.get(索引)`
    * js -- > jq : `$(js对象)`

## jQuery 高级
### 动画
三种方式显示和隐藏元素动画
1. 默认显示和隐藏方式
- `show([speed,[easing],[fn]])`
        参数：
        1. speed：动画的速度。三个预定义的值("slow","normal", "fast")或表示动画时长的毫秒数值(如：1000)
        2. easing：用来指定切换效果，默认是"swing"，可用参数"linear"
            * swing：动画执行时效果是 先慢，中间快，最后又慢
            * linear：动画执行时速度是匀速的
        3. fn：在动画完成时执行的函数，每个元素执行一次。

- `hide([speed,[easing],[fn]])`
- `toggle([speed],[easing],[fn])`

2. 滑动显示和隐藏方式
- `slideDown([speed],[easing],[fn])`
- `slideUp([speed,[easing],[fn]])`
- `slideToggle([speed],[easing],[fn])`

3. 淡入淡出显示和隐藏方式
- `fadeIn([speed],[easing],[fn])`
- `fadeOut([speed],[easing],[fn])`
- `fadeToggle([speed,[easing],[fn]])`

### 遍历
1. javascript的遍历方式
```js
for(初始化值;循环结束条件;步长)
```
2. jquery的遍历方式
- `jq对象.each(callback)`
```js
    1. 语法：
        jquery对象.each(function(index,element){});
            * index:就是元素在集合中的索引
            * element：就是集合中的每一个元素对象

            * this：集合中的每一个元素对象
    2. 回调函数返回值：
        * true:如果当前function返回为false，则结束循环(break)。
        * false:如果当前function返回为true，则结束本次循环，继续下次循环(continue)
```
- `$.each(object, [callback])`
- ` for..of` jquery 3.0 版本之后提供的方式
```js
    for(元素对象 of 容器对象)
```

### 事件绑定
1. jquery标准的绑定方式
`jq对象.事件方法(回调函数)`；
注：如果调用事件方法，不传递回调函数，则会触发浏览器默认行为。
    `表单对象.submit()`;//让表单提交
2. on绑定事件/off解除绑定
`jq对象.on("事件名称",回调函数)`
`jq对象.off("事件名称")`
    如果off方法不传递任何参数，则将组件上的所有事件全部解绑
3. 事件切换：toggle
`jq对象.toggle(fn1,fn2...)`
    当单击jq对象对应的组件后，会执行fn1；第二次点击会执行fn2.....
    
**注意**：1.9版本 `.toggle()` 方法删除，jQuery Migrate（迁移）插件可以恢复此功能。
```html
 <script src="../js/jquery-migrate-1.0.0.js" type="text/javascript" charset="utf-8"></script>
```
       
### 插件
增强JQuery的功能，实现方式如下：
1. `$.fn.extend(object)`
    增强通过Jquery获取的对象的功能 `$("#id")`
2. `$.extend(object)`
    增强JQeury对象自身的功能 `$/jQuery`

# Ajax

 AJAX 即“Asynchronous JavaScript and XML”（异步JavaScript和XML)，AJAX 并非缩写词，而是由Jesse James Gaiiett创造的名词，是指一种创建交互式网页应用的网页开发技术。WEB2.0

## 概述
使用javascript向服务器提出请求并处理响应而不阻塞用户，核心对象 XMLHTTPRequest。通过这个对象，JavaScript 可在不重载页面的情况与Web服务器交换数据。
AJAX 在浏览器与 Web 服务器之间使用异步数据传输（HTTP 请求），这样就可使网页从服务器请求少量的信息，而不是整个页面。
AJAX 可使因特网应用程序更小、更快、更友好。

## XMLHttpRequest 对象初始化
为了应对所有的现代浏览器，包括 IE5 和 IE6，请检查浏览器是否支持 XMLHttpRequest 对象。如果支持，则创建 XMLHttpRequest 对象。如果不支持，则创建 ActiveXObject ：
```js
//创建XMLHttpRequest对象
function ajaxFunction(){
    var xmlHttp;   
    try{ // Firefox, Opera 8.0+, Safari
            xmlHttp=new XMLHttpRequest();
        }
        catch (e){
            try{// Internet Explorer
                xmlHttp=new ActiveXObject("Msxml2.XMLHTTP");
            }
            catch (e){
            try{
                xmlHttp=new ActiveXObject("Microsoft.XMLHTTP");
            }
            catch (e){}
        }
    }
    return xmlHttp;
}
 ```
### XMLHttpRequest对象方法
 ![method](/images/tech/web-ajax-xmlhttprequest-method.png)

## 服务器端向客户端进行响应(注册监听)
```js
xhr.onreadystatechange=function(){
    if(xhr.readyState==4){
        if(xhr.status==200||xhr.status==304){
                var data = xhr.responseText;
        }
    }
}
```
`readyState`： 属性表示Ajax请求的当前状态。它的值用数字代表。
               0 代表未初始化。 还没有调用 open 方法
               1 代表正在加载。 open 方法已被调用，但 send 方法还没有被调用
               2 代表已加载完毕。send 已被调用。请求已经开始
               3 代表交互中。服务器正在发送响应
               4 代表完成。响应发送完毕

`xhr.status`：
    常用状态码及其含义：
    404 没找到页面(not found)
    403 禁止访问(forbidden)
    500 内部服务器出错(internal service error)
    200 一切正常(ok)
    304 没有被修改(not modified)（服务器返回304状态，表示源文件没有被修改)

`xhr.responseText`： 服务器发回的响应结果，字符串
`xhr.responseXML`： 服务器返回的响应结果，XML对象

## 客户端与服务器端建立连接
使用的是 XMLHttpRequest 对象的 `open(method, url, asynch)`方法
   * `method`：请求类型，类似 “GET”或”POST”的字符串。
   * `url`：路径字符串，指向你所请求的服务器上的那个文件。可以是绝对路径或相对路径。
   * `asynch`：表示请求是否要异步传输，默认值为true(异步)。
```js
xhr.open("GET","../testServlet?timeStamp="+new Date().getTime()+"&c=19",true);
```

## 客户端向服务器端发送请求
使用的是 XMLHttpRequest 对象的`send()`方法
   * 如果请求类型是GET方式的话，使用send()方法发送请求数据，服务器端接收不到
   * 该步骤不能被省略，只能写成`xhr.send(null)`;

### GET方式
```js
    xhr.send(null);         
```

### POST方式
如果请求类型是POST的话，需要设置请求首部信息
```js
    xhr.setRequestHeader("Content-Type","application/json");
    xhr.send("a=7&b=8");    
```

## jQuery实现Ajax
### $.ajax()
* 语法：`$.ajax({键值对});`
```js
    //使用$.ajax()发送异步请求
    $.ajax({
        url:"ajaxServlet1111" , // 请求路径
        type:"POST" , //请求方式
        //data: "username=jack&age=23",//请求参数
        data:{"username":"jack","age":23},
        success:function (data) {
            alert(data);
        },//响应成功后的回调函数
        error:function () {
            alert("出错啦...")
        },//表示如果请求响应出现错误，会执行的回调函数

        dataType:"text"//设置接受到的响应数据的格式
    });
```
### $.get()
* 语法：`$.get(url, [data], [callback], [type])`
* 参数：
        * url：请求路径
        * data：请求参数
        * callback：回调函数
        * type：响应结果的类型

### $.post()
* 语法：`$.post(url, [data], [callback], [type])`
* 参数：
        * url：请求路径
        * data：请求参数
        * callback：回调函数
        * type：响应结果的类型