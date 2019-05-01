---
title: Java - 注解(Annotation)
date: 2019-04-30 12:01:47
tags: [Java]
categories: Java
---


## 定义
注解，也叫元数据；一种代码级别的说明，是JDK1.5之后版本引入的一个特性，与接口、类、枚举在同一个层次。可以声明在包、类、字段、局部变量、方法参数等的前面，用来对这些元素进行注释说明。

**说明**：可用于取代xml、properties配置文件，给对象(Class,Field,Method)添加配置

## 作用分类
1. 编写文档：通过代码里标识的元数据(注解)生成文档（doc）
2. 代码分析：通过代码里标识的元数据(注解)对代码进行分析（利用反射）
3. 编译检查：通过代码里标识的元数据(注解)让编译器能够实现基本的编译检查（Override）

## JDK中预定义的一些注解
### @Override
检测被该注解标注的方法是否是继承父类（接口）的
 * jdk5表示当前被修饰的方法，必须覆写的父类中的方法
 * jdk6表示当前被修饰的方法，可以实现接口

### @Deprecated
该注解标注的内容，表示已过时
### @SuppressWarnings
`rawtypes`： 没有泛型信息
`unchecked`： 给没有泛型信息的集合添加数据不进行数据验证
`unused`： 没有使用
`deprecation`： 方法过期
`null` ：数据为null
`serial`：实现implements java.io.Serializable，但没有提供序列号ID
`all` ：所有
**注意** 压制警告，一般传递参数all   `@SuppressWarnings("all")`，扩展用于类、方法、字段等

## 自定义注解
### 格式
```java
public @interface 注解名称{
    属性列表;
}
```
### 本质  
注解本质上就是一个接口，该接口默认继承(extends)Annotation接口
source：
```java
public interface MyAnno extends java.lang.annotaion.Annotation{}
```

### 属性
类同接口中定义成员抽象方法，但是在注解中称之为属性。
注解属性格式：修饰符 类型 字段名称() [default 默认值]
eg:
```java
public @interface MyAnno{
    [public] [abstract] int show();
    String name();
    //枚举类型
    Genger gender();
    //其他注解类型
    MyAnno anno2();
    //数组类型
    String[] strs();
}
```
- 属性的返回值类型取值范围
1. 基本数据类型
2. String 
3. 枚举
4. 注解
5. 以上类型的数组形式

**注意**：注解中所定义的属性返回值类型不可以是`void`

- 注解属性的赋值使用
1. 使用格式：@注解名称(属性名称=值)
2. 若定义属性时，使用了`default`关键字给属性以默认初始化值，则在使用注解时，可以不进行属性的赋值。
```java
public @interface MyAnno{
    String name() default "techbird";
}
```
3. 特别的：若属性名称为`value`，且属性只有一个时；那么在对属性赋值时候，value可省略，直接定义值即可
```java
@MyAnno(value = 12)
public class Test{}
```
4. 多个属性赋值需要用逗号分隔
```java
@MyAnno(value = 12,gender = Gender.Man,anno2 = @MyAnno,strs = {"a","b"})
```
5. 如果属性类型为数组，赋值时需要用{}包裹，如果只有一个值时，{}可省略

## 元注解
用于描述注解的注解成为元注解。
### @Target
描述注解能够作用的位置(TYPE,FIELD,METHOD)
`@Target(ElementType.TYPE)`:表示注解只能作用于类或接口上
`@Target(ElementType.FIELD)`:表示注解只能作用于成员变量上
`@Target(ElementType.METHOD)`:表示注解只能作用于方法上
`@Target(ElementType.CONSTRUCTOR)`:构造方式

**注意**：如果没有指定Target，说明任意位置都可以使用
```java
@Target(value={ElementType.TYPE,ElementType.METHOD,ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnno{
}
```
### @Retention
描述注解被保留的阶段(SOURCE,CLASS,RUNTIME)
`@Retention(RetentionPolicy.SOURCE)`：修饰的注解存在于源文件，提供给编译器使用。
`@Retention(RetentionPolicy.CLASS)` ：修饰的注解存在于字节码文件，提供JVM使用
`@Retention(RetentionPolicy.RUNTIME)`：修饰的注解存在于内存，提供给程序(自己)
### @Documented
在使用javadoc生成api时，将被修饰的注解添加上
### @Inherited
被修饰的注解，所使用的类的所有子类，都继承当前注解

## 使用(在程序中解析注解)
1. 获取注解定义的位置的对象`(Class,Field,Method)`
2. 获取制定的注解：`位置对象.getAnnotation(Annotation.class)`
3. 调用注解中的抽象方法获取配置的属性值

eg:
- 定义注解
```java
@Target(value = {ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnno {
    String className();
    String methodName();
}
```

- 定义类并使用所定义的注解
```java
@MyAnno(className = "me.techbird.reflect.Test",methodName = "show")
public class Demo {
    public static void main(String[] args) throws Exception{
        Class<Demo> demoClass = Demo.class;
        //下面一行代码相当于内存中搞了一个MyAnnoImpl类复写了两个抽象方法
        MyAnno annotation = demoClass.getAnnotation(MyAnno.class);
        String className = annotation.className();
        String methodName = annotation.methodName();

        Class<?> aClass = Class.forName(className);
        Object obj = aClass.newInstance();
        Method method = aClass.getMethod(methodName);
        method.invoke(obj);
    }
}
```
## 反射注解

检查类/方法/类变量/构造方法是否有具体的注解,获取注解,根据注解来执行不同的操作
`AnnotatedElement`定义了如下方法,而`Class/Method/Field/Constructor`都实现了这个接口,所以通过这些对象都可以调用如下方法在类/方法/成员变量/构造器上获取或判断是否包含指定的注解

`<T extends Annotation>   T  getAnnotation(Class<T> annotationClass`)
如果存在该元素的指定类型的注解，则返回这些注解，否则返回 null。

`Annotation[] getAnnotations()`
返回此元素上存在的所有注解。

`Annotation[] getDeclaredAnnotations()`
返回直接存在于此元素上的所有注解。

`boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)`
如果指定类型的注解存在于此元素上，则返回 true，否则返回 false。 

## 注意
一个对象一个注解只能使用一次