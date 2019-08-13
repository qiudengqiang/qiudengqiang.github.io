---
title: Cookie & Session 基础
date: 2019-07-02 10:01:47
tags: [Web,Cookie,Session]
categories: Web
---
# 会话技术
为了实现某一个需求，浏览器和服务器之间会产生多次的请求和响应；从打开浏览器访问服务器开始，到访问服务器结束关闭浏览器，之间的多次请求和响应称为：「浏览器和服务器之间的一次会话」
「一次会话」：浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止

## 功能
在一次会话的范围内的多次请求间，共享数据

## 方式
1. 客户端会话技术：Cookie
2. 服务器端会话技术：Session

# Cookie
Cookie，有时也用其复数形式 Cookies，指某些网站为了辨别用户身份、进行 session 跟踪而储存在用户本地终端上的数据(可以叫做浏览器缓存）

## 概念
客户端会话技术，将数据保存到客户端

## 快速入门
1. 创建Cookie对象，绑定数据
    new Cookie(String name, String value) 
2. 发送Cookie对象
    response.addCookie(Cookie cookie) 
3. 获取Cookie，拿到数据
    Cookie[]  request.getCookies()  

## 实现原理
基于响应头set-cookie和请求头cookie实现

## 方法
1. new cookie("key","value");
2. setPath(path);
3. setMaxAge(time);
4. response.addCookie(cookie);

删除cookie的时候只需要覆盖同路径，同名，只要存活时间(serMaxAge(0))设置为0就可以

## 一次可不可以发送多个cookie?
可以创建多个Cookie对象，使用response调用多次addCookie方法发送cookie即可。

## cookie在浏览器中保存多长时间？
-  默认情况下，当浏览器关闭后，Cookie数据被销毁
-  持久化存储： `setMaxAge(int seconds)`
    - 正数：将Cookie数据写到硬盘的文件中。持久化存储。并指定cookie存活时间，时间到后，cookie文件自动失效
    - 负数：默认值
    - 零：删除cookie信息

## cookie能不能存中文？
* 在tomcat 8 之前 cookie中不能直接存储中文数据。需要将中文数据转码---一般采用URL编码(%E3)
* 在tomcat 8 之后，cookie支持中文数据。特殊字符还是不支持，建议使用URL编码存储，URL解码解析

## cookie共享问题？
1. 假设在一个tomcat服务器中，部署了多个web项目，那么在这些web项目中cookie能不能共享？
* 默认情况下cookie不能共享
* `setPath(String path)`:设置 cookie 的获取范围。默认情况下，设置当前的虚拟目录
* 如果要共享，则可以将 path 设置为`/`

2. 不同的 tomcat 服务器间 cookie 共享问题？
* `setDomain(String path)`：如果设置一级域名相同，那么多个服务器之间 cookie 可以共享
* `setDomain(".baidu.com")`：那么tieba.baidu.com和news.baidu.com中 cookie 可以共享

## Cookie的特点和作用
1. cookie存储数据在客户端浏览器
2. 浏览器对于单个cookie 的大小有限制`(4kb)` 以及 对同一个域名下的总cookie数量也有限制(20个)

### 作用
1. cookie一般用于存出少量的不太敏感的数据
2. 在不登录的情况下，完成服务器对客户端的身份识别
3. eg： 记住上一次访问时间

# Session
服务器端会话技术，在一次会话的多次请求间共享数据，将数据保存在服务器端的对象中。HttpSession

## 快速入门
1. 获取HttpSession对象：
```java
    HttpSession session = request.getSession();
```
2. 使用HttpSession对象：
```java
    Object getAttribute(String name)  
    void setAttribute(String name, Object value)
    void removeAttribute(String name)   
```
## 生命周期
1. 调用`request.getSession()`;方法时开始创建
2. 默认是`30`分钟用户无操作自动销毁
3. 可以手动销毁

### 三种销毁Session对象情况
1. 不正常关闭服务(正常关闭服务器Session信息会被序列化到硬盘中 保存tomcat/work目录)
2. session 对象调用invalidate() 手动销毁Session对象
3. session 默认失效时间 30分钟（连续不使用Session对象时间）
```bash
    //选择性配置修改，在tomcat/conf/web.xml 配置
    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>
```

## 实现原理
Session的实现是依赖于Cookie的

## 细节
### 当客户端关闭后，服务器不关闭，两次获取 session 是否为同一个？
* 默认情况下，不是同一个。
* 如果需要相同，则可以创建 Cookie,键为 JSESSIONID，设置最大存活时间，让 cookie 持久化保存。
```java
Cookie c = new Cookie("JSESSIONID",session.getId());
c.setMaxAge(60*60);
response.addCookie(c);
```

### 客户端不关闭，服务器关闭后，两次获取的session是同一个吗？
* 不是同一个，但是要确保数据不丢失。tomcat自动完成以下工作
* session的`钝化`： 在服务器正常关闭之前，将session对象系列化到硬盘上
* session的`活化`： 在服务器启动后，将session文件转化为内存中的session对象即可。

## session的特点
1. session用于存储一次会话的多次请求的数据，存在服务器端
2. session可以存储任意类型，任意大小的数据

## 解决浏览器禁止cookie的情况 (了解即可)
浏览器无法保存cookie中jsession id ，无法完成Session追踪，通过程序重写URL(携带session的URL)

# 小结：session与Cookie的区别
1. session存储数据在服务器端，Cookie在客户端
2. session没有数据大小限制，Cookie有
3. session数据安全，Cookie相对于不安全
