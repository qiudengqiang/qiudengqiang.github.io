---
title: iOS-多线程之NSOperation
date: 2018-04-29 17:14:05
tags: [iOS,Objective-C,Thread,NSOperation]
categories: [iOS,Objective-C]
---

# NSOperation
apple提供的多线程解决方案`NSOperation`是一个表示与单个任务关联的代码和数据的抽象类；因为是一个抽象类，所以不能直接使用，需要使用它的两个子类(`NSInvocationOperation` or `NSBlockOperation`) 去执行实际的操作任务；同样我们也可以通过自定义NSOperation。通常将操作添加到操作队列（`NSOperationQueue`类的实例）来执行操作。其实`NSOperation`就是对`GCD`的封装，相对于GCD来说可控性更强，并且可以加入操作依赖（`addDependency:` and `removeDependency`）。

## NSInvocationOperation
### start
```Objc
- (void)demo1 {
    NSInvocationOperation * op = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(oprationTask:) object:@"InvocationOperation"];
    [op start];
}

- (void)operationTask:(id)param {
    NSLog(@"%@",[NSThread currentThread]);
}
```
输出结果：发现在主线程中输出的结果，但`start`方法是在`当前线程`中执行的
``` console
2018-05-19 14:45:07.956558+0800 NSOperation练习[1324:290009] <NSThread: 0x604000078000>{number = 1, name = main}
```
### 将操作添加到队列
```Objc
- (void)demo2 {
	NSInvocationOperation * op = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(oprationTask:) object:@"InvocationOperation"];
	NSOperationQueue * queue = [[NSOperationQueue alloc]init];
	[queue addOperation:op];
}
```
输出结果：开启子线程异步执行
``` console
2018-05-19 14:50:53.013010+0800 NSOperation练习[1408:312296] <NSThread: 0x60400046b600>{number = 3, name = (null)}
```

## NSBlockOperation (使用较多)
### start
```Objc
- (void)demo4 {
    NSBlockOperation * op = [NSBlockOperation blockOperationWithBlock:^{
       NSLog(@"%@",[NSThread currentThread]);
    }];
    [op start];
}
```
输出结果：`start`方法是在`当前线程`中执行
```console
2018-05-19 15:06:11.837276+0800 NSOperation练习[1661:375754] <NSThread: 0x6000000745c0>{number = 1, name = main}
```
### 将操作添加到队列
```Objc
- (void)demo5 {
    NSBlockOperation * op = [NSBlockOperation blockOperationWithBlock:^{
       NSLog(@"%@",[NSThread currentThread]);
    }];
    NSOperationQueue * queue = [[NSOperationQueue alloc]init];
    [queue addOperation:op];
}
```
输出结果：开启子线程异步执行
```console
2018-05-19 15:10:23.974301+0800 NSOperation练习[1720:389949] <NSThread: 0x60400027ea40>{number = 3, name = (null)}
```
### 执行块
```Objc
- (void)demo6 {
    NSBlockOperation * op = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"block 2 %@", [NSThread currentThread]);
    }];

    [op addExecutionBlock:^{
        NSLog(@"block 1 %@", [NSThread currentThread]);
    }];


    NSOperationQueue * queue = [[NSOperationQueue alloc]init];
    [queue addOperation:op];

    NSLog(@"%@", op.executionBlocks);
}
```
输出结果：执行块和操作享有共同的属性设置，异步执行
```console
2018-05-19 15:20:27.839119+0800 NSOperation练习[1882:431125] (
    "<__NSGlobalBlock__: 0x1086f9080>",
    "<__NSGlobalBlock__: 0x1086f90c0>"
)
2018-05-19 15:20:27.839131+0800 NSOperation练习[1882:431333] block 2 <NSThread: 0x60000027bbc0>{number = 3, name = (null)}
2018-05-19 15:20:27.839133+0800 NSOperation练习[1882:431330] block 1 <NSThread: 0x604000466fc0>{number = 4, name = (null)}
```
### 线程间通讯
``` Objc
- (void)demo7{
	[[NSOperationQueue new] addOperationWithBlock:^{
		NSLog(@"consuming time：%@",[NSThread currentThread]);
		/// 回到主线程刷新UI
		[[NSOperationQueue mainQueue] addOperationWithBlock:^{
			NSLog(@"refresh ui: %@",[NSThread currentThread]);
		}];
	}];
}
```
### 监听block执行完成
```Objc
- (void)demo13 {
    NSBlockOperation * op = [NSBlockOperation blockOperationWithBlock:^{
        for (NSInteger i=0; i<5; ++i) {
            NSLog(@"%zd %@",i,[NSThread currentThread]);
        }
    }];
    //设置监听操作执行完成的block，必须要在把操作添加到队列之前设置
    [op setCompletionBlock:^{
        NSLog(@"setCompletionBlock  %@",[NSThread currentThread]);
    }];
    [[NSOperationQueue new]addOperation:op];
}
```
输出结果：
```console
2018-05-19 16:12:00.170053+0800 NSOperation练习[2607:585084] 0 <NSThread: 0x60000027bd80>{number = 3, name = (null)}
2018-05-19 16:12:00.170267+0800 NSOperation练习[2607:585084] 1 <NSThread: 0x60000027bd80>{number = 3, name = (null)}
2018-05-19 16:12:00.170410+0800 NSOperation练习[2607:585084] 2 <NSThread: 0x60000027bd80>{number = 3, name = (null)}
2018-05-19 16:12:00.171319+0800 NSOperation练习[2607:585084] 3 <NSThread: 0x60000027bd80>{number = 3, name = (null)}
2018-05-19 16:12:00.171538+0800 NSOperation练习[2607:585084] 4 <NSThread: 0x60000027bd80>{number = 3, name = (null)}
2018-05-19 16:12:00.171816+0800 NSOperation练习[2607:585083] setCompletionBlock  <NSThread: 0x60000027bd40>{number = 4, name = (null)}
```
## 自定义NSOperation

### 自定义类继承NSOperation
```Objc
@interface CustomOperation : NSOperation

@end
```
```Objc
- (void)viewDidLoad {
    [super viewDidLoad];
    NSOperationQueue * queue = [[NSOperationQueue alloc]init];
    DownloadOperation * op = [[DownloadOperation alloc]init];
    [queue addOperation:op];
}
```

### 重写main方法
任何操作在执行时,首先会调用start方法,start方法会更新操作的状态(过滤操作)；经start方法过滤之后，只有正常可执行的操作，就会调用这个main方法，重写操作的入口方法(main方法)，就可以在这个方法里面指定操作执行的任务。`main`方法默认在子线程中异步执行
```Objc
- (void)main
{
		//在这个方法中做想要做的操作，即自定义 NSOperation的目的
    NSLog(@"%@",self.URLString,[NSThread currentThread]);
}
```













# NSOperationQueue
![](http://p7xd6yrmx.bkt.clouddn.com/%E6%93%8D%E4%BD%9C%E6%B7%BB%E5%8A%A0%E5%88%B0%E9%98%9F%E5%88%97.png)
## 使用
`NSOperationQueue`只有一种类型，就是并发队列。在开发使用到`NSOperationQueue`时，建议将其定义为`全局`队列。
```Objc
// 定义为属性
@property (nonatomic,strong) NSOperationQueue *queue;

- (NSOperationQueue * )queue
{
    if (self.queue == nil) {
        self.queue = [[NSOperationQueue alloc] init];
    }
    return self.queue;
}
```
## 最大并发数
![](http://p7xd6yrmx.bkt.clouddn.com/%E9%98%9F%E5%88%97%E6%9C%80%E5%A4%A7%E5%B9%B6%E5%8F%91%E6%95%B0.png)
`maxConcurrentOperationCount`是队列的一个属性，可以限制队列`同时执行`的任务数量，从而间接的控制了线程数量(线程可以复用)，但队列最大并发数不是线程数。如果队列最大并发数设置为`1`，那么队列实际上就是一个串行队列了。
```Objc
	// 设置最大并发数 : 每次只能调度两个操作执行
	queue.maxConcurrentOperationCount = 2;
```

## 验证队列的并发性
### NSOperationQueue & NSInvocationOperation
```Objc
- (void)demo8 {
    NSOperationQueue * queue = [[NSOperationQueue alloc]init];
    for (NSInteger i=0; i<10; ++i) {
        NSInvocationOperation * op = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(operationTask:) object:@(i)];
        [queue addOperation:op];
    }
}
```
输出结果：会开启多条线程,不是顺序执行.与GCD中并发队列&异步执行效果一样，说明`NSOperationQueue`默认是并发执行
``` console
2018-05-19 15:01:47.478515+0800 NSOperation练习[1593:358939] <NSThread: 0x60400047a580>{number = 3, name = (null)}
2018-05-19 15:01:47.478518+0800 NSOperation练习[1593:358938] <NSThread: 0x600000263680>{number = 4, name = (null)}
2018-05-19 15:01:47.478519+0800 NSOperation练习[1593:358936] <NSThread: 0x60400047a500>{number = 6, name = (null)}
2018-05-19 15:01:47.478572+0800 NSOperation练习[1593:358935] <NSThread: 0x60400047a640>{number = 5, name = (null)}
2018-05-19 15:01:47.478740+0800 NSOperation练习[1593:358974] <NSThread: 0x600000263a40>{number = 7, name = (null)}
2018-05-19 15:01:47.478881+0800 NSOperation练习[1593:358938] <NSThread: 0x600000263680>{number = 4, name = (null)}
2018-05-19 15:01:47.478902+0800 NSOperation练习[1593:358939] <NSThread: 0x60400047a580>{number = 3, name = (null)}
2018-05-19 15:01:47.479084+0800 NSOperation练习[1593:358975] <NSThread: 0x6000002636c0>{number = 8, name = (null)}
2018-05-19 15:01:47.479096+0800 NSOperation练习[1593:358976] <NSThread: 0x600000263d00>{number = 10, name = (null)}
2018-05-19 15:01:47.479128+0800 NSOperation练习[1593:358977] <NSThread: 0x600000263dc0>{number = 9, name = (null)}
```
### NSOperationQueue & NSBlockOperation
```Objc
- (void)demo10 {
    NSOperationQueue * queue = [[NSOperationQueue alloc]init];
    for (NSInteger i=0; i<10; ++i) {
        NSBlockOperation * op = [NSBlockOperation blockOperationWithBlock:^{
            NSLog(@"%@",[NSThread currentThread]);
        }];
        [queue addOperation:op];
    }
}
```
输出结果：会开启多条线程,不是顺序执行.与GCD中并发队列&异步执行效果一样，说明`NSOperationQueue`默认是并发执行
``` console
2018-05-19 15:26:52.141566+0800 NSOperation练习[1984:456384] <NSThread: 0x60000027ef80>{number = 5, name = (null)}
2018-05-19 15:26:52.141566+0800 NSOperation练习[1984:456319] <NSThread: 0x604000468dc0>{number = 4, name = (null)}
2018-05-19 15:26:52.141569+0800 NSOperation练习[1984:456314] <NSThread: 0x60000027f040>{number = 3, name = (null)}
2018-05-19 15:26:52.141641+0800 NSOperation练习[1984:456313] <NSThread: 0x604000469d80>{number = 6, name = (null)}
2018-05-19 15:26:52.141661+0800 NSOperation练习[1984:456385] <NSThread: 0x60000027ee80>{number = 8, name = (null)}
2018-05-19 15:26:52.141676+0800 NSOperation练习[1984:456318] <NSThread: 0x604000469c40>{number = 7, name = (null)}
2018-05-19 15:26:52.141928+0800 NSOperation练习[1984:456384] <NSThread: 0x60000027ef80>{number = 5, name = (null)}
2018-05-19 15:26:52.141945+0800 NSOperation练习[1984:456319] <NSThread: 0x604000468dc0>{number = 4, name = (null)}
2018-05-19 15:26:52.141954+0800 NSOperation练习[1984:456314] <NSThread: 0x60000027f040>{number = 3, name = (null)}
2018-05-19 15:26:52.142048+0800 NSOperation练习[1984:456386] <NSThread: 0x60000027f380>{number = 9, name = (null)}
```
## 队列暂停继续和取消全部
### isSuspended:
暂停和继续队列的属性；YES代表暂停队列，NO代表恢复队列。将队列挂起之后，队列中的操作就不会被调度，但是正在执行的操作不受影响
`operationCount`: 操作计数,没有执行和没有执行完的操作,都会计算在操作计数之内
 > 注意 : 如果先暂停队列,再添加操作到队列,队列不会调度操作执行.所以在暂停队列之前要判断队列中有没有任务.如果没有操作就不暂停队列.

```Objc
#pragma mark - 演示队列的暂停
- (IBAction)pause:(id)sender
{
    // 暂停队列之前判断队列中有无操作
    if (self.queue.operationCount == 0) {
        return;
    }

    // 暂停队列
    self.queue.suspended = YES;
    NSLog(@"pause %zd",self.queue.operationCount);
}
```
### cancelAllOperations:
取消队列中的全部操作；旦调用的 `cancelAllOperations`方法，队列中的操作，都会被移除，正在执行的操作除外；正在执行的操作取消不了，如果要取消，需要自定义NSOperation；队列取消全部操作时，会有一定的时间延迟。
```Objc
- (IBAction)cancelAll:(id)sender
{
    [self.queue cancelAllOperations];
    NSLog(@"cancelAll %zd",self.queue.operationCount);
}
```
# qualityOfService
服务质量的枚举：
```Objc
typedef NS_ENUM(NSInteger, NSQualityOfService) {
    NSQualityOfServiceUserInteractive = 0x21,
    NSQualityOfServiceUserInitiated = 0x19,
    NSQualityOfServiceUtility = 0x11,
    NSQualityOfServiceBackground = 0x09,
    NSQualityOfServiceDefault = -1
} API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
```
## operation
让队列里面的操作有更多的机会被队列调度执行，类似于线程优先级
```Objc
- (void)demo11 {
    NSBlockOperation * op1 = [NSBlockOperation blockOperationWithBlock:^{
       NSLog(@"op1 block %@",[NSThread currentThread]);
    }];
    for (NSInteger i=0; i<5; ++i) {
        [op1 addExecutionBlock:^{
           NSLog(@"opt1 %@",[NSThread currentThread]);
        }];
    }
    op1.qualityOfService = NSQualityOfServiceBackground;

    NSBlockOperation * op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"op2 block %@",[NSThread currentThread]);
    }];
    for (NSInteger i=0; i<5; ++i) {
        [op2 addExecutionBlock:^{
            NSLog(@"opt2 %@",[NSThread currentThread]);
        }];
    }

    op2.qualityOfService = NSQualityOfServiceUserInteractive;

    NSOperationQueue * queue = [[NSOperationQueue alloc]init];
    [queue addOperations:@[op1,op2] waitUntilFinished:false];
}
```
输出结果：op2优先于op1执行
```console
2018-05-19 16:05:46.292652+0800 NSOperation练习[2511:564443] op2 block <NSThread: 0x600000278580>{number = 4, name = (null)}
2018-05-19 16:05:46.292656+0800 NSOperation练习[2511:564453] opt2 <NSThread: 0x600000278680>{number = 7, name = (null)}
2018-05-19 16:05:46.292654+0800 NSOperation练习[2511:564445] opt2 <NSThread: 0x604000470ac0>{number = 6, name = (null)}
2018-05-19 16:05:46.292709+0800 NSOperation练习[2511:564454] opt2 <NSThread: 0x604000470c40>{number = 8, name = (null)}
2018-05-19 16:05:46.292933+0800 NSOperation练习[2511:564453] opt2 <NSThread: 0x600000278680>{number = 7, name = (null)}
2018-05-19 16:05:46.292937+0800 NSOperation练习[2511:564443] opt2 <NSThread: 0x600000278580>{number = 4, name = (null)}
2018-05-19 16:05:46.292944+0800 NSOperation练习[2511:564446] opt1 <NSThread: 0x604000470b00>{number = 5, name = (null)}
2018-05-19 16:05:46.292960+0800 NSOperation练习[2511:564444] op1 block <NSThread: 0x604000470b80>{number = 3, name = (null)}
2018-05-19 16:05:46.293083+0800 NSOperation练习[2511:564454] opt1 <NSThread: 0x604000470c40>{number = 8, name = (null)}
2018-05-19 16:05:46.293089+0800 NSOperation练习[2511:564453] opt1 <NSThread: 0x600000278680>{number = 7, name = (null)}
2018-05-19 16:05:46.294076+0800 NSOperation练习[2511:564446] opt1 <NSThread: 0x604000470b00>{number = 5, name = (null)}
2018-05-19 16:05:46.294207+0800 NSOperation练习[2511:564444] opt1 <NSThread: 0x604000470b80>{number = 3, name = (null)}
```
## queue
```Objc
- (void)demo12 {
    NSOperationQueue * q1 = [[NSOperationQueue alloc]init];
    NSOperationQueue * q2 = [[NSOperationQueue alloc]init];

    q1.qualityOfService = NSQualityOfServiceBackground;
    q2.qualityOfService = NSQualityOfServiceUserInteractive;

    for (NSInteger i=0; i<5; ++i) {
        [q1 addOperationWithBlock:^{
           NSLog(@"q1");
        }];
        [q2 addOperationWithBlock:^{
            NSLog(@"q2");
        }];
    }
}
```
输出结果：q2优先于q1执行
```console
2018-05-19 16:05:11.965991+0800 NSOperation练习[2487:561497] q2 <NSThread: 0x604000274e00>{number = 4, name = (null)}
2018-05-19 16:05:11.965998+0800 NSOperation练习[2487:561496] q2 <NSThread: 0x600000275740>{number = 3, name = (null)}
2018-05-19 16:05:11.966106+0800 NSOperation练习[2487:561516] q2 <NSThread: 0x600000275940>{number = 5, name = (null)}
2018-05-19 16:05:11.966106+0800 NSOperation练习[2487:561517] q2 <NSThread: 0x604000274f80>{number = 6, name = (null)}
2018-05-19 16:05:11.966316+0800 NSOperation练习[2487:561516] q2 <NSThread: 0x600000275940>{number = 5, name = (null)}
2018-05-19 16:05:11.968284+0800 NSOperation练习[2487:561494] q1 <NSThread: 0x600000263840>{number = 7, name = (null)}
2018-05-19 16:05:11.968303+0800 NSOperation练习[2487:561495] q1 <NSThread: 0x604000275680>{number = 8, name = (null)}
2018-05-19 16:05:11.968376+0800 NSOperation练习[2487:561517] q1 <NSThread: 0x604000274f80>{number = 6, name = (null)}
2018-05-19 16:05:11.968409+0800 NSOperation练习[2487:561496] q1 <NSThread: 0x600000275740>{number = 3, name = (null)}
2018-05-19 16:05:11.969839+0800 NSOperation练习[2487:561494] q1 <NSThread: 0x600000263840>{number = 7, name = (null)}
```
# 支持KVO的属性
```
isCancelled - read-only   //是否取消

isAsynchronous - read-only //是否异步

isExecuting - read-only  //是否正在执行

isFinished - read-only	//是否结束

isReady - read-only	//是否就绪

dependencies - read-only	//依赖的其他的操作

queuePriority - readable and writable	//队列优先级

completionBlock - readable and writable	//结束回调
```

# 操作间依赖
## 需求实例
场景：用户需要先登录->付费->下载->通知用户
```Objc
- (void)dependency
{
    NSBlockOperation * op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"login %@",[NSThread currentThread]);
    }];

    NSBlockOperation * op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"pay %@",[NSThread currentThread]);
    }];

    NSBlockOperation * op3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"download %@",[NSThread currentThread]);
    }];

    NSBlockOperation * op4 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"notice user %@",[NSThread currentThread]);
    }];
}
```
## 添加依赖
```Objc
[op2 addDependency:op1]; // 操作2依赖于操作1
[op3 addDependency:op2]; // 操作3依赖于操作2
[op4 addDependency:op3]; // 操作4依赖于操作3

// waitUntilFinished : 是否等到指定的操作执行结束再执行后面的代码
[self.queue addOperations:@[op1,op2,op3] waitUntilFinished:NO];

// 通知用户的操作在主线程中执行
[[NSOperationQueue mainQueue] addOperation:op4];

// 验证 waitUntilFinished
NSLog(@"end");
```
## 小结
- 不能循环建立操作间依赖关系.否则,队列不调度操作执行
- 操作间可以跨队列建立依赖关系
- 要将操作间的依赖建立好了之后,再添加到队列中（先建立操作依赖关系，再把操作添加到队列）

# NSOperation和GCD的区别
![](http://p7xd6yrmx.bkt.clouddn.com/GCD%E5%92%8COP%E7%9A%84%E5%85%B3%E7%B3%BB%E5%9B%BE.png)
## GCD
GCD `iOS 4.0` 推出，针对多核处理器的并发技术。GCD属于C语言的框架。将任务封装在block中，如果要停止已经加入 队列(queue) 的 任务(block) 需要写复杂的代码。只能设置队列的优先级不能设置任务的优先级。
### 高级功能
- barrier
- once
- after
- group

## NSOperation
NSOperation `iOS 2.0` 推出，但在苹果推出 GCD 之后，对NSOperation的底层全部重写。NSOperation属于OC 框架，更加面向对象，底层是对 GCD 的封装。支持取消掉队列中的任务(正在执行的除外)，还可以设置队列中每个操作的优先级
### 高级功能
- 最大操作并发数(GCD不好做)
- 继续/暂停/全部取消
- 跨队列设置操作的依赖关系
