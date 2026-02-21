---
title: iOS - pthread 使用和 __bridge
date: 2016-05-08 19:41:58
tags: [iOS,Thread,Objc]
categories: [iOS,Objc]
---

pthread是POSIX thread的简写，跨平台多线程的C语言开发框架,pthread是实现多线程的技术方案之一，NSThread就是对它的封装。

## pthread开启子线程的函数介绍
```
#import <pthread.h>
int pthread_create(pthread_t * __restrict, const pthread_attr_t * __restrict,
				   void *(*)(void *), void * __restrict);
```

### 参数
`pthread_t *` : 线程标示符，传入指向线程标示符的指针地址。
`pthread_attr_t *` :线程属性，传入指向线程属性的指针地。
`void*( * )(void * )` :新线程要执行的函数(任务)，传入函数地址，即函数名。
`void * `:传入到函数的参数。

### 返回值
- 返回int类型的值,0表示创建新线程成功,反之,创建新线程失败,返回失败的编号。
- C语言框架里面并不是非零即真原则；因为他们认为成功的结果只有一个，但是失败的原因有很多。

## pthread开启子线程的函数实现
``` objc
- (void)pthreadDemo {
	// 新线程的标示符
	pthread_t ID;
	// 创建子线程
	int result = pthread_create(&ID, NULL, demo, NULL);
	// 判断创建子线程是否成功
	if (result == 0) {
		NSLog(@“success”);
	} else {
		NSLog(@“failure”);
	}
}
```

## 子线程异步执行的函数/任务
```objc
void * demo(void *param)
{
	NSLog(@"demo %@",[NSThread currentThread]);
	return NULL;
}
```

## 小结
- C 语言中 `void *` 与 OC 中的 id 类似。
-  `void *(*)(void *)` 中的`(*)` 表示指向函数的指针，即函数指针，即函数名或者函数地址。


## *__bridge*
用作于普通的 C 指针与 OC 指针的转换，不做任何操作。
```
void *p;
NSObject *objc = [[NSObject alloc] init];
p = (__bridge void*)objc;
```
这里的 `void * p` 指针直接指向了 `NSObject * objc` 这个 OC 类，p 指针并不拥有 OC 对象，跟普通的指针指向地址无疑。所以这个出现了一个问题，OC 对象被释放，p 指针也就释放了。
## *__bridge_retained*
用作 C 指针与 OC 指针的转换，并且也用拥有着被转换对象的所有权
## *__bridge_transfer*
用作 C 指针与 OC 指针的转换，并在拥有对象所有权后将原先对象所有权释放。(只支持 C 指针转换 OC 对象指针)
其实可以理解为先将对象的引用计数器 +1，然后再将引用计数器 -1。
