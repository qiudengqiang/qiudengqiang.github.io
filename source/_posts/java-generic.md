---
title: Java - 泛型(Generic)
date: 2019-05-01 10:01:47
tags: [Java]
categories: Java
---

## 定义
泛型是一种未知的数据类型，当我们不知道使用什么数据类型的时候，可以使用泛型；也可以把泛型看做是一个变量，用来接收数据类型。泛型是提供给java编译器使用的，只在编译阶段可见，在源码文件编译为字节码文件的过程中，泛型会被移除掉，这个过程叫做**泛型的擦除**。

## 名词解释
E:Element元素
T:Type类型

ArrayList<E>中的E称为Element类型参数变量
ArrayList<Integer>中的Integer称为实际类型参数

整个称为ArrayList<E>泛型类型
整个ArrayList<Integer>称为参数化的类型ParameterizedType

## 集合泛型
在jdk1.5以前对象一旦存入集合就丢失了类型，在获取时需要进行强转，麻烦并且容易出错。
在jdk1.5以后提供了泛型技术，可以在定义泛型时明确指定存储的类型；如果不指定默认为Object，可以存储任意类型数据。
```java
List<Integer> list = new ArrayList<Integer>();
//jdk1.7之后，后面的泛型可以说省略不写，会自动推导。
List<String> list = new ArrayList<>();
```
**注意**：泛型不支持基本数据类型，但支持其包装类。
## 自定义泛型
### 定义在方法上的泛型
作用范围是当前方法，在方法被调用时候，编译器会自动判断出泛型的具体类型。

### 定义在类上的泛型
作用范围是整个类，在创建类的对象时，就需要明确的指定泛型的具体类型；如果不指定默认为此泛型的上边界，如果没有上边界则默认上边界为Object。

- 定义格式：
```
修饰符 class  类名<代表泛型的变量> { }
```
- 例如，API中的ArrayList集合
```java
class ArrayList<E>{
    public boolean add(E e){}
    public E get(int index){}
    ...
}
```
- 使用格式：在使用时候在指定具体的泛型类型
```java
//具体为String
List<String> list = new ArrayList<>();
list.add("string");
//具体为Integer
List<Integer> list = new ArrayList<>();
list.add("integer");
```

**注意**： 静态方法不能使用类定义的泛形，而应单独定义泛形

### 定义在接口上的泛型

- 定义格式：
```java
修饰符 interface 接口名<代表泛型的变量> { }
```
1. 第一种方式
eg：
```java
public interface MyGenericInterface<E>{
    public abstract void add(E e);
    public abstract E get(); 
}
```
- 使用格式：
```java
public class MyImpl implements MyGenericInterface<String>{
    @Override
    public void add(String s){}

    @Override
    public String get(){}
}
```

2. 第二种方式，接口是什么类型，继承实现类还使用什么抽象类型。仿ArrayList
eg：
```java
public interface MyGenericInterface<I>{
    public abstract void method(I i);
}
```
- 使用格式：
```java
public class MyImpl<I> implements MyGenericInterface<I>{
    @Override
    public void method(I i){}
}
```

## 泛型通配符
当使用泛型类或者接口时，传递的数据中，泛型的类型不确定，可以用通配符`<?>`表示。但是一旦使用泛型的通配符后，只能使用Object类中的共性方法，集合中自身元素方法无法使用。

eg：
```java
List<String> list1 = new ArrayList<String>();
List<Integer> list2 = new ArrayList<Integer>();

List<Object> list0= null;
list0=list1;//报错
list0=list2;//报错
```
list0 没有办法引用 list1 和 list2，这时该如何设计 list0 的泛型使他能够引用 list1 和 list2 呢?
此时引入泛型通配符可以写成：
```java
List<?> list0 = null;
```
表明当前集合泛型的具体类型是不确定的，此时这个引用就可以接受任意具体类型泛型。

## 泛型的边界
使用了泛型通配符以后，List<?> list0 就可以等于任意的类型了，但是有时我们还可能需要对这个引用可以接受的泛型类型进行一些限定：
### 泛型的上限
- 格式：`类型名称 <? extends 类> 对象名称`
也就是说，可以通过这种方式定义此泛型所接受的具体类型必须是限定的`该类型或其子类类型`。
在使用这个泛型时，可以使用这个泛型上边界所具有的方法；
如果不明确限定泛型的上边界，则上边界默认为Object。

### 泛型的下限
- 格式：`类型名称 <? super 类> 对象名称`
也就是说，可以通过这种方式定义此泛型接受的具体类型必须是限定的`该类型或其父类类型`。

eg:Integer extends Number extends Object
```java
Collection<Integer> list1 = new ArrayList<Integer>
Collection<String> list2 = new ArrayList<String>
Collection<Number> list3 = new ArrayList<Number>
Collection<Object> list4 = new ArrayList<Object>

getElement1(list1);
getElement1(list2);//报错
getElement1(list3);
getElement1(list3);//报错

getElement2(list1);//报错
getElement2(list2);//报错
getElement2(list3);
getElement2(list4);
//泛型的上限：此时的泛型? 必须是Number类型或者Number类型的子类
public static void getElement1(Collection<? extends Number> coll){}
//泛型的下限：此时的泛型? 必须是Number类型或者Number类型的父类
public static void getElement2(Collection<? super Number> coll){}
```
             