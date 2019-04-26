---
title: IDEA常用快捷键&编码小技巧
date: 2019-04-11 10:01:47
tags: [IDEA]
categories: IDEA
---

## IDEA 常用快捷键
1. `Alt+Enter` ：自动修正代码
2. `Option+/`：自动补全提示
3. `Cmd+F12` 显示当前类的方法和变量列表
4. `Shift+F6` 重构名称

### Debug
1. F8：逐行执行程序
2. F7：进入到方法中
3. Shift+F8：跳出方法
4. F9：调到下一个断点
5. Ctrl+F2：退出debug模式，停止程序

## IDEA 编写代码小技巧
1. 方法.var ：.var的使用可以直接创建好getInstance方法的返回值对象
    eg:
    ```java
      Calendar.getInstance().var 
    ```
2. fori ：可以直接`list.fori`
    eg：
    ```java
    ArrayList<Integer> list = new ArrayList<>();
    list.fori //生成下面的代码
       /* for (int i = 0; i < list.size(); i++) {
            
        }*/
    ```
3. `psvm`：自动生成main方法
4. `sout`：自动生成打印语句
5. foreach：`lists.for`