---
title: iOS - 零散的小知识点收集
date: 2018-05-18 10:20:16
tags: [iOS,Objc]
categories: [iOS,Objc]
---
## APP启动程序执行过程
### main函数
### 执行UIApplicationMain函数
1. 创建UIApplication对象
2. 创建UIApplicationDelegate对象并复制
3. 读取配置文件info.plist，设置程序启动的一些属性
4. 创建应用程序的Main Runloop循环

### UIApplicationDelegate对象开始处理监听到的事件
1.程序启动成功之后，首先调用application:didFinishLaunchingWithOptions:方法，如果info.plist文件中配置了启动storyboard文件名，则加载storyboard文件。如果没有配置，则根据代码来创建UIWindow--->UIWindow的rootViewController-->显示

## 类方法initialize
- 会在类第一次被使用时调用，且只会调用一次
- 此方法的调用是线程安全的

## loadView的注意事项
1.  用于加载指定的视图，一旦重写了这个方法，Storyboard里面就不会去加载根视图了
2.  先于`viewDidLoad`调用
3.  不可以调用`super.loadView()`
4.  当`self.view == nil`时回调用此方法


## ViewController的生命周期
![](/images/controller_life_circle.png)
- `loadView`：用于加载制定的根试图
- `viewDidLoad`：试图加载完毕
- `viewWillAppear`：界面即将显示在屏幕上
- `viewDidAppear`：界面已经完全渲染在屏幕上
- `viewWillDisappear`：界面即将从屏幕上消失
- `viewDidDisappear`：界面已经完全消失
- `dealloc`：控制器销毁

## stringWithFormat 和 initWithFormat 区别（关于内存，ARC，释放，性能来说）
- `+stringWithFormat`:类方法，返回一个`autorelease`的NSString实例，不用手动release，在自动释放池中会自动释放。
- `-initWithFormat`:实例方法，返回一个自己alloc申请内存的NSString实例，根据OC内存管理黄金法则，管杀管埋，它则需要自己手动release。
- **小结**：这两个方法只是在没有使用ARC的时候有所不同，一个需要手动release一个则是自动进入autoreleasepool，所以在使用ARC的时候他们俩几乎没有什么区别。

## @synthesize是啥？什么情况下使用？
- 首先一旦重写来属性的setter和getter方法后,系统不再自动生成带下划线的成员变量,而这行代码会创造一个带下划线前缀的实例变量名,同时使用这个属性生成getter 和 setter 方法。
- 使用`@synthesize` 只有一个目的——给实例变量起个别名,或者说为同一个变量添加两个名字。
- 如果要阻止自动合成，记得使用 `@dynamic` 。经典的使用场景是你知道已经在某处实现了getter/setter 方法,而编译器不知道的情况。
- 如何使用：
``` Objc
@synthesize obj = _obj;
```

## 栈区/堆区/常量区
- 操作内存的栈区速度快;栈区存储空间地址是连续的
- 操作内存的常量区速度快;内存空间只开辟一次
- 操作内存的堆区速度相对栈区和常量区要慢些;堆区内存空间不连续,需要寻址的过程
``` Objc
	// 存储在栈区
	 int num = 10;
	// 存储在常量区  
	 NSString *str1 = @"hello";
	// 存储在堆区
	 NSString *str2 = [NSString stringWithFormat:@"hello_%d",i];
```

## `NSUInteger`和`NSInteger`的区别
- `NSUInteger` 无符号整数(没有负数)用 `%tu`
`%tu`NSUInteger的占位符，可以适配 NSUInteger的32位设备和64位设备
32位设备: NSUInteger是`无符号的int` (无符号表示没有正负数)
64位设备: NSUInteger是`无符号的long`
- `NSInteger `有符号整数(有正负数)用 `%zd`
`%zd`NSInteger的占位符，可以适配 NSInteger的32位设备和64位设备
32位设备: NSInteger是`有符号的int` (有符号表示有正负数)
64位设备: NSInteger是`有符号的long`
- 以上这种设计是为了自适应32位和64位CPU的架构.
