---
title: iOS - KVC 和 KVO 的使用和原理
date: 2018-05-23 12:55:57
tags: [iOS,Objc,KVC,KVO]
categories: [iOS,Objc]
---
# KVC
KVC(键值编码)，即 Key-Value Coding，一个非正式的 Protocol，使用字符串(键)访问一个对象实例变量的机制。而不是通过调用 Setter、Getter 方法等显式的存取方式去访问。
## 简介
KVC（Key-value coding），键值编码；是指在iOS开发中，可以允许开发者通过属性名`Key`直接访问对象的属性并给属性`编码`（赋值value），而不是调用对应的getter/setter方法。很多高级的iOS开发技巧都是基于KVC实现，例如修改系统控件内部属性；json->model的映射框架等。
## KVC最重要的四个方法
```Objc
- (nullable id)valueForKey:(NSString * )key;                          //直接通过Key来取值
- (void)setValue:(nullable id)value forKey:(NSString * )key;          //通过Key来设值
- (nullable id)valueForKeyPath:(NSString * )keyPath;                  //通过KeyPath来取值
- (void)setValue:(nullable id)value forKeyPath:(NSString * )keyPath;  //通过KeyPath来设值
```
## valueForKey 和 valueForKeyPath区别
### 先来看一段代码
```Objc
- (void)viewDidLoad {
    [super viewDidLoad];
    NSDictionary * dict = @{@"key1":@"value1",
                           @"second":@{
                                   @"k1":@"v1",
                                   @"k2":@"v2",
                                   @"third":@{
                                           @"t1":@"h1",
                                           @"t2":@"h2"
                                           }
                                   }
                           };

    NSDictionary * second = [dict valueForKey:@"second"];
    NSDictionary * third1 = [second valueForKey:@"third"];
    NSLog(@"%@",third1);
    NSDictionary * third2 = [dict valueForKeyPath:@"second.third"];
    NSLog(@"%@",third2);
}
```
### 输出结果：
```console
2018-05-23 13:52:33.430617+0800 KVC-Demo[1584:403407] {
    t1 = h1;
    t2 = h2;
}
2018-05-23 13:52:33.430800+0800 KVC-Demo[1584:403407] {
    t1 = h1;
    t2 = h2;
}
```
### 小结
通过代码我们可以看出，我们想要从dict这个字典中获取到`third`这个key所对应的值得话,使用`valueForKey`需要通过一层一层的对象才能取到想要的字典，而使用valueForKeyPath则只需要输入third在字典中的`路径（path）`一次就可以取到third字典。
### 参考博客
https://www.jianshu.com/p/c0e099f72a3b
https://www.jianshu.com/p/a6a0abac1c4a

## KVC的使用
### 代替getter/setter
```Objc
@interface Model : NSObject
@property (copy, nonatomic) NSString * text;
@property (copy, nonatomic) SubModel * subModel;
@end

@interface SubModel : Model
@property (copy, nonatomic) NSString * subText;
@end
```
- 不使用kvc
```Objc
 //赋值
 Model *model = [[Model alloc]init];
 model.text = @"text";
 SubModel *subModel = [[SubModel alloc]init];
 subModel.subText = @"subText";
 model.subModel = subModel;
 //取值
 NSString *text =  model.text;
 NSString *subText = model.subModel.subText;
```
- 使用kvc
```Objc
    //赋值
    Model *model = [[Model alloc]init];
    [model setValue:@"text" forKey:@"text"];
    [model setValue:@"subText" forKeyPath:@"subModel.subText"];
    //取值
    NSString *text =  [model valueForKey:@"text"];
    NSString *subText = [model valueForKeyPath:@"subModel.subText"];
```
### 字典转模型（仿YYModel）
- 创建NSObject的扩展NSObject+Model
```Objc
@interface NSObject (Model)
+ (instancetype) tb_modelWithDictionary:(NSDictionary * )dictionary;
@end
```
- 实现SObject+Model
利用Runtime取到对应类的属性列表，在使用kvc对所有属性进行赋值
```Objc
@implementation NSObject (Model)
+ (NSArray * )getPropertyList:(Class)cls{
    NSArray * array = objc_getAssociatedObject(self, `_cmd`);
    if (array != nil){
        return array;
    }
    NSMutableArray * arrM = [NSMutableArray array];
    //输出个数
    unsigned int outCount;
    //获取属性列表（ objc_property_t * ）
    objc_property_t * properties = class_copyPropertyList(cls, &outCount);
    for (NSInteger i=0; i<outCount; ++i) {
        objc_property_t property = properties[i];
        //属性名字
        NSString * name = [NSString stringWithUTF8String:property_getName(property)];
        [arrM addObject:name];
    }
    objc_setAssociatedObject(self, @selector(getPropertyList:), [arrM copy], OBJC_ASSOCIATION_RETAIN);
    free(properties);
    return [arrM copy];
}

+ (instancetype)tb_modelWithDictionary:(NSDictionary * )dictionary{
    NSObject * object = [[self alloc]init];

    NSArray * array = [self getPropertyList:[self class]];
    [dictionary enumerateKeysAndObjectsUsingBlock:^(NSString * key, id value, BOOL * stop) {
        if ([array containsObject:key]){
            [object setValue:value forKey:key];
        }
    }];

    return  object;
}
@end
```
- 使用tb_modelWithDictionary模仿YYModel的字典转模型方式
```Objc
- (void)viewDidLoad {
    [super viewDidLoad];
    NSDictionary * dict = @{
                           @"text":@"text"
                           };
    Model * model = [Model tb_modelWithDictionary:dict];
    NSLog(@"%@",model);
}
```

### 修改系统控件内部属性（runtime+kvc）
- 需求：修改UIPageControl小圆点的背景图片
- 查看UIPageControl.h如下
```Objc
NS_CLASS_AVAILABLE_IOS(2_0) @interface UIPageControl : UIControl

@property(nonatomic) NSInteger numberOfPages;          // default is 0
@property(nonatomic) NSInteger currentPage;            // default is 0. value pinned to 0..numberOfPages-1

@property(nonatomic) BOOL hidesForSinglePage;          // hide the the indicator if there is only one page. default is NO

@property(nonatomic) BOOL defersCurrentPageDisplay;    // if set, clicking to a new page won't update the currently displayed page until -updateCurrentPageDisplay is called. default is NO
- (void)updateCurrentPageDisplay;                      // update page display to match the currentPage. ignored if defersCurrentPageDisplay is NO. setting the page value directly will update immediately

- (CGSize)sizeForNumberOfPages:(NSInteger)pageCount;   // returns minimum size required to display dots for given page count. can be used to size control if page count could change

@property(nullable, nonatomic,strong) UIColor * pageIndicatorTintColor NS_AVAILABLE_IOS(6_0) UI_APPEARANCE_SELECTOR;
@property(nullable, nonatomic,strong) UIColor * currentPageIndicatorTintColor NS_AVAILABLE_IOS(6_0) UI_APPEARANCE_SELECTOR;
@end
```
没有发现UIPageControl暴露的操作中有设置小圆点背景图片的方法和属性，那么就可以利用runtime遍历UIPageControl类的成员变量（`ivar`）和属性（`property`）

- 利用runtime遍历UIPageControl成员变量

导入头文件：
```Objc
#import <objc/runtime.h>
```
遍历成员变量：
```Objc
- (void)viewDidLoad {
    [super viewDidLoad];
    UIPageControl * pc = [[UIPageControl alloc]init];
    NSArray * array = [self getIvarList:[pc class]];
    NSLog(@"%@",array);

}
- (NSArray * )getIvarList:(Class)cls{
    NSMutableArray * arrM = [NSMutableArray array];
    unsigned int outCount;
    Ivar * ivars = class_copyIvarList(cls, &outCount);
    for (NSInteger i=0; i<outCount; ++i) {
        Ivar ivar = ivars[i];
        NSString * name = [NSString stringWithUTF8String:ivar_getName(ivar)];
        NSString * type = [NSString stringWithUTF8String:ivar_getTypeEncoding(ivar)];
        NSString * str = [name stringByAppendingFormat:@" -- %@",type];
        [arrM addObject:str];
    }
    free(ivars);
    return [arrM copy];
}
```
输出结果：
```console
2018-05-23 16:45:18.380346+0800 KVC-Demo[4400:856521] (
    "_lastUserInterfaceIdiom -- q",
    "_indicators -- @\"NSMutableArray\"",
    "_currentPage -- q",
    "_displayedPage -- q",
    "_pageControlFlags -- {?=\"hideForSinglePage\"b1\"defersCurrentPageDisplay\"b1}",
    "_currentPageImage -- @\"UIImage\"",
    "_pageImage -- @\"UIImage\"",
    "_currentPageImages -- @\"NSMutableArray\"",
    "_pageImages -- @\"NSMutableArray\"",
    "_backgroundVisualEffectView -- @\"UIVisualEffectView\"",
    "_currentPageIndicatorTintColor -- @\"UIColor\"",
    "_pageIndicatorTintColor -- @\"UIColor\"",
    "_legibilitySettings -- @\"_UILegibilitySettings\"",
    "_numberOfPages -- q"
)
```
- 利用kvc设置`_currentPageImage`和`_pageImage`

```Objc
    UIPageControl *pc = [[UIPageControl alloc]init];
    [pc setValue:[UIImage imageNamed:@"pageImage"] forKeyPath:@"_pageImage"];
    [pc setValue:[UIImage imageNamed:@"currentPageImage"] forKeyPath:@"_currentPageImage"];
```

### XIB/Storyboard
在xib/Storyboard中，也可以使用KVC，例如下面是在xib中使用KVC把图片边框设置成圆角。
![](http://p7xd6yrmx.bkt.clouddn.com/kvc_layer_cornerradius.png)

# KVO
KVO(键值监听)，即 Key-Value Observing，它提供一种机制,当指定的对象的属性被修改后，对象就会接受到通知，前提是执行了 setter 方法、或者使用了 KVC 赋值。
## 简介
KVO 是 Objective-C 对观察者设计模式的一种实现；[另外一种是：通知机制（notification）]。
## 使用（Swift）
需求：UIScrollView内包含一部分原生控件和UIWebView的组合;这种情况下UIWebView的高度无法得知，因为UIWebView写完中包含UIScrollerView，所以需要利用KVO技术监听UIWebView中UIScrollerView的contentSize的变化以达到需求的目的。

### addOberver
- 一般在viewDidLoad中添加监听
```Swift
if let scrollView = mWebView.subviews.first as? UIScrollView {
            scrollView.alwaysBounceVertical = false
            scrollView.alwaysBounceHorizontal = false
            scrollView.bounces = false
            scrollView.addObserver(self, forKeyPath: "contentSize", options: .new, context: nil)
        }
```

### observerValueForkeyPath
- 当contentSize发生变化时,会回调到`observerValueForkeyPath`这个方法
```Swift
override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
        if keyPath == "contentSize" && change != nil {
            let value = (change![NSKeyValueChangeKey.newKey] as! NSValue).cgSizeValue.height
            desViewHeightCons.constant = value
        }
    }
```

### removeObserver
- 当界面销毁时，移除监听
```Swift
deinit {
        if let scrollView = mWebView?.subviews.first as? UIScrollView {
            scrollView.removeObserver(self, forKeyPath: "contentSize")
        }
    }
```

## KVO的原理
键值编码(KVC)和键值观察（KVO）是根据`isa-swizzling`技术来实现的，主要依据runtime的强大动态能力。
当某个类第一次被观察时，系统会在运行时期动态的创建一个该类的派生类，在这个派生类中重写任何被观察属性的`setter`方法。派生类在被重写的setter方法实现真正的通知机制，这么设计是基于设置属性会调用setter方法，而通过重写就获得了KVO需要的通知机制，当然前提是要遵循KVO的属性设置方式来变更属性值，如果直接修改属性对应的成员变量是无法实现KVO的。
同时派生类还重写了class方法`欺骗`外部调用者它就是起初的那个类，然后系统将`isa`指针指向这个新诞生的派生类，因此这个对象就成为该派生类的对象了，因为在该对象上对setter的调用就会调用的重写的setter，从而激活键值通知机制。此外派生类还重写了`delloc`方法来释放资源。

在Runtime篇章中介绍过，isa指针其实指向的是类的元类，如果添加监听之前的类名为`Person`,那么添加监听之后被runtime更改以后的类名会变成：`NSKVONotifying_Person`

新的派生类`NSKVONotifying_Person`会重写以下方法：
增加了监听的属性对应的`setter`,`class`,`delloc`,`_isKVOA`

### class
重写class方法是为了方便我们调用它的时候，返回跟重写继承类之前同样的内容。
```Objc
Person *person = [[Person alloc]init];
NSLog(@"before isa:%@  class:%@",object_getClass(person), [person class]);
[person addObserver:self forKeyPath:@"age" options:NSKeyValueObservingOptionNew context:nil];
NSLog(@"end isa:%@  class:%@",object_getClass(person), [person class]);
_person = person;
```
输出结果：
```console
2018-05-23 19:55:23.430908+0800 KVO-Demo[6395:1312349] before isa:Person  class:Person
2018-05-23 19:55:23.431456+0800 KVO-Demo[6395:1312349] end isa:NSKVONotifying_Person  class:Person
```
这也是`isa`指针和`class`方法的一个区别，使用的时候要`特别注意`。

### setter
新的派生类会重写对应的setter方法，其实是为了在setter中增加另外两个方法的调用
```Objc
- (void)willChangeValueForKey:(NSString * )key  
- (void)didChangeValueForKey:(NSString * )key  
```
其中 `didChangeValueForKey`负责触发：`observeValueForKeyPath:keyPath :object :change :context`方法，这就是`kvo`的原理。
如果没有执行`setter`之类的调用，那么使用`setValue:forKey`方法也会直接调用`observeValueForKeyPath:keyPath :object :change :context`方法
再如果既没有调用`setter`也没有调用`setValue:forKey`，那么
```Objc
- (void)willChangeValueForKey:(NSString * )key  
- (void)didChangeValueForKey:(NSString * )key  
```
我们只需要显示调用上述两个方法，就会触发`observeValueForKeyPath:keyPath :object :change :context`方法，同样可以使用KVO。

### `_isKVOA`
这个私有方法是用来表示该类是一个KVO机制声明的类

### 小结（触发KVO的三种方法）
1. 使用KVC （运行时会在`setValue:forKey`中来调用`will/didChangeValueForKey:`）
2. 使用setter方法（运行时会在setter方法中调用`will/didChangeValueForKey:`）
3. 显示调用`will/didChangeValueForKey:`方法

## 如何更优雅的使用KVO
只需要使用 Facebook 开源的 [KVOController](https://github.com/facebook/KVOController) 框架就可以优雅地解决这些问题了。
```Objc
[self.KVOController observe:person
                    keyPath:@"age"
                    options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld
                      block:^(id  observer, id  object, NSDictionary<NSString  * , id> *  change) {
                          NSLog(@"%@", change);
                      }];
```
我们可以在任意对象上获得 `KVOController` 对象，然后调用它的实例方法 `-observer:keyPath:options:block:` 就可以检测某个对象对应的属性了，该方法传入的参数非常容易理解，在 block 中也可以获得所有与 KVO 有关的参数。

使用 KVOController 进行键值观测可以说完美地解决了在使用原生 KVO 时遇到的各种问题:

不需要手动移除观察者；
实现 KVO 与事件发生处的代码上下文相同，不需要跨方法传参数；
使用 block 来替代方法能够减少使用的复杂度，提升使用 KVO 的体验；
每一个 keyPath 会对应一个属性，不需要在 block 中使用 if 判断 keyPath；

# 参考文档和博客
http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/KeyValueCoding/Articles/KeyValueCoding.html#//apple_ref/doc/uid/10000107-SW1
https://blog.csdn.net/wzzvictory/article/details/9674431
https://blog.csdn.net/kesalin/article/details/8194240
https://draveness.me/kvocontroller
