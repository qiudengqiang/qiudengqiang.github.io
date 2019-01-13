---
title: iOS - RunLoop & AutoreleasePool
date: 2018-05-24 09:30:51
tags: [iOS,Objc,RunLoop,AutoreleasePool]
categories: [iOS,Objc]
---
# RunLoop
>A NSRunLoop object processes input for sources such as mouse and keyboard events from the window system, NSPort objects, and NSConnection objects. A NSRunLoop object also processes NSTimer events.

## RunLoop 的概念
运行循环也叫消息循环，作用是循环的捕捉消息，然后执行消息对应的操作；并且保证应用程序不会退出。
OSX/iOS 系统中，提供了两个这样的对象：`NSRunLoop` 和 `CFRunLoopRef`。
`CFRunLoopRef` 是在 `CoreFoundation` 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。
`NSRunLoop` 是基于 `CFRunLoopRef` 的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。

## Input source 与 Timer source
`Input source` 与 `Timer source`这两个都是 RunLoop 事件的来源

### Input Source 可以分为三类
- Port-Based Sources：系统底层的 Port 事件源，例如`CFSocketRef`，但在应用层中几乎用不到。
- Custom Input Sources：用户手动创建的事件源，例如手势，触摸，键入。
- Cocoa Perform Selector Sources：Cocoa 提供的`performSelector`系列方法也是一种事件源。

### Timer Source
即是指定时器事件（`NSTimer`）。

## RunLoop 与线程
线程和 RunLoop  之间是一一对应的，其关系是保存在一个全局的 Dictionary 里。线程刚创建时候并没有 RunLoop，如果你不主动获取，那么它一直不会有。RunLoop 的创建是发生在第一次获取时，RunLoop 的销毁是发生在线程结束时。所以只能在一个线程的内部获取 RunLoop(主线程除外)。

下图中展现了 RunLoop 在线程中的作用：从 input source 和 timer source 接受事件，然后在线程中处理事件。
![RunLoop](https://github.com/qiudengqiang/blog-images/blob/master/objc_runloop.jpg)

### Warning
>The NSRunLoop class is generally not considered to be thread-safe and its methods should only be called within the context of the current thread. You should never try to call the methods of an NSRunLoop object running in a different thread, as doing so might cause unexpected results.

## RunLoop Observer
RunLoop 通过监听 Source 来决定有没有任务要做；除此之外，我们还可以用 RunLoop Observer 来监控 RunLoop 本身的状态。

RunLoop Observer 可以监听以下 RunLoop 事件：
- The entrance to the run loop.
- When the run loop is about to process a timer.
- When the run loop is about to process an input source.
- When the run loop is about to go to sleep.
- When the run loop has woken up, but before it has processed the event that woke it up.
- The exit from the run loop.


## RunLoop Mode
在监听与被监听中，RunLoop 要处理的事情非常复杂；为了让 RunLoop专注于处理特定事件而引入了 RunLoop Mode 概念
![RunLoop Mode](https://github.com/qiudengqiang/blog-images/blob/master/objc_runloop_mode.png)
如果所示，RunLoop Mode 实际上是 Source、Timer、Observer 的集合，不同的 Mode 把不同组的 Source、Timer、Observer 隔绝开来；而 RunLoop 在某个时刻下只能跑在某一个 Mode 下，处理这一个 Mode 当中的Source、Timer 和 Observer。

苹果文档中提到的 Mode 有五个，分别是：
- `NSDefaultRunLoopMode`
- `NSConnectionReplyMode`
- `NSModalPanelRunLoopMode`
- `NSTrackingRunLoopMode`
- `NSRunLoopCommonModes`

iOS 中公开暴露出来的只有 `NSDefaultRunLoopMode` 、`UITrackingRunLoopMode` 和 `NSRunLoopCommonModes。` 而`NSRunLoopCommonModes` 实际上是一个 Mode 的集合，默认包括 `NSDefaultRunLoopMode` 和 `NSTrackingRunLoopMode`。

## RunLoop 的使用
RunLoop 和线程是绑定在一起的；每个线程（包括主线程）都有一个对应的 RunLoop 对象。

### 获取 RunLoop
我们不能自己创建 RunLoop 对象，只能获取到系统提供的 RunLoop 对象。
```objc
[NSRunLoop currentRunLoop];
[NSRunLoop mainRunLoop];
```
### 在主线程和子线程中区别
RunLoop 在主线程和子线程中的 **区别** 在于：主线程的 RunLoop 会在应用启动的时候默认开启；其他线程（子线程）的 RunLoop 默认并不会启动，需要手动开启。
```objc
//手动启动RunLoop，无法控制结束
[[NSRunLoop currentRunLoop] run];
//手动启动RunLoop，指定结束时间
[[NSRunLoop currentRunLoop] runUntilDate:[[NSDate date] dateByAddingTimeInterval:5]];
```

## RunLoop 与 NSTimer
### timerWithTimeInterval:
```Objc
 //需要手动加入 RunLoop 并设置 Mode
 NSTimer *timer = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(timeFire) userInfo:nil repeats:YES];
 //立即调用一次
 [timer fire];
 //手动加入 RunLoop
 [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];
```
### scheduledTimerWithTimeInterval:
```Objc
 //默认加入 RunLoop 并设置 Mode 为 NSDefaultRunLoopMode
 NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(timeFire) userInfo:nil repeats:YES];
 //立即调用一次
 [timer fire];
```
### 坑
日常开发中，与 RunLoop 接触得最近可能就是通过 NSTimer，一个 Timer 一次只能加入到一个 RunLoop 中。我们日常使用的时候，通常就是加入到当前的 RunLoop 的 `NSDefaultRunLoopMode` 中，而 ScrollView 在用户滑动时，主线程 RunLoop 会转到 `UITrackingRunLoopMode`；而这个时候， Timer 就不会运行。

**解决方案：**
设置RunLoop Mode，例如 NSTimer，我们指定它运行于 `NSRunLoopCommonModes` ，这是一个Mode的集合。这样无论当前 RunLoop 运行哪个 Mode ，事件都能得到执行。

例如在 AFNetworking 中，就有相关的代码，如下：
```Objc
- (void)startActivationDelayTimer {
    self.activationDelayTimer = [NSTimer
                                 timerWithTimeInterval:self.activationDelay target:self selector:@selector(activationDelayTimerFired) userInfo:nil repeats:NO];
    [[NSRunLoop mainRunLoop] addTimer:self.activationDelayTimer forMode:NSRunLoopCommonModes];
}
```
这里就是添加了一个计时器，由于指定了 `NSRunLoopCommonModes`，所以不管 RunLoop 处于什么状态，都会执行这个计时器任务。

# Runloop 和 Autoreleasepool 的关系图解
![关系图解](https://github.com/qiudengqiang/blog-images/blob/master/objc_autoreleasepool.png)

# Autorelease Pool
Autorelase Pool 提供了一种可以允许你向一个对象延迟发送 `release` 消息的机制；当你想放弃对象的所有权，同时又不希望这个对象被立即释放掉（例如在一个方法中返回一个对象时），Autoreleasepool就可以发挥作用。所谓的延迟发送`release`消息是指，当我们把一个对象标记为autorelease时:
```Objc
NSString* str = [[[NSString alloc] initWithString:@"hello"] autorelease];
```
这个对象的`retainCount+1`，但不会发生 `release`；当这个变量所处的`autoreleasepool`被 `倾倒(drain)` 操作时，所有标记了autorelease的对象的 `retainCount-1`；即`release`消息的发送被延迟到`autoreleasepool`释放的时候了。
在 ARC 环境下，苹果引入了 `@autoreleasepool` 语法，不在需要手动调用`autorelease` 和 `drain` 等方法。
>提示 : 此处讨论的自动释放池不是手动创建的，是跟运行循环相关的，并非 `main.m` 中的 `@autoreleasepool`

## Autorelease Pool 的用处
在 ARC 下我们并不需要手动调用`autorelease`有关的方法，甚至可以完全不知道 autorelease 的存在，就可以正确的管理好内存；因为 Cocoa Touch 的 RunLoop 中，每个 RunLoop Circle中系统都加入了 Autorelease Pool 的创建和释放。

当我们需要创建和销毁大量对象时，使用手动创建的`autoreleasepool`可以有效的避免内存峰值的出现；因为如果不手动创建的话，外层系统创建的 Pool 会在整个 Runloop Circle 结束之后才执行 drain 操作，手动创建的话会在 block 结束之后就就执行 drain 操作。详情请见[苹果官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html#//apple_ref/doc/uid/20000047-CJBFBEDI)

一个普遍被使用的例子如下：
```Objc
for (int i = 0; i < 10000000; i++)
{
    @autoreleasepool
    {
        NSString* string = @"abc";
        NSArray* array = [string componentsSeparatedByString:string];
    }
}
```
在上面的例子中如果不使用`autoreleasepool`，需要在循环结束之后释放1000000个字符串；如果使用的话会在每次循环结束的时候都进行release操作。

## Autorelease Pool 进行 Drain 的时机
如上所述，系统在 RunLoop 中创建的 autoreleasepool 会在 RunLoop 的一个 Event 结束时进行 Drain 操作；而我们手动创建的 autoreleasepool 会在 block 执行完后进行 Drain  操作。

但需要注意的是：
- 当 block 以异常(exception)结束时，pool 不会进行 drain 操作
- Pool 的 drain 操作会把所有标记为 autorelease 的对象`retainCount-1`，但并不意味着这个对象一定会被释放掉；我们可以在 autoreleasepool 中手动 retain 该对象，以延长它的声明周期(在MRC中）

## main.m 中的 autoreleasepool 的解释
在 iOS 程序的 main.m 文件中有类似这样的语句：
```Objc
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
在面试中问到有关autorealeasepool的知识时，也多半会问一下，这里的 pool 有什么作用？能不能去掉之类...
根据[苹果官方文档](https://developer.apple.com/library/archive/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/TheAppLifeCycle/TheAppLifeCycle.html)：`UIApplicationMain`函数是整个 app 的入口，用来创建 application对象（单例）和application delegate；尽管这个函数有返回值，但是却永远不会返回，当按下Home键时，app 只是切换到了后台状态。

由文档知道，UIApplication会自己创建一个 main runloop，大致可以得到下面的结论：
1. `main.m`中 UIApplicationMain 永远不会返回，只有在系统kill掉整个app时候，系统会把应用占用的全部内存释放出来。
2. 因为 UIApplicationMain 永远不会返回，所以这里的 autoreleasepool 就永远不会进入到drain阶段。
3. 假设真的有变量进入了main.m 中的这个 Pool（而没有被更内层的 Pool 捕获），那么这些内存实际上就是被泄露的，这个autoreleasepool 等于把这种泄露情况进行了隐藏。
4. UIApplication自己会创建 main runlooop，在 Cocoa 的 RunLoop 中实际上也是包含 autoreleasepool 的，因此main.m中的autoreleasepool可以认为是 **没有必要** 的。

另外，在基于 AppKit 框架的 Mac OS 开发中，`main.m`中就是不存在 autoreleasepool 的，这也进一步印证了上面的结论。不过因为不知道更底层的代码，加上苹果文档中不建议修改main.m 文件，所以我们也没有理由把它删掉。但是，删掉之后也不会影响 App 运行，用 `Instruments` 也没有发现内存泄露。

## autoreleasepool的创建与销毁
- 创建 : 运行循环检测到事件并启动后，就会创建自动释放池
- 销毁 : 一次完整的运行循环结束之前，会被销毁

以上，autoreleasepool的创建与销毁都和运行循环（RunLoop）息息相关。

# 参考资料
https://hit-alibaba.github.io/interview/iOS/ObjC-Basic/RunLoop.html
https://blog.ibireme.com/2015/05/18/RunLoop/
