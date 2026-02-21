---
title: Java 7、8、9+版本的一些新特性
date: 2019-04-10 10:01:47
tags: [Java]
categories: Java
---

## 集合
JDK1.7之前，创建集合对象必须把前后的泛型都写上；JDK1.7之后，后面的反省可省略，会自动推导。


## 内部类
从JDK1.8开始只要局部变量事实上不变，那么内部类访问外部变量时，`final`关键字可以省略。

## 哈希表
JDK1.8之前是哈希表=数组+链表结构。从JDK1.8开始，哈希表=数组+链表/红黑树（提高查询的速度），如果链表的长度超过了8位，就会把链表转换为红黑树结构。
![hashtable-structure](/assets/img/tech/java_hashtable_structure.png)

## Objects类
在JDK1.7添加了一个`Objects`工具类，它提供了一些方法来操作对象，它由一些静态的使用方法组成。
这些方法是`null-save`(空指针安全)、`null-tolerant`（容忍空指针），用于计算对象的`hashcode`，返回对象的字符串形式、比较两个对象。
在比较两个对象的时候`Object`的`equals`方法容易抛出空指针异常，而`Objects`类中的`equals`就优化了这个问题，方法如下:
``` java
public static boolean equals(Object a, Object b){
     return (a == b) || (a != null && a.euals(b));
}
```

## IO流异常
### JDK1.7之前的写法：
```java
try{
}catch(异常类变量 变量名){
}finally{
}
```
### JDK1.7之后的写法：
```java
try(定义对象1;定义对象2;...){
     //定义的对象作用范围仅在此大括号内
}catch(异常类变量 变量名){
}
```
### JDK1.9之后：
`try`前面也可以定义流对象，在`try`后面的`()`中可以直接引入流对象的名称(变量名)；在`try`执行完毕后，流对象也可以释放掉，不用写`finally`。但是比较麻烦的是既需要 `throws` 也需要`try catch`。
```java
A a = new A();
B b = new B();
tru(a,b){
}catch(异常类变量 变量名){
}
```

## Lambda表达式
JDK1.8开始，Oracle加入了**Lambda表达式**重量级新特性；函数式接口在Java中是指，有且仅有一个**抽象方法**的接口，称为**函数式接口**；即适用于函数式编程场景的接口，在Java中函数是编程体现就是`Lambda`，所以函数式接口就是可以适用于`Lambda`使用的接口，只有确保接口中有且只有一个抽象方法，Java中的`Lambda`才能顺利的进行推导。
### Lambda接口
```java
修饰符 interface 接口名称{
     [public] [abstract] 返回值类型 方法名称(可选参数信息);
     //其他非抽象方法(默认，静态，私有)
}
```
### @FunctionalInterface
这个注解可以确保我们定义的接口是一个函数式接口，也就是接口中有且必须只有一个抽象方法。 
### 调用Lambda的标准格式：
- 由三部分组成：
     a. 一些参数
     b. 一个箭头
     c. 一段代码
**格式**：（参数列表）-> {一些重写方法的代码};

### 特性：可推导，可省略
- 可省略的内容：
    1. (参数列表)：括号中参数列表的数据类型，可以省略不写
    2. (参数列表)：括号中参数如果只有一个，那么类型和()都可省略
    3. {一些代码}：如果{}中代码只有一行，无论是否有返回值，都可以省略({},return,分号)
    注意：要省略{},return,分号必须一起省略

### 常用的函数式接口
JDK提供了大量常用的函数式接口以丰富Lambda的典型使用场景，他们主要在`java.util.function` 包中被提供。
- eg:
`Supelier` 中方法有：get()
`Consumer` 中方法有：andThen(Consumer)、accept(T)
`Predicte` 中方法有：and(Predicte)、or(Predicte)、negate()、test(T)、isEqual(Object)
`Function` 中方法有：andThen(Funtion)、apply(T)、conpose(Function)、identity()

### 使用前提
1. 使用`Lambda`必须具有接口，且要求接口中有且仅有一个抽象方法。
   无论是JDK内置的`Runnable`、`Comparator`接口还是自定义的接口，只有当接口中的抽象方法存在且唯一时，才可以使用`Lambda`
2. 使用`Lambda`必须具有上下文**上下文推断**。
   也就是方法的参数或局部变量类型必须为`Lambda`对应的接口类型，才能使用`Lambda`作为该接口的实例。

## 方法引用(Method Reference)
### 对象名引用成员方法
对象引用::实例方法名 `obj::method`
```java
public class Demo {
    public void printString(Printable lambda){
        lambda.print("Hello");
    }
    public static void main(String[] args) {
        Demo obj = new Demo();
        obj.printString(System.out::print);
    }
}

```
### 类名引用静态方法
类名::静态方法名 `Class::method`
```java
    public static int method(int num,Calcable lambda){
       return lambda.calcAbs(num);
    }
    public static void main(String[] args) {
        int number1 = method(-10, (num) -> {
            return Math.abs(num);
        });
        System.out.println(number1);

        //省略写法
        int number2 = method(-10, num -> Math.abs(num));
        System.out.println(number2);

         //静态方法引用
        int number3 =method(-10,Math::abs);
        System.out.println(number3);
    }
```
### this引用本类成员方法
`this::method`
### super引用父类成员方法
`super::method`
### 类构造器引用
`类名::new`
### 数组构造器引用
`类型[]::new`

## Stream流
说到`Stream`很容易联想到`I/O Stream`，实际上两者是完全不同的概念。在JDK1.8中，得益于`Lambda`所带来的**函数式编程**，引入了一种**全新的Stream概念**，用于解决已有集合类库/数组既有的弊端。
eg:
```java
list.stream().filter(name->name.starWith("zhang"))
             .filter(name->name.length == 3)
             .forEach(name->System.out.println(name));
```
### 延迟方法
返回值类型仍然是`Stream`接口自身类型的方法，因此支持链式调用。（除了终结方法外，其他均为延迟方法）
例如：`filter`、`map`、`skip`、`limit`等
### 终结方法
返回值类型不再是`Stream`自身接口类型的方法，因此不再支持`StringBuilder`那样的链式调用。
例如：`forEach`和`count`
### 静态方法
```java
Stream.of(1,2,3)//把Int数组转为Stream流
Stream.concat(stream1,stream2);//合并两个流为一个
```

## Interface(接口)
### 在Java 9+的版本中，接口包含的内容有如下：

1. 成员变量其实是常量，格式：
[public] [static] [final] 数据类型 常量名称 = 数据值;
注意：常量必须进行赋值，而且一旦赋值不能改变。
     常量名称完全大写，用下划线进行分割。

2. 接口中最重要的就是抽象方法，格式：
[public] [abstract] 返回值类型 方法名称（参数列表);
注意：实现类必须覆盖重写接口所有的抽象方法，除非实现类是抽象类。

3. 从Java 8开始，接口里允许定义默认方法，格式：
[public] default 返回值类型 方法名称（参数列表）{方法体}
注意：默认方法也可以被覆盖重写

4. 从Java 8开始，接口里允许定义静态方法，格式：
public static 返回值类型 方法名称（参数列表）{方法体}
注意：应该通过接口名称进行调用，不能通过实现类对象调用静态方法（因为多实现中可能存在相同的方法）

5. 从Java 9开始，接口里允许定义私有方法，格式：
普通的私有方法：private 返回值类型 方法名称（参数列表){方法体}
静态的私有方法：private static 返回值类型 方法名称(参数列表){方法体}
注意：private的私有方法只有接口自己才能调用，不能被实现类或别人使用。

### 注意：
1. 接口中没有静态代码块或者构造方法
2. 接口中没有静态代码块或者构造方法
