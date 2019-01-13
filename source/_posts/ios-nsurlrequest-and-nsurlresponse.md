---
title: iOS - NSURLRequest & NSURLResponse
date: 2018-05-02 22:30:51
tags: [iOS,Objc,Network,NSURLRequest,NSURLResponse]
categories: [iOS,Objc]
---
![](/images/tech/network_http_commute.png)
## NSURLRequest
### 创建请求对象 (缓存策略和超时时长都是默认的)
``` Objc
NSURLRequest * request = [NSURLRequest requestWithURL:url];
```
  ### 创建请求对象的同时指定缓存策略和超时时长
``` Objc
NSURLRequest * request = [NSURLRequest requestWithURL:url cachePolicy:0 timeoutInterval:15];
```
### 缓存策略
| 枚举 | 数值 | 说明 |
| -- | -- | -- |
| `NSURLRequestUseProtocolCachePolicy` | 0 | 默认的缓存策略 |
| `NSURLRequestReloadIgnoringLocalCacheData` | 1 | <ul><li>忽略本地缓存数据，始终加载服务器的数据</li><li>对数据的及时性要求高的应用</li></ul> |
| `NSURLRequestReturnCacheDataElseLoad` | 2 | 如果有缓存，就返回缓存，否则加载最新数据 |
| `NSURLRequestReturnCacheDataDontLoad` | 3 | 只加载缓存数据,不去服务器上获取(离线地图) |

### 超时时长
- 默认网络时长是 `60 s`
> `SDWebImage` 的默认超时时长是 `15` 秒
> `AFN` 的默认超时时长是 `60` 秒


### NSMutableURLRequest（可变请求）
``` Objc
// 可变的请求对象才能设置额外的信息
NSMutableURLRequest *requestM = [NSMutableURLRequest requestWithURL:url cachePolicy:0 timeoutInterval:15];
// 设置请求头 : 告诉服务器,我的设备是iphone
[requestM setValue:@"iphone AppleWebKit" forHTTPHeaderField:@"User-Agent"];
```
## NSURLResponse
### 响应头
| 响应属性 | 说明 |
| -- | -- |
| `URL` | 服务器反馈的 URL，有的时候，服务器会重定向新的 URL |
| `MIMEType` | <ul><li>服务器告诉客户端，返回的二进制数据的类型（纯文本，视频，语音，超文本等）</li><li>`ContentType`</li><li>根据 MIMEType 客户端就知道使用什么软件处理返回的二进制数据</li></ul> |
| `statusCode` | 状态码<br /><ul><li>1XX消息</li><li>2XX 成功</li><li>3XX 更多选择</li><li>4XX 客户端错误</li><li>5XX 服务器错误</li></ul> |
| `expectedContentLength` | 数据长度，下载文件总长度 |
| `suggestedFilename` | 获取服务器的文件的名称 |
| `allHeaderFields ` | 返回数据的头部信息，key－value格式 |
| `textEncodingName ` | 编码的名称 |

### 响应体 data
- `data` 服务器返回的二进制数据，程序员最关心的内容
- 拿到响应体之后,无法直接使用,需要进行反序列化,转换成OC对象.


## GET请求 URL中有中文时如何处理？
``` Objc
NSString * URLString = [URLString stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
```
> 注意：GET请求时,问号`?`后面的查询字符串里面不能有中文或者空格.如果有就需要使用%转义,不然URL会为nil. POST请求时,请求体里面可以有中文.
> URLQueryAllowedCharacterSet : 百分号转义查询字符串

## URL转字符串的方法
``` objc
[filePath.path / filePath.absoluteString]
```
