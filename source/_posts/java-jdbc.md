---
title: JDBC 基本使用
date: 2019-05-25 10:01:47
tags: [Java,JDBC]
categories: DataBase
---

JDBC：Java DataBase Connection，简言就是使用 Java 语言操作数据库。

## 快速入门
```java
    Class.forName("com.mysql.jdbc.Driver");
    Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/db2", "root", "root");
    Statement statement = connection.createStatement();
    String sql = "update account set balance = 500 where id = 1";
    int count = statement.executeUpdate(sql);
    System.out.println(count);
    connection.close();
    statement.close();
```

## 详解各个类
### DriverManager 
1. 驱动管理对象

注册驱动：通知数据库该使用哪一个数据库驱动的jar
```java
//注册给定的驱动程序 DriverManeger
static void registerDriver(Driver driver)
//写代码时使用
Class.forName("com.mysql.jdbc.Driver");
```
通过查看源码发现：在 `com.mysql.jdbc.Driver` 类中存在静态代码块
```java
static {
    try{
        java.mysql.DriverManeger.registerDriver(new Driver());
    }catch(SQLException e){
        throw new RuntimeException("Can't register driver!")
    }
}
```
2. 获取数据库连接
- 方法：
```java
static Connection get Connection(String url, String user, String password)
```
- 参数：
`url`：指定连接的路径
        语法：jdbc:mysql//ip地址:端口号/数据库名称
        例子：jdbc:mysql//localhost:3306/db2
        细节：如果连接的是本机mysql服务器，并且默认端口为3306，则url可以简写为：jdbc:mysql:///数据库名称
`user`:用户名
`password`:密码

### Connection
数据库连接对象
1. 获取执行sql对象
```java
Statement createStatement()
PreparedStatement prepareStatement(String sql)
```
2. 管理事务
开启事务：`setAutoCommit(boolean autoCommit)`，调用该方法设置参数为false开启事务。
提交事务：`commit()`
回滚事务：`rollback()`

### Statement
用于执行sql的对象
1. `boolean execute(String sql)`，可以执行任意的sql
2. `int executeUpdate(String sql)`，执行DML(insert、update、delete)语句、DDL(create、alter、drop)语句；返回值是影响的行数，可以用来判断是否执行成功。
3. `ResultSet executeQuery(String sql)`：执行DQL（select）语句

### ResultSet
结果集对象
`boolean next()`:游标向下移动一行，判断当前行是否是末尾（是否有数据），如果是则返回false，如果不是则返回true
`getXxx(参数)`:获取数据，Xxx代表数据类型，如：getString()、getInt()；参数可以使用列编号也可以使用列名
```java
while(rs.next()){
    int id = rs.getInt(1);
    String name = rs.getString("name");
    double banlance = rs.getDouble(3);
}
```

### PreparedStatement
用于执行安全执行sql的对象，防止注入问题
1. SQL注入问题：在拼接sql时，有一些sql的特殊关键字参与字符串的拼接，会造成安全问题。
2. 解决SQL注入问题：使用`PreparedStatement`对象来解决
3. 预编译的SQL：参数使用`?`作为占位符
4. 使用
    - 获取执行SQL语句的对象： 
    ```java
    PreparedStatement Connection.prepareStatement(String sql)
    ```
    - 给`?`赋值：
    ```java
    setXxx(参数1，参数2)，
        参数1：? 位置的编号，从1开始
        参数2：? 的值
    ```
5. 注意：`PreparedStatement` 可以防止sql注入，且效率更高。

## JDBC 控制事务
使用`Connection`对象来管理事务
开启事务：`setAutoCommit(boolean autoCommit)`，参数设置为false，即开启事务。一般在sql执行之前开启事务
提交事务：`commit()`，当所有 sql 执行完提交事务
回滚事务：`rollback()`，在 catch 中回滚事务
设置回滚点：`setSavePoin()`
回滚到回滚点：`rollback(sp)`

## JDBC 连接池
频繁向系统申请连接数据库的操作是一个相对耗时的操作。为了提高效率并节省资源，引入数据库连接池的概念，而 JDBC 连接池其实就是一个用于存放数据库连接的容器（集合）；当系统初始化完毕之后，容器就会被创建，容器中会申请一些连接对象；当用户访问数据库时，从容器中获取连接对象；访问完毕将连接归还给容器。使得更节约资源，访问更高效。

### DataSource
 DataSource 是 `javax.sql` 包下操作数据库连接池的标准接口，其提供统一的获取和释放数据库连接池的操作。
 - `getConnection()`：获取连接
 - `Connecion.close()`：归还至连接池

### C3P0
C3P0 是 Apache 一个开源的 JDBC 连接池，它实现了 DataSource 和 JNDI 绑定，支持 JDBC3 规范和 JDBC2 的标准扩展。 目前使用它的开源项目有Hibernate，Spring等。
使用步骤：
1. 导入jar包（2个）`c3p0-0.9.5.2.jar` 及其依赖包 `mchange-commons-java-0.2.12.jar`
2. 导入数据库驱动 jar 包
3. 定义配置文件：
    名称：`c3p0.properties`/`c3p0-config.xml`
    路径：直接放在 `src` 目录下
4. 创建核心数据库连接对象 `ComboPooledDateSource`
5. 获取连接：getConnection()

### Druid
Druid是一个阿里巴巴提供的JDBC组件库，Druid是一个JDBC组件库，包括数据库连接池、SQL Parser等组件。 DruidDataSource 高效可管理的数据库连接池。
[https://github.com/alibaba/druid](https://github.com/alibaba/druid)
使用步骤：
1. 导入 jar 包 druid-1.0.9.jar
2. 定义配置文件：`任意名字.properties`放在任意目录下
3. 加载配置文件
4. 获取数据库连接池对象 `DruidDataSourceFactory`
5.获取连接：getConnection()

## Spring JDBC
Spring 框架对JDBC的简单封装，提供了一个 JdbcTemplate 对象简化JDBC的开发
使用步骤：
1. 导入jar包
2. 创建 JdbcTemplate 对象，依赖于数据源 Datasource
```java
JdbcTemplate template = new JdbcTemplate(ds);
```
3. 调用 JdbcTemplate 方法来完成 CRUD 的操作
- update()：执行 DML 增删改语句
- queryForMap()：查询结果并将结果集封装为 Map 集合，将列名作为 key ，将值作为 value ，将这条记录封装为一个 Map 集合。注意这个方法查询的结果集长度为 1
- queryForList()：查询结果并将结果集封装到 List 集合中，其中每一条记录封装为 Map 集合，再将 Map 装在到 List
- query(参数)：查询结果并将结果封装为 JavaBean 对象
    参数：接口 RowMapper、实现类 BeanPropertyRowMapper(可以实现数据到 JavaBean 的自动封装)
    ```java
    new BeanPropertyRowMapper<类型>(类型.class);
    ```
- queryForObject()：查询结果并将结果封装为对象，一般用于聚合函数的查询