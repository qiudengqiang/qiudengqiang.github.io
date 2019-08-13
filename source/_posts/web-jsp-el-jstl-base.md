---
title: JSP/EL/JSTL 基础
date: 2019-07-08 10:01:47
tags: [Web,JSP,EL,JSTL]
categories: Web
---

# JSP
JSP(JavaServer Pages) 是一种动态页面技术，它实现了Html语法中的java扩展（以 `<%, %>`形式）。JSP 与 Servlet 一样，是在服务器端执行的。通常返回给客户端的就是一个 HTML 文本，因此客户端只要有浏览器就能浏览。
## 指令
### 作用
用于配置 JSP 页面，导入资源文件
### 格式
<%@ 指令名称 属性名1=属性值1 属性名2=属性值2%>
### 分类
1. page：用于标识和配置 JSP 页面
- contentType：类似于response.setContentType()，用于设置响应体的mime类型及字符集，设置当前 JSP 页面编码
- import：导入包
- errorPage：当前页面发生异常后，会跳转到指定的错误页面，例如 404 等
- isErrorPage：标识当前是否是错误页面，参数：true（可以使用内置对象 exception），页面默认值是 false
- session：用于指定当前页面是否可以使用session
- pageEncoding：解决乱码问题(在低级工具中声明)

2. include：包含页面，导入其他页面资源文件：
<%@ include file="index.jsp"%>
3. taglib：用于导入第三方资源
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%> 其中prefix支持自定义

### 其他指令
1. JSP脚本表达式 ：       <%=  java表达式 %>
2. JSP脚本片断   ：       <% 任意多的java语句 %>
3. JSP声明      ：       <%!  java代码 %>

## 注释
### html注释
	`<!-- -->`:只能注释 html 代码片段
### jsp注释
推荐使用：`<%-- --%>`：可以注释所有

## 内置对象

* 在jsp页面中不需要创建，直接使用的对象, 一共有9个：
        变量名					真实类型						作用
    * `pageContext`				PageContext					当前页面共享数据，还可以获取其他八个内置对象
    * `request`					HttpServletRequest			一次请求访问的多个资源(转发)
    * `session`					HttpSession					一次会话的多个请求间
    * `application`				ServletContext				所有用户间共享数据
    * `response`					HttpServletResponse			响应对象
    * `page`						Object						当前页面(Servlet)的对象  this
    * `out`						JspWriter					输出对象，数据输出到页面上
    * `config`					ServletConfig				Servlet的配置对象
    * `exception`					Throwable					异常对象

### 详解
- `out`
相当于`response.getWriter()`获取到的输入流，内置单独的缓冲区
[ buffer="none | `8kb` | sizekb" ] out隐式对象所使用的缓冲区的大小
[ autoFlush="true | false" ] out隐式对象是否自动刷新缓冲区，默认为true，不需要更改
- `pageContext`
代表当前 JSP 的运行环境
生命周期:  随着 JSP 页面打开而创建,随着关闭而销毁.
用处:     在整个 JSP 页面中共享数据
    
1. 可以作为入口对象获取其他八大隐示对象的引用
    `getException`
    `getPage`
    `getRequest`
    `getRespons`
    `getServletConfig`
    `getServletContext` 方法返回 application 隐式对象
    `getSession`
    `getOut`
    _______________返回的都是本类型的隐示对象
    
    
2. 首先是一个域对象，作为入口操作四大作用域中的数据
```java
ServletContext(application域) > Session(session域)  > request(request域) >pageContext(page域)

//用来操作page域中的属性
public void setAttribute(java.lang.String?name,java.lang.Object?value)
public java.lang.Object?getAttribute(java.lang.String?name)
public void?removeAttribute(java.lang.String?name)

//用来操作四大作用域中任意域中的属性
public void setAttribute(java.lang.String?name, java.lang.Object?value,int?scope)
public java.lang.Object?getAttribute(java.lang.String?name,int?scope)
public void?removeAttribute(java.lang.String?name,int?scope)

PageContext.APPLICATION_SCOPE
PageContext.SESSION_SCOPE
PageContext.REQUEST_SCOPE
PageContext.PAGE_SCOPE
```
 **小结**： 四大作用域什么时候用:
    如果一个数据只在当前jsp页面中使用，存入page域。
    如果一个数据，在Servlet中处理好后，请求转发到其他serlvet或jsp使用此数据，放入request域。
    如果一个数据，我当前要用，过一会我自己还要用。存入session。
    如果一个数据，我当前要用，过一会其他人也要用。存入application域。
    ___________________________________

    `findAttribute`: (此方法用来查找各个域中的属性)
    从最小的域开始向最大的域搜索(page,request,session,application),找到就则返回该值,找完四大作用域找不到则返回null


3. 提交了快捷方法,实现请求转发和请求包含
        pageContext.forward("");
        pageContext.include("");

# EL
EL(Expression Language) 表达式语言,用于获取数据、执行运算、操作web中隐含对象、调用java函数
## 语法
`${标识符}`
EL表达式语句在执行时，会调用`pageContext.findAttribute`方法，用标识符为关键字，分别从 `page` 、 `request` 、 `session` 、`application` 四个域 中查找相应的对象，找到则返回相应对象，找不到则返回`””` （注意：不是null，而是空字符串）。
**「经验」**：只要标识符中没有逻辑错误，语法错误，就不会报错，`${a}`的结果是个`“”`字符串。


## 运算符
1. 算数运算符： + - * /(`div`) %(`mod`)
2. 比较运算符： > < >= <= == !=
3. 逻辑运算符： &&(`and`) ||(`or`) !(`not`)
4. 空运算符： `empty`
    * 功能：用于判断字符串、集合、数组对象是否为 null 或者长度是否为 0
    * `${empty list}`:判断字符串、集合、数组对象是否为 null 或者长度为 0
    * `${not empty str}`:表示判断字符串、集合、数组对象是否不为 null 并且 长度 > 0

## 获取值
EL 表达式只能从域对象中获取值

### 语法
1. `${域名称.键名}`：从指定域中获取指定键的值
    * 域名称：
        1. pageScope		--> pageContext
        2. requestScope 	--> request
        3. sessionScope 	--> session
        4. applicationScope --> application（ServletContext）
    * [举例]：在request域中存储了name=zhangsan
    * [获取]：${requestScope.name}

2. `${键名}`：表示依次从最小的域中查找是否有该键对应的值，直到找到为止。

3. 获取对象、List 集合、Map 集合的值
    1. 对象：`${域名称.键名.属性名}`
        * 本质上会去调用对象的 `getter` 方法

    2. List集合：`${域名称.键名[索引]}`

    3. Map集合：
        * `${域名称.键名.key名称}`
        * `${域名称.键名["key名称"]}`
 
 4. 调用java方法                       
    * 调用的方法必须是静态方法
    * 在开发中一般是直接使用(taglib)指令导入相关的包,j2ee5.0以上已经内置

### 隐式对象
EL 表达式中有11个内置对象：不需要定义直接在 EL 中可以直接使用

![域对象](/images/tech/web-scope-object-hidden01)
![6个隐式对象](/images/tech/web-scope-object-hidden02)

**「注意」**
* 只能用来获取集合数组中的数据不能用来遍历
* 只能用来获取数据不能用来设置数据

## 小结
### getParameter 和 getAttribute 区别 ？
=====================================================
getParameter 获得 HTTP中请求参数的值

getAttribute 获得服务器端各种数据范围的值 `request.getAttribute` 获得request数据范围的值
* 值在服务器端通过 `request.setAttribute` 保存的

getParameter 获得客户端提交的数据，getAttribute 服务器内部传递的数据

### getParameter() 与 getParamValues() 的区别 ?
=====================================================
param 、paramValues 用户获得请求参数的值
`${param.name}` ==== request.getParameter("name");
`${paramValues.name}` ===== request.getParameterValues("name");

### getHeader() 与 getHeaderValues() 区别 ? 
=====================================================
header、headerValues 获得请求头信息的数据
`${header["user-agent"]}` ========== request.getHeader("user-agent");
`${headerValues["user-agent"]}` ============= request.getHeaders("user-agent");

### cookie 
=====================================================
用来在开发中快速获得 cookie 的值  ------- 是一个Map<String,Cookie> //保持的value是对象
`${cookie.name.value}` 获得指定名称的cookie的value值

### initParam 
=====================================================
用来快速读取 ServletContext 的全局初始化参数
`${initParam.name}` ===================== getServletContext().getInitParameter("name");
* <context-param> 配置全局参数

### 最常使用
=====================================================
cookie   ----------- `${cookie.name.value}` 这里 name 是 cookie 的 name 值
pageContext ------------- `${pageContext.request.contextPath}` ==== pageContext.getRequest().getContextPath() 返回 `/virtual-directory`
```jsp
<!-- 编写一个链接 -->
<a href="/virtual-directory/el/1.jsp">link</a>
<!-- 使用EL 获得工程名 -->
<a href="${pageContext.request.contextPath }/el/1.jsp">link</a>
```
快速获得ip ： `${pageContext.request.remoteAddr}`
注册信息的回调 ： `${param.username}` ，因为提交的信息还保存在request里

## 注意
JSP 默认支持 EL 表达式的。如果要忽略 EL 表达式则使用以下方法：
1. JSP 中 page 指令中：`isELIgnored="true" `忽略当前jsp页面中所有的 EL 表达式
2. `\${表达式}` ：忽略当前这个el表达式

# JSTL
JSTL (JavaServer Pages Tag Library) 是 JSP 标准标签库，是由 Apache 组织提供的开源的免费的 JSP <标签>，用于简化和替换 JSP 页面上的Java 代码

## 使用步骤
1. 导入jstl相关jar包
2. 引入标签库：taglib指令：  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
3. 使用标签

## 常用标签

`<c:out>` 标签用于输出一段文本内容到`pageContext`对象当前保存的`out`对象中。
            
`<c:set>`标签用于把某一个对象存在指定的域范围内，或者设置 Web 域中的 java.util.Map 类型的属性对象或 JavaBean 类型的属性对象的属性。

`<c:remove>`标签用于删除各种 Web 域中的属性

`<c:catch>`标签用于捕获嵌套在标签体中的内容抛出的异常，其语法格式如下：
```jsp
<c:catch [var="varName"]>nested actions</c:catch>
```
        
`<c:if test=“”>` 标签可以构造简单的`if-then`结构的条件表达式
```jsp
<c:if test="条件,可以写el表达式">
</c:if>
```
`<c:choose>` 标签用于指定多个条件选择的组合边界，它必须与`<c:when>`和`<c:otherwise>`标签一起使用。
                    使用`<c:choose>`，`<c:when>`和`<c:otherwise>`三个标签，可以构造类似 `if-else if-else` 的复杂条件判断结构。
                    
`<c:forEach>` 标签用于对一个集合对象中的元素进行循环迭代操作，或者按指定的次数重复迭代执行标签体中的内容。语法格式如下：
```jsp
//类似于java中的fori
<c:forEach begin="1" end="length" var="i">
</c:forEach>
//类似于java中的foreach
<c:forEach items="strs" var="str" varStatus="s">
</c:forEach>
```
    
`<c:forTokens>`用来浏览一字符串中所有的成员，其成员是由定义符号所分隔的
    
`<c:param>`标签  在JSP页面进行URL的相关操作时，经常要在 URL 地址后面附加一些参数。
                `<c:param>`标签可以嵌套在`<c:import>`、`<c:url>`或`<c:redirect>`标签内，为这些标签所使用的URL地址附加参数。
                    
`<c:import>` 标签,实现 include 操作

`<c:url>` 标签用于在 JSP 页面中构造一个 URL 地址，其主要目的是实现 URL 重写。URL 重写就是将会话标识号以参数形式附加在 URL 地址后面

`<c:redirect>` 标签用于实现请求重定向