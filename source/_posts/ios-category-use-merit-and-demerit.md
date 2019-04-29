---
title: iOS - Category 的使用和优缺点
date: 2016-05-16 14:30:47
tags: [iOS,Category]
categories: [iOS,Objc]
---
## 什么是Category?
分类就是对一个类的功能进行扩展，让这个类能够适应不同情况的需求；在实际开发中我们都会对系统的一些常用类进行扩展，例如：NSString,Button,Label等；简单来说类别是一种为现有的类添加新方法的方式。利用OC的动态运行时分配机制，category提供了一种比继承更为简洁的方法来对类进行扩展，无需创建对象的子类就能为现有的类添加新的方法，category可以为任何已经存在的类添加方法，包括系统的框架UIKit等。

## Category的优点：
- 可以将类的实现分散到多个不同的文件或者不同的框架中，方便代码的管理；也可以对框架提供类的扩展，把不同的功能组织到不同的category里，从而按需加载想要的category。
- 创建对私有方法的前向引用：如果其他类中的方法未实现时，或者在访问该类私有方法时编译器报错时；在类别中声明这些方法（不必提供方法实现）从而绕过编译器不会再产生警告或者错误。
- 向对象添加非正式协议：创建一个NSObject的类别成为“创建一个非正式协议”，因为可以作为任何类的委托对象使用（声明私有方法）。

apple的SDK中就大面积的使用了category这一特性。比如UIKit中的UIView。apple把不同的功能API进行了分类，这些分类包括UIViewGeometry、UIViewHierarchy、UIViewRendering等。
不过除了apple推荐的使用场景，广大开发者脑洞大开，还衍生出了category的其他几个使用场景：
	1. 模拟多继承（另外可以模拟多继承的还有protocol）
	2. 把framework的私有方法公开

## Category的局限性：
- category只能给某个已有的类扩充方法，不能扩充成员变量。
- category中也可以添加属性，只不过@property只会生成`setter`和`getter`的声明，不会生成实现以及成员变量。
- 如果category中的方法和类中原有的方法同名，运行时会优先调用category中的方法。也就是，category中的方法会`覆盖`掉类中原有的方法。所以开发中尽量保证不要让分类中的方法和原有类中的方法名相同;避免出现这种情况的解决方案是给分类的方法名统一添加前缀。比如`category_xxx`。
- 如果多个category中存在同名的方法，运行时到底调用那个方法由编译器决定，最后一个参与编译的方法会被调用。

如下：给UIView添加两个category（one和two）并且给这两个分类都添加了名为log的方法
- UIView+One
``` objc
#import "UIView+One.h"

@implementation UIView (One)
- (void)log {
    NSLog(@"调用One分类的方法");
}
@end
```
- UIView+Two
``` objc
#import "UIView+Two.h"

@implementation UIView (Two)
- (void)log {
    NSLog(@"调用One分类的方法");
}
@end
```
- 在UIViewController中引用这两个分类的头文件并调用log方法
```objc
#import "UIView+One.h"
#import "UIView+Two.h"

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    UIView * view = [UIView new];
    [view log];
}
@end
```

- 当编译顺序如下图时输出：
![](/images/tech/compile_last_one.png)
``` console
2018-05-18 15:23:13.379081+0800 CategoryDemo[2373:700484] 调用One分类的方法
```
- 将UIView+One.m移动到UIView+Two.m上面，编译顺序如下图时输出：
![](/images/tech/compile_last_two.png)
``` console
2018-05-18 15:27:25.008682+0800 CategoryDemo[2441:715950] 调用Two分类的方法
```

## 调用优先级
Category->本类->父类

## 为什么Category不能添加成员变量？
Objective-C的类是由`Class`类型来表示的，它实际上是一个指向`objc_class`结构体的指针
```C
typedef struct objc_class * Class;
```
objc_class结构体的定义如下：
```
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                       OBJC2_UNAVAILABLE;  // 父类
    const char *name                        OBJC2_UNAVAILABLE;  // 类名
    long version                            OBJC2_UNAVAILABLE;  // 类的版本信息，默认为0
    long info                               OBJC2_UNAVAILABLE;  // 类信息，供运行期使用的一些位标识
    long instance_size                      OBJC2_UNAVAILABLE;  // 该类的实例变量大小
    struct objc_ivar_list *ivars            OBJC2_UNAVAILABLE;  // 该类的成员变量链表
    struct objc_method_list **methodLists   OBJC2_UNAVAILABLE;  // 方法定义的链表
    struct objc_cache *cache                OBJC2_UNAVAILABLE;  // 方法缓存
    struct objc_protocol_list *protocols    OBJC2_UNAVAILABLE;  // 协议链表
#endif
} OBJC2_UNAVAILABLE;
```
在上面的`objc_class`结构体中，`ivars`是`objc_ivar_list`（成员变量列表）指针；`methodLists`是指向`objc_method_list`指针的指针。
在`Runtime`中，`objc_class`的大小是`固定`的，不可能往这个结构体中添加数据，只能修改。所以`ivars`指向的是一个固定区域，只能修改成员变量的值，不能增加成员变量的个数。
`methodLists`是一个`二维数组`，所以可以修改`*methodLists`的值来增加成员变量方法，虽然没办法扩展`methodLists`指向的内存区域，却可以改变这个内存区域的值（存储的是指针），因此可以动态添加方法，不能添加成员变量。

## Category中能添加属性吗？
Category中不能直接添加成员变量，那么可以添加属性吗？
需要从Category的结构体开始分析：
```
typedef struct category_t {
    const char *name;  //类的名字
    classref_t cls;  //类
    struct method_list_t *instanceMethods;  //category中所有给类添加的实例方法的列表
    struct method_list_t *classMethods;  //category中所有添加的类方法的列表
    struct protocol_list_t *protocols;  //category实现的所有协议的列表
    struct property_list_t *instanceProperties;  //category中添加的所有属性
} category_t;
```
从Category的结构体定义`也`可以看出:Category可以添加`实例方法`、`类方法`、`协议`、`属性`，但不能添加`成员变量`（实例变量）

### 为什么网上很多人说Category不可以添加属性?
实际上，category是可以添加属性的，同样可以使用`@property`，但是不会生成带下划线的成员变量也不会生成属性getter和setter方法的实现。所以，尽管添加了属性，也无法使用点语法调用getter和setter方法（实际上，点语法是可以写的，只不过在运行时调用到这个方法的时候会报Unrecognised selector send to instance的错误），但可以使用Runtime去实现Category为已有的类添加新的属性并生成getter和setter方法
### 为什么不能为Category手动添加一个下划线开头的成员变量
成员变量是一个类的东西，而分类本身就不是一个类，它并没有自己的isa指针，分类本来就是OC里通过运行时动态的为一个类添加属性和方法等，不是一个真正的类无法添加成员变量。

>可以使用Runtime技术中的关联对象可以为类别添加属性

## 两点注意：
1. 当category中的方法和原类中的方法同名时，category中的方法并没有完全替换掉原类中的方法，也就是说如果category和原类中都有一个methodA方法，那么category附加完成之后，类的方法里面会有两个methodA，实际上category的方法只是被放到新方法列表的前面，而原来类的方法只是被放到了新方法列表的后面，这也就是通常说的`覆盖同名方法`；这是因为运行时在查找方法的时候是顺着方法列表顺序查找的，它只要已找到对应名字的方法，就直接调用不会再往后面找了。
2. 由于category的实现原理，和Objc的动态绑定有很强的关系，所以实际上类的扩展比较占用启动时间，因尽量合并一些在工程，架构上没有太大意义的扩展，会对启动有一定的优化作用。

## 扩展：成员变量和属性的区别？
@property声明的属性默认会生成一个以下划线开头的成员变量，同事也会生成getter/setter方法。但这仅仅是在iOS5之后，苹果才推出的一个机制。在一些比较老的项目经常可以看到一大括号里面定义了成员变量，同时用了@property声明，而且还在@implementation中使用了@synthesize方法。
``` Objc
@interface ViewController ()
{
   // 1.声明成员变量
    NSString * myString;  
 }
 //2.在用@property
@property(nonatomic, copy) NSString * myString;  
@end

@implementation ViewController
//3.最后在@implementation中用synthesize生成set方法
@synthesize myString;   
@end
```
实际上，发生这种状况的根本原因是苹果将默认编译器从GCC转换为LLVM（low level virtual machine）后，才不再需要为属性声明实例变量了。
在没有更改之前，属性的正常写法需要 `成员变量 + @property + @synthesize成员变量`三个步骤
如果我们只写成员变量+@property
```Objc
@interface GBViewController :UIViewController
{
    NSString * myString;
}
@property (nonatomic, strong) NSString * myString;
@end
```
这时，编译器会警告：
```bash
Autosynthesized property 'myString' will use synthesized instance variable '_myString', not existing instance variable 'myString'
```
但更换为`LLVM`之后，编译器在编译过程中发现没有生成实例变量时，就会生成一个下划线开头的实例变量。因此现在我们不必在声明一个实例变量（**注意**：是不必要，不是不可以）
对于`@synthesize`我们要明白，`@synthesize`不仅可以帮助生成setter/getter方法；同时还有一个作用就是可以指定与属性对应的实例变量
```Objc
@synthesize myString = _xxx；
```
那么`self.myString`其实是操作的实例变量_xxx，而不是`_myString`了。
