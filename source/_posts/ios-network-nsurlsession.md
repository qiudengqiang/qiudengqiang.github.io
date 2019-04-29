---
title: iOS - NSURLSession
date: 2016-04-02 19:31:42
tags: [iOS,Objc,Network,NSURLSession]
categories: [iOS,Objc]
---

Apple在 iOS9.0 之后已经放弃了 NSURLConnection，所以在现在的实际开发中，一般使用的是 iOS7.0 之后推出的 NSURLSession。NSURLSession 和 NSURLConnection 都提供了与各种协议，诸如 HTTP 和 HTTPS 进行交互的API。会话对象（NSURLSession 类对象）就是用于管理这种交互过程。它是一个高度可配置的容器，通过使用其提供的API，可进行细粒度的管理控制。它提供了在 NSURLConnection 中的所有特性，此外，它还可以实现 NSURLConnection 不能完成的任务，例如实现私密浏览。
## 结构图
![](/images/tech/network_class_struct.png)
## NSURLSession发送网络请求
``` Objc
- (void)viewDidLoad {
    [super viewDidLoad];
    NSURL * URL = [NSURL URLWithString:@"http://www.baidu.com"];
    NSURLSession * session = [NSURLSession sharedSession];
		NSURLSessionDataTask * dataTask = [session dataTaskWithURL:URL completionHandler:^(NSData * data, NSURLResponse * response, NSError * error) {
			 // data : 响应体; response : 响应头; error : 错误信息
			 if (error == nil && data != nil){
					 NSLog(@"%@ -- %@ -- %@",response,data,[NSThread currentThread]);
			 } else {
					 NSLog(@"%@",error);
			 }
	 }];
   [dataTask resume];
}
```

## HTML字符串反序列化
``` Objc
 [[NSOperationQueue mainQueue] addOperationWithBlock:^{
				// 反序列化HTML字符串
				NSString * html = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
				// 展示HTML字符串
				[self.webView loadHTMLString:html baseURL:URL];
	}];
```

## 小结
- 为了方便程序员使用，NSURLSession提供了一个全局单例 session.
- 所有的 任务(Task) 都是由 session 发起的.
- 所有的任务默认是挂起的，需要 resume.
- 完成回调是异步的
- session可以自定义,自定义的时候可以同时设置代理.
> AFNetworing 底层其实就是对 NSURLSession 的封装
