---
title: Objc - 对象之类对象和元类对象
tags: [iOS,Objc]
categories: [iOS,Objc]
date: 2018-06-09 08:12:45
---
# Objc 中的类（Class）
众所周知，在 Objc 中所有的对象都由类实例化而来，**殊不知类本身也是一种对象**。
在 Objc 中几乎所有的类都是 NSObject 的子类，NSObject类定义如下(忽略方法声明)：
``` Objc
@interface NSObject <NSObject> {
    Class  isa  OBJC_ISA_AVAILABILITY;
}
@end
```
这个`isa`是什么呢？在 `objc.h` 中我们发现它仅仅是一个 typedef 的结构体(struct)定义，如下:
```Objc
typedef struct objc_class *Class;
```
同样的`objc_class`又是什么呢？在 Objc2.0 中 `objc_class`定义如下：
``` Objc
struct objc_class {
    Class  isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class  super_class  OBJC2_UNAVAILABLE;
#endif
} OBJC2_UNAVAILABLE;
```
这里又出现了一个`isa`指针，这个 `isa`和上一个有什么区别和联系？

答：其实在 Objc 中任何类的定义都是一个对象。在编译的时候，编译器会给每一个类生成且只生成一个「描述其定义的对象」，也就是苹果公司所说的类对象（class object）；因为 Objc 和是一门动态语言，程序中所有的实例对象（instance object）是在运行时库生成的，而这个类对象（class object）就是运行时库用来创建实例对象（instance object）的依据。

再回到之前的问题，为什么实例对象（instance object）的`isa`指针指向的类对象（class object）里面还有一个`isa`指针？而这个类对象（class object）的 `isa` 指向的仍然是一个`objc_class`，它就是元类对象（metaclass object）；那么实例对象（instance object）、类对象（class object）、元类对象（metaclass object）之间的关系是怎样的呢？如下图：
![类关系图](http://p7xd6yrmx.bkt.clouddn.com/objc_class_relationship.png)

## 类对象（class object）
### 类对象的实质
类对象是由编译器创建的；**即在编译时所谓的类，就是指类对象**。
>官方文档中这样描述：The class object is the compiled version of the class.

任何直接或间接继承了 NSObject 的类，它的实例对象(instance object)中都有一个`isa`指针，指向它的类对象(class object)。**这个类对象(class object)中存储了关于这个实例对象(instance object)所属的类定义的一切：包括变量、方法、遵守的协议等**。因此类对象(class object)能访问所有关于这个类的信息，利用这些信息可以产生一个新的实例对象(instance object)，但是类对象不能访问任何实力对象的内容。
当调用一个类方法(class method)时，例如：`[NSObject alloc]`，事实上是发送了一个消息给它的类对象。

### 类对象和实例对象的区别
尽管类对象保留了一个类实例的原型，但它并不是实例本身；它没有自己的实例变量，也不能执行那些类的实例方法

### 类对象与类名
在代码中，类对象由类名表示。

举个例子：定义类 Person 继承 NSObject
``` Objc
int versionNumber = [Person version];
```
在上面的例子中，Person 类从 NSObject 继承 `version`方法来返回类的版本号；只有在消息表达式中作为接受者时，类名才代表类对象。

另外，类对象和其他对象一样也是`id`类型，例如：
``` objc
id clazz = [anyObject class];
id pClazz = [Person class];
```
总之，类对象是一个功能完整的对象，所以也能被动态识别(dynamically typed)接收消息，从其他类继承方法。特殊之处在于它们是由编译器创建的，缺少它们自己的数据结构(实例变量)，只是在运行时产生实例的代理。

## 元类对象（metaclass object）
### 元类对象的实质
实际上，**类对象是元类对象的一个实例**，元类对象描述了一个类对象，就像类对象描述了普通的实例对象一样。不同的是元类的方法列表是类方法的集合，由类对象的选择器来响应；当一个类发送消息时候，`objc_msgSend`会通过类对象的`isa`指针定位到元类，并检查元类的方法列表（包括父类）来决定调用哪个方法。元类代替了类对象描述了类方法，就像类对象代替实例对象描述了实例化方法。

很显然，**元类也是对象，也应该是其他类的实例**，实际上元类是根元类(root class's metaclass)的实例，而根元类是其自身的实例，即根元类的 `isa` 指针指向自身。

类的`super_class`指向其父类，而元类的 `super_class` 则指向父类的元类；元类的 `super_class` 链与类的`super_class`链平行，所以类方法的继承与实例方法的继承也是平行的。而根元类(root class's metaclass)的 `super_class`指向根类(root class)，这样整个指针链就链起来了。

综上所述: 类对象(class object)保存的是关于实例对象的信息(ivar，实例方法等)，而元类对象(metaclass object)存储的是关于类的信息（类的版本，名字，类方法等）
要注意的是，类对象和元类对象的定义都是`objc_class`结构。其不同仅仅是在用途上，比如其中的方法列表在类对象(class object)中保存的是实例方法(instance method)，而在元类对象(metaclass object)中保存的则是类方法(class method)

## 类对象和元类对象的相关方法
### object_getClass
`object_getClass`跟随实例的`isa`指针，返回此实例所属的类，对于实例对象(instance object)返回的是类(class)，而对于类(class)则返回的是元类(metacalss)

### class
`class`方法对于实例对象(instance object)会返回类(class)，但对于类(class)则不会返回元类(metaclass)，而只会返回类本身，即`[@“instance” class]`则返回的是`__NSCFConstantString`；而`[NSString class]`则返回的是 `NSString`

### class_isMetaClass
`class_isMetaClass`判断某类是否是元类

### objc_allocateClassPair
使用`objc_allocateClassPair`可在运行时创建新的类对与元类对，使用`class_addMethod`和`class_addIvar`可向类中增加方法和实例变量，最后使用`objc_registerClassPair`注册后，就可以使用此类了。

# 参考资料
- https://blog.csdn.net/wzzvictory/article/details/8592492
