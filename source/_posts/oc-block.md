---
title: Objective-C-Block
date: 2018-05-10 11:21:20
tags: [iOS,Objective-C,Block]
categories: [iOS,Objective-C]
---
# 引言
Apple 在iOS4.0之后推出Block，它本身封装了一段代码并可以将这段代码当做变量，参数，属性，数据类型，函数（匿名），代码块（只有在被调用时才会执行）等等，使用方式灵活，功能十分强大。

# Block的简单使用
## 定义Block
- 无参无返回值
 ``` objc
  void(^block)() = ^(){
   NSLog(@"this is a no param and no return of block");
 };
 block();
 ```
- 有参无返回值
 ``` objc
 void(^block)(NSString * ) = ^(NSString * param){
   NSLog(@"this is a has param and no return of block");
 };
 block(@"param");
 ```
- 有参有返回值
 ``` objc
 NSString *(^block)(NSString * ) = ^(NSString * param){
   NSLog(@"this is a has param and return of block");
   return @"";
 };
 block(@"param");
 ```

## 定义Block的快捷方式
> Block 的定义可以借助键入 inlineBlock 速记.

``` objc
// inlineBlock
returnType (^blockName)(parameterTypes) = ^(parameters) {
   statements
};
```
## Block定义成属性
``` objc
@property (copy, nonatomic) void(^block)();
```
## Block作参数
``` objc
-(void)demo04{
 void(^block)() = ^{
   NSLog(@"block become param");
 };
 [self callbackWith:block];
}

-(void)callbackWith:(void(^)())block{
 //调用外界传入的block
 block();
}
```
## typedef 定义Block
``` objc
// 声明
typedef void(^MyBlock)();
// 使用
MyBlock block01 = ^{
 NSLog(@"block01");
};
block01();
```
- 小结：方便之处在于可以常用类型的Block可以用typedef来定义
# Block与外部变量
## Block内部访问/引用 外部变量
``` objc
 int num = 10;
 NSLog(@"%p %d",&num,num);
 void(^block)() = ^{
   NSLog(@"%p %d",&num,num);
 };
 num = 20;
 NSLog(@"%@ %p %d",block,&num,num);
 block();
```
- `MRC`输出结果：
``` console
018-05-10 12:33:49.178446+0800 BlockDemo[1825:245809] 0x7ffee31dc9dc 10
2018-05-10 12:33:49.178655+0800 BlockDemo[1825:245809] <__NSStackBlock__: 0x7ffee31dc9a8> 0x7ffee31dc9dc 20
2018-05-10 12:33:49.178770+0800 BlockDemo[1825:245809] 0x7ffee31dc9c8 10
```
- `ARC`输出结果：
``` console
2018-05-10 12:36:47.248638+0800 BlockDemo[1887:256355] 0x7ffee517f9dc 10
2018-05-10 12:36:47.248856+0800 BlockDemo[1887:256355] <__NSMallocBlock__: 0x604000259530> 0x7ffee517f9dc 20
2018-05-10 12:36:47.249115+0800 BlockDemo[1887:256355] 0x604000259550 10
```
- 小结
在block中访问外部的变量时，会自动拷贝到内存一份并开辟新的地址，这就是深拷贝

## Block内部修改外部变量
``` objc
__block int num = 10;
 NSLog(@"%p %d",&num,num);
 void(^block)() = ^{
   num = 30;
   NSLog(@"%p %d",&num,num);
 };
 num = 20;
 NSLog(@"%@ %p %d",block,&num,num);
 block();
```
- `MRC`输出结果：
``` console
2018-05-10 12:41:56.490868+0800 BlockDemo[2018:272473] 0x7ffee14519d8 10
2018-05-10 12:41:56.491055+0800 BlockDemo[2018:272473] <__NSStackBlock__: 0x7ffee1451980> 0x7ffee14519d8 20
2018-05-10 12:41:56.491165+0800 BlockDemo[2018:272473] 0x7ffee14519d8 30
```
- `ARC`输出结果：
``` console
2018-05-10 12:43:13.402390+0800 BlockDemo[2047:277009] 0x7ffeea3149d8 10
2018-05-10 12:43:13.402662+0800 BlockDemo[2047:277009] <__NSMallocBlock__: 0x60400024da70> 0x604000034cf8 20
2018-05-10 12:43:13.402777+0800 BlockDemo[2047:277009] 0x604000034cf8 30
```
- 小结
block内修改的外部变量，需要用`__block`修饰.在此后若被block访问修改，变量的内存地址会重新指向拷贝后开辟的新的内存地址.

## 验证一个想法
> 在`ARC`/`MRC`中block内`引用`外部变量地址如何在堆栈中变化，通过将Block自身作为自身的参数传入


``` objc
int num = 10;
   NSLog(@"%p %d",&num,num);
   void(^block)(void(^)()) = ^(void(^block2)()){
       NSLog(@"%@ %p %d",block2,&num,num);
   };
   num = 20;
   NSLog(@"%@ %p %d",block,&num,num);
   //本身作为参数传入并输出指针对象的堆栈
   block(block);
```

  - 1.在`ARC`中block内引用外部变量地址如何在堆栈中变化，输出结果:

``` console
2018-05-10 12:51:29.359318+0800 BlockDemo[2216:302301] 0x7ffee29ab9dc 10
2018-05-10 12:51:29.359563+0800 BlockDemo[2216:302301] <__NSMallocBlock__: 0x600000259230> 0x7ffee29ab9dc 20
2018-05-10 12:51:29.359708+0800 BlockDemo[2216:302301] <__NSMallocBlock__: 0x600000259230> 0x600000259250 10
```
  - 2.在`MRC`中block内引用外部变量地址如何在堆栈中变化，输出结果:

``` console
2018-05-10 12:46:27.120908+0800 BlockDemo[2153:287894] 0x7ffee8a699dc 10
2018-05-10 12:46:27.121106+0800 BlockDemo[2153:287894] <__NSStackBlock__: 0x7ffee8a699a8> 0x7ffee8a699dc 20
2018-05-10 12:46:27.121239+0800 BlockDemo[2153:287894] <__NSStackBlock__: 0x7ffee8a699a8> 0x7ffee8a699c8 10
```
- 小结：
`ARC`下，永远在堆区中。`MRC`下永远在栈区中

# block内存管理（MRC/ARC）
## 函数体
### 当函数体不变时
``` objc
void(^myBlock)() = ^ {
 NSLog(@"hello world");
};
NSLog(@"%@", myBlock);
```
- 输出结果：
``` console
2018-05-09 22:59:07.637768+0800 BlockDemo[3288:930159] <__NSGlobalBlock__: 0x10afa20d8>
```
- 小结：
不引用任何外部变量的 block 保存在全局区 NSGlobalBlock，如果Block没有引用外部变量,那么这个Block的函数体内部包装的代码都不会发生变化,而且执行效率高,保存在全局区;(类似不变的字符串存在常量区)
### 当函数体可变时（即访问修改外部变量时）
``` objc
int i = 10;
void(^myBlock)() = ^ {
 NSLog(@"hello world %d", i);
};
NSLog(@"%@", myBlock);
```
- 输出结果：
``` console
2018-05-09 23:03:01.082413+0800 BlockDemo[3349:946888] <__NSMallocBlock__: 0x600000440090>
```
- 小结：
引用外部变量的 block 保存在 :
`ARC` : 堆区 NSMallocBlock
`MRC` : 栈区 NSStackBlock
因此在定义 block 属性时应该使用 `copy` 关键字，将 block 从栈区复制到堆区

## Block属性
``` objc
//定义 block 属性
@property (nonatomic, copy) void (^demoBlock)();

-(void)blockDemo {
 int i = 10;
 void(^myBlock)() = ^ {
   NSLog(@"hello world %d", i);
 };

 NSLog(@"%@", myBlock);
 // 错误的写法，不会调用 setter 方法,MRC下,无法拷贝到堆区
 //`_demoBlock` = myBlock;
 // 正确的写法，调用 setter 方法，并且对 block 进行 copy
 self.demoBlock = myBlock;
 NSLog(@"%@", self.demoBlock);
}
```
- `MRC`下输出结果：会执行`copy`操作，从栈区拷贝到堆区
``` console
2018-05-10 12:03:17.374375+0800 BlockDemo[1225:138257] <__NSStackBlock__: 0x7ffee68509a8>
2018-05-10 12:03:17.374581+0800 BlockDemo[1225:138257] <__NSMallocBlock__: 0x60000025dca0>
```
- `ARC`下输出结果：不会执行`copy`操作,本身就在堆区
``` console
2018-05-10 12:04:42.891691+0800 BlockDemo[1308:145791] <__NSMallocBlock__: 0x6040002471d0>
2018-05-10 12:04:42.891902+0800 BlockDemo[1308:145791] <__NSMallocBlock__: 0x6040002471d0>
```
> **注** : 为了避免程序员的麻烦,在 ARC 中,定义了引用外部变量的 block,系统默认都是在堆区的!

## copy关键字的探讨
### ARC （为什么可以用`copy / strong`）
``` objc
- (void)blockDemo
{
 int num = 10;
 void (^task)() = ^ {
   NSLog(@"%d",num);
 };
 // ARC : 堆区 ==> __NSMallocBlock__
 // ARC环境下,属性也是强引用,同时会copy
 // self.task = task;
 // ARC环境下,成员变量也是强引用,同时会copy
 `_task` = task;
 NSLog(@"%@ -- %@",task,`_task`);
}
```
- 输出结果：
``` console
2018-05-09 23:14:43.139413+0800 BlockDemo[3468:994925] <__NSMallocBlock__: 0x60400024ea60> -- <__NSMallocBlock__: 0x60400024ea60>
```
- 小结：
在`ARC`环境下,上述block本来就保存在堆区,给属性赋值的时候,调用`setter`方法时,只会给一个引用.故`ARC`下使用strong和copy的效果是一模一样的

### MRC（为什么必须用`copy`）
``` objc
-(void)blockDemo
{
 int num = 10;
 void (^task)() = ^ {
   NSLog(@"%d",num);
 };
 // MRC : 栈区 ==> __NSStackBlock__
 // 这个赋值过程会copy,也会引用计数+1
 //    self.task = task;
 // 这个赋值过程不会copy,仅仅是引用计数+1,内存依然在栈区
 `_task` = task;
 NSLog(@"%@ -- %@",task,`_task`);
}
```
- 输出结果：
``` console
2018-05-09 23:17:25.352415+0800 BlockDemo[3528:1006735] <__NSStackBlock__: 0x7ffee98cc9a8> -- <__NSStackBlock__: 0x7ffee98cc9a8>
```
- 小结：
在`MRC`环境下,调用成员变量进行赋值,仅仅是引用计数加1;不会进行`copy`操作. 所以内存依然在栈区.

### 面试时如何回答？
当Block被引入到OC时 ，OC仍是MRC的管理内存模式 ，在MRC管理模式中，Block处于栈区，超出作用域就会被销毁 ，如果用一个属性来全局的记录这个Block，就必须满足两个条件：
1.这个属性必须对Block强引用
2.需要把Block拷贝到堆区
要满足以上两个条件就需要使用`copy`修饰符

# Block的循环引用
## 满足什么条件会出现循环应用
- block和外部变量`互相强引用`导致出现循环引用,内存不能正常释放
- **坑**：不要在Block的内部使用成员变量(`_name`)，而要尽量使用属性，因为看不到`self`字段，会造成如果出现循环引用不容易发现的问题.
> **注**：不是所有的 self. 都会出现循环引用 —— block 执行完毕就销毁，例如 UIView 的动画代码

## 如何解决循环引用
``` objc
// 使用 __weak 修饰符,定义一个弱引用的对象
__weak typeof(self) weakSelf = self;
```
