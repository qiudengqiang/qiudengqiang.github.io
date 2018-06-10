---
title: iOS - 多线程之 GCD
date: 2018-04-29 15:59:59
tags: [iOS,Objc,Thread,GCD]
categories: [iOS,Objc]
---
## GCD与 NSThread 的对比

### NSThread的劣势
- 如果要开多个线程 NSThread 必须实例化多个线程对象
- NSThread 靠 NSObject 的分类方法实现的线程间通讯，GCD依靠 block 实现

### GCD的优势
- 让代码更加简单，易于阅读和维护
- 使用GCD 不需要管理线程的创建/销毁/复用的过程，不用关心线程的生命周期
- GCD会充分利用CPU的内核

## 队列有哪些，任务有几种
### 队列
- 串行队列（Serial Dispatch Queue）
``` Objc
dispatch_queue_t queue = dispatch_queue_create(“flag”, DISPATCH_QUEUE_SERIAL);
```
- 并行队列（Concurrent Dispatch Queue）
``` Objc
dispatch_queue_t queue = dispatch_queue_create(“flag”, DISPATCH_QUEUE_CONCURRENT);
```

### 任务
- 同步任务（sync）
``` Objc
dispatch_sync(queue, ^block);
```
- 异步任务（async）
``` Objc
dispatch_async(queue, ^block);
```

## 写一个死锁程序
- 主队列+同步任务 = 死锁
``` Objc
 - (void)deadlockDemo
{
	dispatch_queue_t queue = dispatch_get_main_queue();
	NSLog(@"start”);
	dispatch_sync(queue, ^{
		NSLog(@"excuting...%@",[NSThread currentThread]);
	});
	NSLog(@"end");
}
```
- 死锁解决办法：主队列中的同步任务放进子线程中，不使其阻塞主线程
``` Objc
 - (void)resolveDemo
{
	dispatch_async(dispatch_queue_create("flag", DISPATCH_QUEUE_CONCURRENT), ^{
		NSLog(@"start");
		dispatch_sync(dispatch_get_main_queue(), ^{
			NSLog(@"excuting...%@",[NSThread currentThread]);
		});
		NSLog(@"end");
	});
}
```

## 全局队列的两个参数分别代表什么？
- 参数1：服务质量(队列对任务调度的优先级)/iOS 7.0 之前，是优先级，传入`0`在所有系统上使用默认设置
- 参数2：预留参数，以便于扩展，一般传入`0`
- **小结**：如果要适配 iOS 7.0 & 8.0，需要使用以下代码：
``` Objc
 dispatch_get_global_queue(0, 0);
```

## 全局队列和并发队列的区别
### 全局队列
- 没有名称
- 无论MRC & ARC都不需要考虑释放

### 并发队列
- 有名称，和 NSThread 的 name 属性作用类似
- 如果在 `MRC` 开发时,需要使用 `dispatch_release(q);` 释放相应的对象


### 队列和任务组合总结
- 串行和并发决定了任务的执行方式(串行一次一个，并发一次多个)
- 同步和异步决定了要不要开启新的线程 (同步不开，异步开)

## GCD延迟执行(after)
- 延迟操作: `dispatch_after` 这个函数默认是异步执行的
``` Objc
-(void)afterDemo{
  NSLog(@"start");
  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0*NSEC_PER_SEC)),dispatch_get_global_queue(),^{
    NSLog(@"delay code");
  });
  NSLog(@"end");
}
```

### GCD一次性执行(once)
- `dispatch_once_t` 内部有一把锁,能够保证线程安全.
- **原理**：`onceToken` 有个初始值,当第一次执行时,判断是否是初始值,如果是初始值就执行函数内部的代码,执行结束之前会修改`onceToken`初始值.反之,就不执行.
``` Objc
- (void)onceDemo
{
	NSLog(@"mark");
	static dispatch_once_t onceToken;
	NSLog(@"result %ld",onceToken);
	dispatch_once(&onceToken, ^{
		NSLog(@"hello");
	});
}
```

## 单例设计模式（iOS）

### 单例设计模式的特点
1. 有一个全局访问点（供全局实例化单例的类方法）
2. 单例保存在静态存储区
3. 在内存有且只有一份
4. 生命周期跟APP一样长


### 如何做到被子类继承
- 在给instance赋值时要使用`[self new];`或者`[[self alloc] init];`


### 懒汉式
``` Objc
+ (instancetype)shared
{
	// 保存在静态存储区
	static id instance;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		instance = [[self alloc] init];
	});
	return instance;
}
```
### 饿汉式
``` Objc
@implementation Singleton
// 声明静态对象
static id instance;
+ (void)initialize
{
  // 只会开辟一次内存空间,只会被实例化一次
  instance = [[self alloc] init];
}

+ (instancetype)shared
{
	return instance;
}
@end
```
