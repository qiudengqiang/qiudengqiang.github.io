---
title: Junit的使用
date: 2019-04-29 10:01:47
tags: [Java]
categories: Java
---

## 使用步骤
1. 定义一个测试类（测试用例）
- 测试类名：被测试的类名+Test   `eg: CaculatorTest`
- 包名：xxx.xxx.xxx.test     `eg: me.techbird.test`

2. 定义测试方法：可以独立运行
- 方法名：test+被测试的方法名    `eg: testAdd()`
- 返回值：`void`
- 参数列表：空参
3. 给方法加`@Test`
4. 导入junit依赖环境

## 判定结果
- 红色：失败
- 绿色：成功
- 一般可以使用断言操作来处理结果：
```java
Assert.assertEquals(期望的结果，运算的结果);
```

## 补充
- `@Before` 修饰的方法会在测试方法执行之前被自动执行
- `@After` 修饰的方法会在测试方法执行之后被自动执行