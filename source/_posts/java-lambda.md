---
title: lambda表达式详解
date: 2021-02-24 19:01:47
tags: [Java,lambda]
categories: Java
---

lambda 表达式是一个可传递的代码块，可以在以后执行一次或多次。

## 语法

表达式形式：参数，箭头(->)，以及一个表达式。例如：

(String first, String second) -> first.length() - second.length()

如果代码要完成的计算无法放在一个表达式中，就可以像写方法一样，把这些代码放在{}中，并包含显示的return语句。例如：

```java
（String first, String second）-> {

  if(first.length() < second.length()) return -1;

   else if(first.length() > second.length()) return 1;

   else return 0;

}
```

即使lambda表达式没有参数，仍然要提供空括号，就像无参数方法一样：

```java
() -> {for (int i = 100; i >= 0; i—) System.out.println(i);}
```



如果可以推导出一个lambda表达式的参数类型，则可以忽略其类型。例如:

```java
Comparator<String> comp = (first, second) -> first.length() - second.length();
```


在这里，编译器可以推导出first和second必然是字符串，因为这个lambda表达式将赋给一个字符串比较器。

如果方法只有一个参数，而且这个参数的类型可以推导得出，那么甚至还可以省略小括号：

```java
ActionListener listener = event -> System.out.println(“test”);
```


无须指定lambda表达式的返回类型。lambda表达式的返回类型总是会由上下文推导得出。例如：

```java
(String first, String second) -> first.length() - second.length()
```


可以在需要int类型结果的上下文中使用。

注释：如果lambda表达式只在某些分支返回一个值，而另外一些分支不返回值，这是不合法的。例如：

```java
(int x) -> { if (x >= 0) return 1; }//不合法
```



## 函数式接口

对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个lambda表达式，这种接口成为函数式接口（functional interface）。例如：

```java
Arrays.sort(words,(first, second) -> first.length() - second.length());
```


在底层，Arrays.sort 方法会接受实现了Comparator<String>的某个类的对象。在这个对象上调用compare方法会执行这个lambda表达式的体。

注释：最好把lambda表达式看作是一个函数，而不是一个对象，另外要接受lamda表达式可以传递到函数式接口。实际上在Java中，对lambda表达式所能做的也只是转换为函数式接口，Java设计者没有为Java语言增加函数类型（在其他程序设计语言中，可以声明函数类型的变量）。不能把lambda表达式赋值给类型为Object的变量，因为Object不是函数式接口。

`java.util.function` 包中定义了很多非常通用的函数式接口。例如：

Predicate：

```java
public interface Predicate<T> {

  boolean test(T t);

  //additional default and static methods

}
```



ArrayList类中有一个removeIf方法，它的参数就是一个Predicate。这个接口专门用来传递lambda表达式。例如，下面的语句将从一个数组列表中删除所欲的null值：

```java
list.removeIf(e -> e == null);

Supplier：

public interface Supplier<T> {

  T get();

}
```

供应者（supplier）没有参数，调用时会生成一个T类型的值。供应者用于实现懒计算：例如：

```java
LocalDate hireDay = Objects.requireNonNullOrElse(day, new LocalDate(1970,1,1));
```



这不是最优的，我们与day很少为null，所以希望只在必要时才构造默认的LocalDate，通过使用供应者，我们就能延迟这个计算：

```java
LocalDate hireDay = Objects.requireNonNullOrElseGet(day, () -> new LocalDate(1970,1,1));
```

requireNonNullOrElseGet 方法只在需要值时才调用供应者。

## 方法引用

有时，lambda表达式涉及一个方法。例如：

```java
var timer = new Timer(1000, event -> System.out.println(event));
```

但是，如果直接把println方法传递到Timer构造器就更好了。具体做法如下：

```java
var timer = new Timer(1000,System.out::println);
```



表达式System.out::println是一个方法引用（method reference），他只是编译器生成一个函数式接口的实例，覆盖整个接口的抽象方法来调用给定的方法。在这个例子中，会生成一个ActionListener，它的actionPerformed(ActionEvent e) 方法要调用 System.out.println(e)。

注释：类似于lambda表达式，方法引用也不是一个对象。不过，为一个类型为函数式接口的变量赋值时会生成一个对象。

再来看一个例子，假设相对字符串进行排序，而不考虑字母的大小写，可以传递以下方法表达式：

```java
Arrays.sort(strings,String:compareToIgnoreCase)
```



小结：要用 :: 运算符分割方法与对象或者类名。主要有3种情况：

1. object::instanceMethod
2. Class::instanceMethod
3. Class::staticMethod

在第1种情况下，方法引用等价于向方法传递参数的lambda表达式。对于System.out::println，对象是System.out，所以方法表达式等价于 x -> System.out.println(x)。

对于第2种情况，第1个参数会成为方法的隐式参数。例如，String::compareToIgnoreCase

等同于(x, y) -> x.compareToIgnoreCase(y)。

在第3种情况下，所有参数都会传递到静态方法：Math::pow等价于(x, y) -> Math.pow(x, y)

注意，只有当lambda表达式的体只调用一个方法而不做其他操作时，才能把lambda表达式重写为方法引用。考虑以下表达式：

```java
s -> s.length == 0
```

这里有一个方法调用。但是还是有一个比较，所以这里不能使用方法引用。



## 构造器引用

构造器引用与方法引用很类似，只不过方法名为new。例如，Person::new是Person构造器的一个引用。

可以用数组类型简历构造器引用。例如，int[]::new是一个构造器引用，他有一个参数：即数组的长度。这等价于lambda表达式x -> new int[x]。

Java有一个限制，无法构造泛型类型T的数组。但是利用数组构造器可以克服这个限制，例如：

Person[ ] people = stream.toArray(Person[ ]::new);

toArray方法调用这个构造器来得到一个有正确类型的数组。然后填充并返回这个数组。



## 变量作用域

lambda 表达式有3个部分：

1. 一个代码块；
2. 参数；
3. 自由变量的值，这里指非参数而且不在代码中定义的变量

lambda表达式中访问外围方法或者类中的变量。lambda表达式中捕获的变量必须实际上是事实最终变量（effectively final），事实最终变量是指，这个变量初始化之后就不会再为它赋新值。在下面例子中，text总是指示同一个String对象，所以捕获这个变量是合法的。

```java
public static void repeatMessage(String text){
  ActionListener listener = event -> {
		   System.out.println(text);
   }

}
```



在lambda表达式中，只能引用值不会改变的变量，（因为，如果在lambda表达式中更改变量，并发执行多个动作是就会不安全）下面这种做法是不合法的：

```java
public static void countDown(int start){
  ActionListener listener = event -> {
     start--; //ERROR：Can’t mutate captured variable
     System.out.println(start);

  }
}
```



如果在lambda表达式中引用一个变量，而这个变量可能在外部改变，这也是不合法的。例如：

```java
public static void repeat(String text, int count){
  for(int i = 1; i <= count; i++){
     ActionListener listener = event -> {   
       //i的值会改变，因此不能捕获i
       System.out.println(String.valueOf(i) + text);
     }
   }
}
```



注：

1. lambda表达式的体与嵌套块有相同的作用域。所以这里同样适用命名冲突和遮蔽的有关规则。在lambda表达式中声明与一个局部变量同名的参数或局部变量是不合法的。
2. 在一个lambda表达式中使用this关键字时，是指创建这个lambda表达式的方法的this参数

```java
public class Application{
   public void init(){
    ActionListener listener = event -> {  
      System.out.println(this.toString());
     }
   }
}
```

表达式this.toString()会调用Application对象的toString方法，而不是ActionListener实例的方法。

## 处理lambda表达式

如何编写方法处理lambda表达式?，使用lambda表达式的重点是延迟执行

1. 需要选择一个合适的函数式接口
2. 可以选择自定义函数式接口，建议使用@FunctionalInterface注解标记这个接口，这样如果无意中增加了另一个抽象方法，编译器就会产生一个错误消息



## 再谈Comparator

Comparator接口包含很多方便的静态方法来创建比较器，这些方法可以用于lambda表达式或方法引用。



## 参考资料

Core Java Volume I — Fundamentals (Eleventh Edition)