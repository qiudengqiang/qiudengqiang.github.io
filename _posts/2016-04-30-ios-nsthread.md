---
title: iOS - 多线程之 NSThread
date: 2016-04-30 14:56:59
tags: [iOS,Objc,Thread,NSThread]
categories: [iOS,Objc]
---
# 多线程基础（NSThread）

## NSThread创建线程的三种方式
``` Objc
1. NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(demo:) object:@"alloc"];
2. [NSThread detachNewThreadSelector:@selector(demo:) toTarget:self withObject:@"detach"];
3. [self performSelectorInBackground:@selector(demo:) withObject:@"perform"];
```
## target和selector的关系
- 执行哪个对象上的哪个方法.


## 线程的状态-生命周期
- `start` ：就绪状态，等待被CPU调用，当被调用的时候为运行状态
- `sleep/加锁`：阻塞状态
- `exit`：完全杀死（非正常死亡），**不要在主线程中调用**


## 线程属性
### `name` (线程名称)
- 设置线程名称可以当线程执行的方法内部出现异常时，记录异常和当前线程


### `stackSize`(栈区大小)
- 默认情况下，无论是主线程还是子线程，栈区大小都是 512K
- 栈区大小可以设置 
  ``` Objc
  [NSThread currentThread].stackSize = 1024 * 1024;
  ```
- 必须是 `4KB` 的倍数


### `isMainThread` (是否主线程)

### `threadPriority` (线程优先级)
- 优先级，是一个浮点数，取值范围从`0~1.0`
- `1.0`表示优先级最高
- `0.0`表示优先级最低
- 默认优先级是 `0.5`
- **优先级高只是保证 CPU 调度的可能性会高**


### `qualityOfService` (服务质量,iOS 8.0 推出)
- `NSQualityOfServiceUserInteractive` - 用户交互，例如绘图或者处理用户事件
- `NSQualityOfServiceUserInitiated` - 用户需要
- `NSQualityOfServiceUtility` - 实用工具，用户不需要立即得到结果
- `NSQualityOfServiceBackground` - 后台
- `NSQualityOfServiceDefault` - 默认，介于用户需要和实用工具之间


## 线程安全-资源共享（互斥锁小结）
- `@synchronized`互斥锁，使用了线程同步技术
- 同步锁/互斥锁：可以保证被锁定的代码，同一时间，只能有一个县城可以操作
- `self`：锁对象，任何继承自NSObject的对象都可以是锁对象，因为内部都有一把锁，而且是默认开着的
- 锁对象：一定要是全局的锁对象，要保证所有的线程都能访问，`self`是最方便使用的锁对象
- 互斥锁锁定的范围应该尽量小，但是一定要锁住资源的`读写`部分
- 加锁后程序的执行效率比不加锁的时候要低，因为线程要的等待解锁
- 牺牲了性能保证了安全


## 原子属性和非原子属性-以及自旋锁
### `nonatomic` : 非原子属性
- 线程不安全，不考虑多线程情况时使用此属性
- 编译器少生成一些互斥加锁代码，可以提高效率。


### `atomic` : 原子属性
- 线程安全的,针对多线程设计的属性修饰符,是默认值.
- 特点 : 单写多读
- 单写多读 : 保证同一时间,只有一个线程能够执行setter方法,但是可以有多个线程执行getter方法.
- atomic 属性的setter里面里面有一把锁,叫做自旋锁.
- 原子属性的setter方法是线程安全的;但是,getter方法不是线程安全的.

### `nonatomic`和`atomic`对比：
- `nonatomic `: 非线程安全,适合内存小的移动设备.
- `atomic` : 线程安全,需要消耗大量的资源.性能比非原子属性要差一点

## 自旋锁和互斥锁的区别
举个比较形象的例子就是：在火车上上厕所，自旋锁是比较着急那个人一直会敲门，问好了没有好了没有；而互斥锁就会等待厕所内的人出来之后自己在进去。显然自旋锁更为高效

## 线程间通信（为什么能通信？）
- `performSelectorInBackground`
- `performSelectorOnMainThread`
- 因为多线程共享地址空间和数据空间， 一个线程的数据可以直接提供给其他线程使用,叫做线程间通信
