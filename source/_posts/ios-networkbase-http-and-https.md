---
title: iOS - 网络基础 & Http & Https
date: 2018-04-29 17:29:30
tags: [iOS,Network,Http,Https]
categories: iOS
---
# HTTP
HTTP：Hyper Text Transfer Protocol（超文本传输协议）的缩写，HTTP是一个基于TCP/IP通信协议来传递数据，默认端口号为80,是一个应用层协议，由请求和响应构成，是一个标准的客户端服务器模型
## Request
### GET
``` html
GET / HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.117 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8

（这里是请求数据）
```
- 第一部分：第一行是请求行（request line）
- 第二部分：请求头（header），用来说明服务器要使用的附加信息
- 第三部分：空行，请求头后面的空行是`必须`的
- 第四部分：请求数据也叫主体，可以添加任意数据

### POST
``` html
POST / HTTP1.1
Host: www.baidu.com
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type: application/x-www-form-urlencoded
Content-Length: 40
Connection: Keep-Alive

name=Professional%20Ajax&publisher=Wiley
```
- 第一部分：请求行，第一行是post请求，以及http1.1版本。
- 第二部分：请求头部，第二行至第六行。
- 第三部分：空行，第七行的空行。
- 第四部分：请求数据，第八行。


### GET和POST的区别
- GET请求的数据会附在URL之后显示出来（数据放置在http协议头中）而POST把提交的数据放置在是HTTP包的包体中
- 传输数据的大小：HTTP协议没有对传输的数据大小进行限制，HTTP协议规范也没有对URL长度进行限制。但GET请求时特定浏览器和服务器对URL长度有限制（eg:IE限制2083个字节,2k+35）
- 安全性：POST的安全性要比GET的安全性高（GET提交时，用户名密码会明文出现在URL上）


## Response
``` html
HTTP/1.1 200 OK
Date: Fri, 22 April 2018 06:07:21 GMT
Content-Type: text/html; charset=UTF-8

<html>
	  <head></head>
	  <body>
			<!--body goes here-->
	  </body>
</html>
```
- 第一部分：状态行，由HTTP协议版本号， 状态码， 状态消息 三部分组成。
- 第二部分：消息报头，用来说明客户端要使用的一些附加信息
- 第三部分：空行，消息报头后面的空行是必须的
- 第四部分：响应正文，服务器返回给客户端的文本信息。

## Http状态码
- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示请求已被成功接收、理解、接受
- 3xx：重定向--要完成请求必须进行更进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求


## 网络通信三要素
### IP地址（主机名）
- 网络中设备的标示
- 本地回环地址：127.0.0.1 主机名：localhost

### 端口号
- 用于标示进程的逻辑地址，不同进程的标示
- 有效端口：`0-65535`
- 其中 `0-1024`由系统使用或者保留端口，开发中不要使用 1024 以下的端口
- **注意** : 跟HTTP相关的端口一定是80.服务器上有个进程是专门处理HTTP请求的,端口号是80.处理HTTPS请求的端口号是443.

### 传输协议
- UDP(数据报文协议)
	- 只管发送，不确认对方是否接收到
	- 将数据源和目的封装成数据包中，不需要建立连接
	- 每个数据报的大小限制在64K之内
	- 因为无需连接，因此是不可靠协议
	- 不需要建立连接，速度快
	- 应用场景：多媒体教室／网络流媒体 / 视频实时共享
	- 当视频共享时,出现卡屏,就是因为UDP协议在传递数据时出现丢包.

- TCP(传输控制协议)
	- 建立连接，形成传输数据的通道
	- 在连接中进行大数据传输（数据大小不受限制）
	- 通过三次握手完成连接，是可靠协议
	- 必须建立连接，效率会稍低
	- TCP协议的传输速度比UDP协议慢


### 三次握手的描述
- 图解：![](/images/tech/network_three_time_hand.png)
- 第一次握手：Client将标志位SYN置为1，随机产生一个值seq=J，并将该数据包发送给Server，Client进入SYN_SENT状态，等待Server确认。
- 第二次握手：Server收到数据包后由标志位SYN=1知道Client请求建立连接，Server将标志位SYN和ACK都置为1，ack=J+1，随机产生一个值seq=K，并将该数据包发送给Client以确认连接请求，Server进入SYN_RCVD状态。
- 第三次握手：Client收到确认后，检查ack是否为J+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=K+1，并将该数据包发送给Server，Server检查ack是否为K+1，ACK是否为1，如果正确则连接建立成功，Client和Server进入ESTABLISHED状态，完成三次握手，随后Client与Server之间可以开始传输数据了


### **注：**
> 通过 IP 找机器，通过 端口 找程序，通过 协议 确定如何传输数据


## TCP/IP网络参考模型
### 网络模型（理论）
![](/images/tech/network_theory_model.png)
### 网络参考模型（现实）
![](/images/tech/network_real_model.png)
### 通信过程
![](/images/tech/network_commute_progress.png)
- 应用层 : APP
- 传输层 : TCP,确定数据如何传输
- 网络层 : 确定目标计算机的IP地址
- 链路层 : 硬件,添加帧头帧尾
> HTTP网络传输协议在传输层选择的是TCP/IP协议

# HTTPS
- HTTPS : Hyper Text Transfer Protocol over Secure Socket Layer,是以安全为目标的HTTP通道,简单讲是HTTP的安全版.即HTTP下加入SSL层,HTTPS的安全基础是SSL.
- SSL : Secure Sockets Layer,表示安全套接层.
- TLS : Transport Layer Security,是SSL的继任者,表示传输层安全.
- SSL与TLS是为网络通信提供安全及数据完整性的一种安全协议。TLS与SSL在传输层对网络连接进行加密.
![](/images/tech/network_http_and_https_diff.png)

## HTTPS加密原理
![](/images/tech/network_encode_principle.png)

## 加密科普（对称，非对称，散列 ）
HTTPS一般使用的加密与HASH算法如下：
非对称加密算法：RSA，DSA/DSS
对称加密算法：AES，RC4，3DES
HASH算法：MD5，SHA1，SHA256
其中非对称加密算法用于在握手过程中加密生成的密码，对称加密算法用于对真正传输的数据进行加密，而HASH算法用于验证数据的完整性。由于浏览器生成的密码是整个数据加密的关键，因此在传输的时候使用了非对称加密算法对其加密。非对称加密算法会生成公钥和私钥，公钥只能用于加密数据，因此可以随意传输，而网站的私钥用于对数据进行解密，所以网站都会非常小心的保管自己的私钥，防止泄漏。

## 小结（面试时如何回答)
- HTTP就是一个用文本格式描述报文头并用双换行分隔报文头和内容，在TCP基础上实现的请求-响应模式的双向通信协议。
- HTTPS并不是一个单独的协议，是对工作在一个加密连接（SSL/TLS) 上的常规HTTP协议。通过在TCP和HTTP之间加入TLS（Transport Layer Security）来加密。
- SSL/TLS协议加密会使传输速度会变慢，更耗资源，但是更安全

相关文章：
1.	https://www.cnblogs.com/Yfling/p/6670495.html
2. http://fullstack.blog/2017/03/12/%E4%B9%9D%E4%B8%AA%E9%97%AE%E9%A2%98%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%86%9F%E6%82%89HTTPS/#BS-%E5%88%A9%E7%94%A8%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86
