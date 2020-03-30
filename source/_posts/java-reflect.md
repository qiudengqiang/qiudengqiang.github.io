---
title: Java - 反射(Reflect)
date: 2019-04-30 10:01:47
tags: [Java]
categories: Java
---

## 定义
反射是框架设计的灵魂。Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

## Class
一个Java类中用一个`Class`类的对象来表示，一个类中的组成部分：成员变量(`Field`)，方法(`Method`)，构造方法(`Constructor`)，包(`Package`)等等信息也用一个个的Java类来表示

- 获取`Class`对象的三种方式：
```java
//Class.forName(“全类名”)，多用于配置文件中
Class cls1 = Class.forName("me.techbird.base.Person");
//类名.class，多用于参数的传递
Class cls2 = Person.class;
//对象.getClass()，多用于拥有对象的情况下获取字节码
Person p = new Person();
Class cls3p.getClass();
```

**结论**：同一个字节码文件（*.class）在一次程序运行过程中，只会被加载一次，无论使用哪一种方式获取的对象都是同一个。

- 九个预定义的`Class`：
    1. 包括八种基本类型（byte、short、int、long、float、double、char、boolean）的字节码对象和一种返回值为void类型的`void.class`。
    2. `Integer.TYPE`是Integer类的一个常量，它代表此包装类型包装的基本类型的字节码，所以和`int.class`是相等的。基本数据类型的字节码都可以用与之对应的包装类中的`TYPE`常量表示

- 只要是在源程序中出现的类型都有各自的Class实例对象，如`int[].class`。数组类型的Class实例对象，可以用`Class.isArray()`方法判断是否为数组类型的。

### Class类中的常用方法
```java
static Class<?>	    forName​(Module module, String name)
Annotation[]	    getAnnotations()
ClassLoader	        getClassLoader()
String	            getName()
T	                newInstance()
Class<? super T>	getSuperclass()
String	            getSimpleName()
boolean             isPrimitive()//是否是基本类型

```
注意：字节码文件作比较时候建议使用`==`而不是`equels`，因为字节码在内存中只有一份。
### Field
`Field	getField​(String name)`:获取所有public修饰的成员变量
`Field[]	getFields()`:获取指定名称的public修饰的成员变量
`Field	getDeclaredField​(String name)`:暴力获取指定名称的成员变量，不考虑修饰符
`Field[]	getDeclaredFields()`:暴力获取所有成员变量，不考虑修饰符

### Constructor
`Constructor<T>	getConstructor​(Class<?>... parameterTypes)	`:获取public的带参构造方法
```java
//带参数
Constructor con1 = cls.getConstructor(String.class,int.class);
//不填参数是即为获取空构造方法
Constructor con2s = cls.getConstructor();
//拿到构造方法之后便可以调用constructor.newInstance(...)方法来构造对象
```
`Constructor<?>[]	getConstructors()`:获取所有构造方法
`Constructor<T>	getDeclaredConstructor​(Class<?>... parameterTypes)	`
`Constructor<?>[]	getDeclaredConstructors()`
### Method
`Method	getMethod​(String name, Class<?>... parameterTypes)	`
```java
//带参数
Method method1 = cls.getMethod("show", int.class);
//不带参数
Method method2 = cls.getMethod("show");
```
`Method[]	getMethods()`
`Method	getDeclaredMethod​(String name, Class<?>... parameterTypes)	`
`Method[]	getDeclaredMethods()`
`Object	invoke​(Object obj, Object... args)`:调用方法

**说明**: 如果传递给Method对象的invoke()方法的第一个参数为null，说明Method对象对应的是一个静态方法

### Package
`Package	getPackage()`	:获取当前类的包对象
`String	getPackageName()`   :获取当前类的包字符串

### 数组的反射
``` java
int [] a1 = new int[]{1,2,3};
int [] a2 = new int[4];
int[][] a3 = new int[2][3];
String [] a4 = new String[]{"a","b","c"};
```
具有相同维数和元素类型的数组属于同一个类型，即具有相同得Class实例对象.
``` java
System.out.println(a1.getClass() == a2.getClass());
System.out.println(a1.getClass() == a4.getClass());
System.out.println(a1.getClass() == a3.getClass());
```
代表数组的Class实例对象的`getSuperclass()`方法返回的是Object类对应的Class.
``` java
System.out.println(a1.getClass().getName());
System.out.println(a1.getClass().getSuperclass().getName());
System.out.println(a4.getClass().getSuperclass().getName());
```
基本类型的一维数组可以被当做`Object`类型使用，不能当做`Object[]`类型使用；非基本类型的一维数组，既可以当做`Object`类型使用，又可以当做`Object[]`类型使用.
``` java
Object aObj1 = a1;
Object aObj2 = a4;
//Object[] aObj3 = a1; //不成立
Object[] aObj4 = a3;
Object[] aObj5 = a4;
```
Arrays.asList()方法处理`int[]`和`String[]`时候的差异.
``` java
System.out.println(a1);
System.out.println(a4);
System.out.println(Arrays.asList(a1));
System.out.println(Arrays.asList(a4));	
```
利用`java.util.reflect.Array`类完成对数组的反射操作.
``` java
//可以打印一个obj和一个obj数组的方法
private static void printObject(Object obj) {
		Class clazz = obj.getClass();
		if(clazz.isArray()){
			int len = Array.getLength(obj);
			for(int i=0;i<len;i++){
				System.out.println(Array.get(obj, i));
			}
		}else{
			System.out.println(obj);
		}
}
```
如何获取数组中的元素的类型
``` java
Object[] objs = new Object[]("a",1);
a[0].getClass().getName();

```
### 暴力反射
如果想要获取`非public`类型的`Field`、`Constructor`、`Method`等，需要在执行`Declared`方法前设置以下代码：
```java
obj.setAccessable(ture);
```

## 用反射技术执行某个main方法
```java
//反射方式    
Class clazz=Class.forName(“className”);    
Method methodMain=clazz.getMethod("main",String[].class);    
 //方式一：强制转换为超类Object，不用拆包    
methodMain.invoke(null, (Object)new String[]{"123","456","789"});    
//方式二：将数组打包，编译器拆包后就是一个String[]类型的整体     
methodMain.invoke(null, new Object[]{new String[]{"123","456","789"}});   
```

