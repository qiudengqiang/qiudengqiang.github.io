---
title: iOS - UIWebView & JSContext & WKWebView
date: 2018-05-08 17:33:53
tags: [iOS,Network,WebView]
categories: iOS
---
# UIWebView
> A view that embeds web content in your app.

## UIWebView 的JS注入
> 案例 : 移除网页的某些不需要展示的标签

- 准备网页地址 : http://m.dianping.com/tuan/deal/5501525

### 浏览器终端中演示JS代码删除网页中元素
* 需要处理的网页
![](/images/tech/webpage_process_before.png)
---
*  网页处理的步骤
``` javascript
以删除导航为例 :
  1.先找到该节点 : var headerTag = document.getElementsByTagName('header')[0];
  2.再找到父节点 : headerTag.parentNode
  3.最后用它的父节点删除该节点 : headerTag.parentNode.removeChild(headerTag);

  合并: var headerTag = document.getElementsByTagName('header')[0];headerTag.parentNode.removeChild(headerTag);
```
* 删除导航
``` javascript
var headerTag = document.getElementsByTagName('header')[0];headerTag.parentNode.removeChild(headerTag);
```
* 删除底部悬停按钮
``` javascript
var footerBtnTag = document.getElementsByClassName('footer-btn-fix')[0]; footerBtnTag.parentNode.removeChild(footerBtnTag);
```
* 删除底部布局
``` javascript
var footerTag = document.getElementsByClassName('footer')[0]; footerTag.parentNode.removeChild(footerTag);
```
* 处理之后的网页
![](/images/tech/webpage_process_after.png)

---
### OC调用JS 实现 JS注入
> OC和JS的交互需要使用UIWebView的代理方法作为桥梁实现

``` objc
- (void)viewDidLoad {
	[super viewDidLoad];

	NSURL * URL = [NSURL URLWithString:@"http://m.dianping.com/tuan/deal/5501525"];
	[self.webView loadRequest:[NSURLRequest requestWithURL:URL]];

	// 设置代理
	self.webView.delegate = self;
}
```
* 网页加载完时调用的代理方法
``` objc
- (void)webViewDidFinishLoad:(UIWebView * ) webView;
```
* 网页加载完成之后,调用JS代码的OC方法
``` objc
- (nullable NSString * )stringByEvaluatingJavaScriptFromString:(NSString * )script;
```

### JS注入的具体实现
``` objc
- (void)webViewDidFinishLoad:(UIWebView * )webView
{
	// 拼接JS的代码
	NSMutableString * JSStringM = [NSMutableString string];

	// 删除导航
	[JSStringM appendString:@"var headerTag = document.getElementsByTagName('header')[0];headerTag.parentNode.removeChild(headerTag);"];
	// 删除底部悬停按钮
	[JSStringM appendString:@"var footerBtnTag = document.getElementsByClassName('footer-btn-fix')[0]; footerBtnTag.parentNode.removeChild(footerBtnTag);"];
	// 删除底部布局
	[JSStringM appendString:@"var footerTag = document.getElementsByClassName('footer')[0]; footerTag.parentNode.removeChild(footerTag);"];

	// OC调用JS代码
	[webView stringByEvaluatingJavaScriptFromString:JSStringM];
}

```

## UIWebView监听网页标签的点击(JS调用OC)
> 案例 : 点击网页某个标签跳转到苹果原生控制器
> 核心思想 : 拦截webView上所有的网络请求

### JS调用OC需要实现的代理方法
``` objc
-(BOOL) webView:(UIWebView * )webView shouldStartLoadWithRequest:(NSURLRequest * )request navigationType:(UIWebViewNavigationType)navigationType;
```

### JS注入给标签添加点击事件
- 网页标签添加点击事件
``` objc
[JSStringM appendString:@"var figureTag = document.getElementsByTagName('figure')[0].children[0]; figureTag.onclick = function(){window.location.href = 'custom://techbird.me'};"];
```

- 标签的点击事件注入到JS
``` objc
- (void)webViewDidFinishLoad:(UIWebView * )webView
{
	// 拼接JS的代码
	NSMutableString * JSStringM = [NSMutableString string];

	// 删除导航
	[JSStringM appendString:@"var headerTag = document.getElementsByTagName('header')[0];headerTag.parentNode.removeChild(headerTag);"];
	// 删除底部悬停按钮
	[JSStringM appendString:@"var footerBtnTag = document.getElementsByClassName('footer-btn-fix')[0]; footerBtnTag.parentNode.removeChild(footerBtnTag);"];
	// 删除底部布局
	[JSStringM appendString:@"var footerTag = document.getElementsByClassName('footer')[0]; footerTag.parentNode.removeChild(footerTag);"];

	// 给标签添加点击事件
	[JSStringM appendString:@"var figureTag = document.getElementsByTagName('figure')[0].children[0]; figureTag.onclick = function(){window.location.href = 'custom://techbird.me'};"];

  	// OC调用JS代码
	[webView stringByEvaluatingJavaScriptFromString:JSStringM];
}
```

> * 给标签添加点击事件的目的 : 使标签可点击
> * 点击事件发送网络请求的目的 : 可以拦截到标签的点击事件
> * 自定义协议的目的 : 给事件设计一个特殊的标记,如果拦截到请求,就通过特殊的标记来区别要做的事情

### 拦截webView上所有的网络请求,筛选请求
``` objc
/**
 1.JS与OC交互的桥梁
 2.可以拦截webView上所有的请求
 3.给标签添加点击事件,点击事件主要就是发送请求;发送的请求是自定义协议的,目的是为了做标记.
 */
- (BOOL)webView:(UIWebView * )webView shouldStartLoadWithRequest:(NSURLRequest * )request navigationType:(UIWebViewNavigationType)navigationType
{
	NSLog(@"%@",request.URL.absoluteString);

	// 拿到网页的请求地址
	NSString * URLString = request.URL.absoluteString;
	// 判断网页的请求地址协议是否是我们自定义的那个
	NSRange range = [URLString rangeOfString:@"custom://techbird.me"];
	if (range.length > 0) {
		// 点击网页中的图片,实现OC原生界面的跳转
		TestViewController * VC = [[TestViewController alloc] init];
		[self.navigationController pushViewController:VC animated:YES];
		return NO;
	}
	return YES;
}
```
# JSContext
JSContext是JavaScript的运行上下文，他主要作用是执行js代码和注册native方法接口

## JSContexts实现OC与JS交互
- 获取上下文
``` objc
JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
```
## 使用JSContext 实现 JS调用OC
``` objc
- (void)viewDidLoad {
	[super viewDidLoad];

	NSURLRequest * request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://m.dianping.com/tuan/deal/5501525"]];
	[self.webView loadRequest:request];
	self.webView.delegate = self;

	// 获取上下文
	JSContext * context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
	// 监听图片标签点击
	context[@"imgtag"] = ^ {
		[self.navigationController pushViewController:[TestViewController new] animated:YES];
	};
	// 监听购买标签点击
	context[@"buytag"] = ^ {
		[self.navigationController pushViewController:[Test1ViewController new] animated:YES];
	};
}
```
### 使用JSContext 实现 JS注入
``` objc
- (void)webViewDidFinishLoad:(UIWebView * )webView
{
	// 拿到JS的上下文
	JSContext * context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];

	// 直接调用JS的函数,还可以向函数里面传入需要的参数.在XCode中向JS中的alert传入需要的message
	// 拼接JS的代码
	NSMutableString * JSStringM = [NSMutableString string];

	// 删除导航
	[JSStringM appendString:@"var headerTag = document.getElementsByTagName('header')[0];headerTag.parentNode.removeChild(headerTag);"];
	// 删除底部悬停按钮
	[JSStringM appendString:@"var footerBtnTag = document.getElementsByClassName('footer-btn-fix')[0]; footerBtnTag.parentNode.removeChild(footerBtnTag);"];
	// 删除底部布局
	[JSStringM appendString:@"var footerTag = document.getElementsByClassName('footer')[0]; footerTag.parentNode.removeChild(footerTag);"];

	// 给图片标签添加点击事件 : 自定义协议
	[JSStringM appendString:@"var figureTag = document.getElementsByTagName('figure')[0].children[0]; figureTag.onclick = function imgtagclick() {imgtag();};"];

	// 给以过期的购买标签重新添加点击事件
	[JSStringM appendString:@"var buyBtnTag = document.getElementsByClassName('buy-btn btn-gray')[0]; buyBtnTag.onclick = function buybtnclick() {buytag();};"];

	// 执行这个JS代码
	[context evaluateScript:JSStringM];
}
```

# WKWebView
> Starting in iOS 8.0 and OS X 10.10, use WKWebView to add web content to your app. Do not use UIWebView or WebView.

## WKWebView的OC和JS交互

- 使用前导入头文件
``` objc
   #import <WebKit/WebKit.h>
```

- 遵守代理协议
``` objc
webView.navigationDelegate = self;
```

### 代理方法介绍
- 面即将开始加载时调用 (拦截网页的网络请求 : JS调用OC)
``` objc
- (void)webView:(WKWebView * )webView decidePolicyForNavigationAction:(WKNavigationAction * )navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler;
```
- 页面开始加载时调用
``` objc
- (void)webView:(WKWebView * )webView didStartProvisionalNavigation:(WKNavigation * )navigation;
```
- 收到响应后,决定是否跳转,即是否把这个链接对应的网页加载到WKWebView上
``` objc
- (void)webView:(WKWebView * )webView decidePolicyForNavigationResponse:(WKNavigationResponse * )navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler;
```
- 当内容开始返回时调用,即服务器已经在向客户端发送网页数据
``` objc
- (void)webView:(WKWebView * )webView didCommitNavigation:(WKNavigation * )navigation;
```
- 页面加载完成之后调用 (OC调用JS : JS注入)
``` objc
- (void)webView:(WKWebView * )webView didFinishNavigation:(WKNavigation * )navigation;
```
- 页面加载失败时调用
``` objc
- (void)webView:(WKWebView * )webView didFailProvisionalNavigation:(WKNavigation * )navigation;
```
### 准备WKWebView
``` objc
- (void)viewDidLoad {
	[super viewDidLoad];

	// 创建WKWebView
	WKWebView * webView = [[WKWebView alloc] initWithFrame:[UIScreen mainScreen].bounds];
	[self.view addSubview:webView];
	webView.backgroundColor = [UIColor redColor];
	self.webView = webView;

	// 设置代理
	self.webView.navigationDelegate = self;

	// 加载的网页
	NSURL * URL = [NSURL URLWithString:@"http://m.dianping.com/tuan/deal/5501525"];
	NSURLRequest * request = [NSURLRequest requestWithURL:URL];
	[self.webView loadRequest:request];
}
```
### OC调用JS : JS注入 (类似UIWebView)
``` objc
// 页面加载完成之后调用
- (void)webView:(WKWebView * )webView didFinishNavigation:(WKNavigation * )navigation
{
	// 拼接JS的代码
	NSMutableString * JSStringM = [NSMutableString string];

	// 删除导航
	[JSStringM appendString:@"var headerTag = document.getElementsByTagName('header')[0];headerTag.parentNode.removeChild(headerTag);"];
	// 删除底部悬停按钮
	[JSStringM appendString:@"var footerBtnTag = document.getElementsByClassName('footer-btn-fix')[0]; footerBtnTag.parentNode.removeChild(footerBtnTag);"];
	// 删除底部布局
	[JSStringM appendString:@"var footerTag = document.getElementsByClassName('footer')[0]; footerTag.parentNode.removeChild(footerTag);"];

	// 给标签添加点击事件 : 自定义协议
	[JSStringM appendString:@"var figureTag = document.getElementsByTagName('figure')[0].children[0]; figureTag.onclick = function(){window.location.href = 'custom://techbird.me'};"];

	// OC调用JS代码
	[webView evaluateJavaScript:JSStringM completionHandler:nil];
}
```
### JS调用OC : (类似UIWebView)
``` objc
// 在发送请求之前，决定是否跳转
- (void)webView:(WKWebView * )webView decidePolicyForNavigationAction:(WKNavigationAction * )navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler
{
	NSLog(@"在发送请求之前，决定是否跳转 decidePolicyForNavigationAction");

	NSString * URLString = navigationAction.request.URL.absoluteString;
	NSLog(@"监测到的WKWebView上的请求 %@",URLString);

	NSRange range = [URLString rangeOfString:@"custom://"];
	if (range.length > 0) {

		[self.navigationController pushViewController:[[TestViewController alloc] init] animated:YES];

		// 不允许跳转,即不加载这个链接对应的内容
		decisionHandler(WKNavigationActionPolicyCancel);
	} else {
		// 允许跳转,即加载这个链接对应的内容
		decisionHandler(WKNavigationActionPolicyAllow);
	}
}
```

## WKWebView 监听加载进度
### 初始化WKWebView和进度条
``` objc
- (void)viewDidLoad {
	[super viewDidLoad];

	// 创建进度条
	self.progress = [[UIProgressView alloc] init];
	self.progress.frame = CGRectMake(0, 64, [UIScreen mainScreen].bounds.size.width, 10);
	[self.view addSubview:self.progress];
	self.progress.progress = 0;

	// 创建WKWebView
	WKWebView * webView = [[WKWebView alloc] initWithFrame:[UIScreen mainScreen].bounds];
	[self.view addSubview:webView];
	webView.backgroundColor = [UIColor redColor];
	self.webView = webView;

	// 设置代理
	self.webView.navigationDelegate = self;

	// 加载的网页
	NSURL * URL = [NSURL URLWithString:@"http://m.dianping.com/tuan/deal/5501525"];
	NSURLRequest * request = [NSURLRequest requestWithURL:URL];
	[self.webView loadRequest:request];

	// KVO添加进度监听
	[webView addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:nil];
}
```
### KVO监听进度
``` objc
- (void)observeValueForKeyPath:(NSString * )keyPath ofObject:(id)object change:(NSDictionary * )change context:(void * )context {

	if (object == self.webView && [keyPath isEqualToString:@"estimatedProgress"]) {

		CGFloat newprogress = [[change objectForKey:NSKeyValueChangeNewKey] doubleValue];

		NSLog(@"进度 %f",newprogress);

		if (newprogress != 1.000000) {

			// 网页加载时就展示进度
			self.progress.hidden = NO;
			self.progress.progress = newprogress;

		} else {
			// 网页加载完成就进度
			self.progress.hidden = YES;
		}
	}
}
```
## WKWebView 其他
### WKUIDelegate
- 创建一个新的WebView
``` objc
-(WKWebView * )webView:(WKWebView * )webView createWebViewWithConfiguration:(WKWebViewConfiguration * )configuration forNavigationAction:(WKNavigationAction * )navigationAction windowFeatures:(WKWindowFeatures * )windowFeatures;
```
- 弹出警告的提示框时调用
 ``` objc
/**
 *  弹出警告的提示框时调用
 *
 *  @param webView           实现该代理的webview
 *  @param message           警告框中的内容
 *  @param frame             主窗口
 *  @param completionHandler 警告框消失调用
 */
-(void)webView:(WKWebView * )webView runJavaScriptAlertPanelWithMessage:(NSString * )message initiatedByFrame:(void (^)())completionHandler;
```
- 弹出确认的提示框时调用
``` objc
/**
 *  弹出确认的提示框时调用
 *
 *  @param webView           实现该代理的webview
 *  @param message           确认框中的内容
 *  @param frame             主窗口
 *  @param completionHandler 警告框消失调用
 */
-(void)webView:(WKWebView * )webView runJavaScriptConfirmPanelWithMessage:(NSString * )message initiatedByFrame:(WKFrameInfo * )frame completionHandler:(void (^)(BOOL result))completionHandler;
```
- 弹出输入提示框时调用
``` objc
/**
 *  弹出输入提示框时调用
 *
 *  @param webView           实现该代理的webview
 *  @param message           确认框中的内容
 *  @param defaultText       默认的输入框文本信息
 *  @param frame             主窗口
 *  @param completionHandler 警告框消失调用
 */
-(void)webView:(WKWebView * )webView runJavaScriptTextInputPanelWithPrompt:(NSString * )prompt defaultText:(nullable NSString * )defaultText initiatedByFrame:(WKFrameInfo * )frame completionHandler:(void (^)(NSString * __nullable result))completionHandler;
```

### Bug Tips
https://github.com/ShingoFukuyama/WKWebViewTips
