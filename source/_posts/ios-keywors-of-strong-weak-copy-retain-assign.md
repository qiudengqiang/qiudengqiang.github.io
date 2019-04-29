---
title: strong、weak、copy、retain和assign的区别
date: 2016-02-18 14:12:00
tags: [iOS,keywords]
categories: [iOS,Objc]
---

## assign：
`assign`一般用来修饰基本的数据类型，包括基础数据类型（NSInteger,CGFloat）和C语言数据类型（int,float,double,char）等等。因为assign声明的属性，是不会增加引用计数的，也就是说声明的属性释放后也就没有了，及时其他对象引用了它也不会保留，只会造成crash。但是及时被释放，指针却还在，成为了`野指针`，如果新的对象被分配到了这个内存地址上，又会造成crash。所以一般只用来声明基本的数据类型，因为它们会被分配到`栈区`上，而栈取由系统`自动管理`，不会造成野指针。

## retain：
与assign相对，我们要解决`对象`被其他对象引用后释放造成的问题，就需要用`retain`来声明。使用retain声明的对象会`更改引用计数`，每次被引用，引用计数都会`+1`，释放后就会`-1`。即使这个对象本身被释放了，只要还有对象在引用它，该对象就会`仍然持有`，不会出现任何问题。并且只有当引用`计数为0时`，就会被`dealloc`析构函数回收进内存。

## copy：
最常见到的copy的声明使用是 NSString 等。copy与retain的`区别`在于：retain是拷贝内存指针地址，而copy是拷贝对象本身；也就是说retain是`浅复制`，copy是`深复制`；如果是浅复制，当修改对象值时，都会被修改，而深复制不会。之所以在`NSString`这一类有`可变类型对象`的身上使用`copy`关键字，是因为他们有可能和对应的可变类型如 `NSMutableString` 之间进行赋值操作，为了防止内容被改变，使用copy去深复制一份。copy工作由copy方法执行，此属性只对那些实现了 `NSCopying` 协议的`对象类型`有效。

## weak：
weak是由ARC新引入的对象变量属性，weak类似于assign，叫`弱引用`，也是不增加引用计数，不同在于week指向对象类型时，当对象被释放会指向nil，而assign则会造成野指针。一般只有在防止循环引用时候使用；比如父类引用了子类，子类又引用父类；IBOutlet、Delegate等一般就是使用week，这是因为他们可能会在类外部被调用，防止循环应用。

## strong：
strong也是由ARC新引入的对象变量属性，在ARC下,用strong代替了retain，叫`强引用`，会增加引用计数。，所有的局部变量代码中我们声明的变量默认都是强引用，不需要再额外使用`__strong`来修饰。

## 什么时候用`stong`/`weak`
- 根视图和父视图需要使用`strong`; 子视图使用`weak`
- 没有强指针指向的对象使用`strong`; 有强指针指向的可以可以`weak`

## `__strong,__weak,__unsafe_unretained,__autoreleasing` 的含义
在ARC情况下，对象类型的变量将有所有权修饰符
`__strong`: 是缺省的关键词。
`__weak`: 声明了一个可以自动nil化的引用。
`__unsafe_unretained`: 声明一个弱引用，但是不会自动nil化，也就是说，如果所指向的内存区域被释放了，这个指针就是一个野指针了。
`__autoreleasing`: 用来修饰一个函数的参数，这个参数在函数返回的时候会被自动释放。

`ARC`声明属性时，对于`基本数据类型`默认关键字是 （atomic,readwrite,assign）
`ARC`声明属性时，对于普通的`OC对象`默认关键字是 （atomic,readwrite,strong）
- 示例
``` Objc
@property (nonatomic) int supportOrientation;           默认是assign，因为是基础数据类型，必须是assign
@property (readonly) UIImage* rightImage;                                     默认是atomic
@property (nonatomic) bolo_BasePlayerControlView* ctrlView;            默认是strong
@property (nonatomic, weak) id<WatchVideoDetailDelegate> delegate;         代理使用weak
```

## 扩展：
LLVM官网给出的一些示意，ARC里也可以使用retain等关键字
```
assign implies __unsafe_unretained ownership.
copy implies __strong ownership, as well as the usual behavior of copy semantics on the setter.
retain implies __strong ownership.
strong implies __strong ownership.
unsafe_unretained implies __unsafe_unretained ownership.
weak implies __weak ownership.
```
assign 等同于unsafe_retained
copy的作用和MRC一样，同时又有strong的效果
retain等同于strong
weak和unsafe_unretained的区别在于：weak降被释放指针赋值为nil，而unsafe_unretained则会成为野指针
