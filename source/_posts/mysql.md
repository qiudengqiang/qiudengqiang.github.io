---
title: MySQL 的基本使用
date: 2019-05-09 10:01:47
tags: [MySQL]
categories: DataBase
---

## SQL的分类
### DDL(Data Definition Language)
数据定义语言，用来定义数据库对象：数据库，表，列等。关键字：create，drop，alter 等
### DML(Data Manipulation Language)  
数据操作语言，用来对数据库表中的数据进行增删改。关键字：insert，delete，update 等
### DQL(Data Query Language)
数据查询语言，用来查询数据库表中的记录（数据）。关键字：select，where 等
### DCL(Data Control Language)
用来定义数据库的访问权限和安全级别，及创建用户。关键字：GRANT，REVOKE 等

## DDL 操作数据库(CRUD)
### C(Create)
```mysql
create database dbname;
create database if not exist dbname;
create database dbname character set utf8;
create database if not exist dbname character set utf8;
```
### R(Retrieve)
```mysql
-- 查询所有数据库的名称
show databases;
-- 查询某个数据库的字符集/创建语句
show crate database dbname;
```
### U(Update)
```mysql
alter database dbname character set utf8;
```
### D(Delete)
```mysql
drop database dbname;
drop database if  exist dbname;
```
### 使用数据库
```mysql
-- 查询当前正在使用的数据库
select database();
-- 使用指定的数据库
use dbname;
```
## DML 操作表
### C(Create)
```mysql
create table tbname(
    列名1 数据类型1，
    列名2 数据类型2，
    ...
    列名n 数据类型n
);
-- 赋值一张表
create table 表名 like 被复制的表名;
```
- 数据类型：
1. `int`:整数类型
    * age int
2. `double`:小数类型
    * score double(5,2)
3. `date`:日期，只包含年月日，yyyy-MM-dd
4. `datetime`:日期，年月日时分秒 yyyy-MM-dd HH:mm:ss
5. `timestamp`:时间戳类型，包含年月日时分秒 yyyy-MM-dd HH:mm:ss（如果将来不给这个字段赋值，或者赋值为Null，则默认使用当前的系统时间来自动赋值）
6. `varchar`:字符串类型
    * name varchar(20):姓名最多20个字符
    * zhangsan 8个字符，张三 2个字符

### R(Retrieve)
```mysql
-- 查询数据库中所有表名称
show tables;
-- 查询表结构
desc tbname;
```
### U(Update)
```mysql
-- 修改表名
alter table tbname rename to newtbname;
-- 查看表的字符集
show create table tbname;
-- 修改表的字符集
alter table tbname character set utf8;
-- 添加一列
alter table tbname add column_name varchar(10);
-- 修改列名称、数据类型
alter table tbname change gender sex varchar(20);
-- 只修列的数据类型
alter table tbname modify sex varchar(10);
-- 删除列
alter table tbname drop column_name;
``` 
### D(Delete)
```mysql
drop table tbname;
drop table if exists tbname;
``` 

## DML 操作表
### 添加数据
```mysql
insert into tbname(列名1,...列名n) values(值1,...值n);
```
**注意**：
1. 列名要和值名一一对应，如果表名后不指定列名，则默认给所有列添加值。
2. insert into tbname values(值1,...值n);
3. 值除了数字以外，其他都需要用引号（单双都可以）引起来

### 删除数据
```mysql
delete from tbname [where condition];
```
注意：
- 如果不加条件，则删除表中所有的记录
- 如果要删除所有记录
    1. delete from tbname; -- 不推荐使用，因为有多少条记录就会执行多少次删除操作
    2. truncate table tbname; -- 推荐使用，先删除表，在创建一张一样的空表；效率更高

### 修改数据
```mysql
update tbname set 列名1 = 值1,列名2 = 值2,...[where condition];
```
**注意**：如果不加任何条件，则会将表中所有记录全部修改。

## DQL 查询语句
```mysql
select * from tbname;
```
### 排序查询
- 语法：`order by 子句;` 
```mysql
order by 排序子句1 排序方式1,...排序子句2 排序方式n;
```
- 排序方式
    `ASC`：升序，默认的。
    `DESC`：降序。

**注意**：如果有多个排序条件，则当前面的条件值一样时候，才会判断第二条件，以此类推。

### 聚合函数
将一列数据作为一个整体，进行纵向的计算。
1. count：计算个数
```mysql 
select count(id) from tbname;
```
   * 一般选择非空的列：主键
   * count(*) -- 不建议使用

2. max：计算最大值
```mysql 
select max(列名) from tbname;
```
3. min：计算最小值
```mysql 
select min(列名) from tbname;
```
4. sum：计算和
```mysql 
select sum(列名) from tbname;
```
5. avg：计算平均值
```mysql 
select avg(列名) from tbname;
```

**注意**：聚合函数的计算，排除null值
**解决方案**：
1. 选择不包含空的列进行计算
2. `IFNULL`函数
```mysql
select count(IFNULL(列名,0)) from tbname;
```

### 分组查询
- 语法：`group by 分组字段`

**注意**：
- 分组之后查询的字段：分组字段，聚合函数

- `where` 和 `having` 的区别
1. `where` 在分组之前进行限定，如果不满足条件，则不参与分组。
2. `having` 在分组之后进行限定，如果不满足结果，则不会被查询出来
3. `where` 后可以跟聚合函数，`having` 可以进行聚合函数的判断

### 分页查询
1. 语法：`limit 开始索引,每页查询条数;`
2. 公式：开始的索引 = （当前的页码 - 1）* 每页显示的条数
eg：-- 每页显示三条记录
```mysql
SELECT * FROM TBNAME LIMIT 0,3; -- 第1页
SELECT * FROM TBNAME LIMIT 3,3; -- 第2页
SELECT * FROM TBNAME LIMIT 6,3; -- 第3页
```
3. `limit`是一个mysql的「方言」

## DQL 查询表中的语句 
- select * from tbname;

### 语法
```mysql
select 
    字段列表
from 
    表名列表
where 
    条件列表
group by 
    分组字段
having
    分组之后的条件
order by 
    排序子句列表
limit
    分页限定
```

### 基础查询
1. 多个字段的查询
```mysql
select 字段名1,...字段名n from tbname;
```
**注意**：如果查询所有字段，则可以用*来代替字段列表
2. 去除重复：`distinct`
3. 多列的值计算
* 一般可以使用四则运算计算一些列的值。
* `ifnull(表达式1,表达式2)`：null参与的运算，计算结果都为null
    表达式1:那个字段需要判断是否为Null
    表达式2：如果该字段为Null后的替换值

4. 别名：`as`，也可以省略用空格替代。

### 条件查询
1. where子句后跟条件
2. 运算符
`>、<、<=、>=、=、<>`
`BETWEEN...AND...`
`IN(集合)`、`NOT IN(集合)`
`IS NULL` 、`IS NOT NULL`
`AND / &&`
`OR / ||`
`NOT / !`
`LIKE`：模糊查询。占位符有：`_`表示单个任意字符，`%`表示多个任意字符。

- 用法示例:
```mysql
-- 查询年龄大于20岁
SELECT  * FROM tbname WHERE age > 20;
SELECT  * FROM tbname WHERE age >= 20;
-- 查询年龄等于20岁
SELECT  * FROM tbname WHERE age = 20;
-- 查询年龄不等于20岁
SELECT  * FROM tbname WHERE age != 20;
SELECT  * FROM tbname WHERE age <> 20;
-- 查询年龄大于等于20，小于等于30
SELECT  * FROM tbname WHERE age >= 20 && age <= 30;
SELECT  * FROM tbname WHERE age >= 20 AND age <= 30;
SELECT  * FROM tbname WHERE BETEEN 20 AND 30;

-- 查询年龄为18，20，30岁的人
SELECT  * FROM tbname WHERE age = 18 || age = 20 || age = 30;
SELECT  * FROM tbname WHERE age = 18 OR age = 20 OR age = 30;
SELECT  * FROM tbname WHERE age IN (18,20,30);
-- 查询年龄不为18，20，30岁的人
SELECT  * FROM tbname WHERE age NOT IN (18,20,30);

-- 查询英语成绩为null
SELECT  * FROM tbname WHERE english IS NULL;
-- 查询英语成绩为不null 
SELECT  * FROM tbname WHERE english IS NOT NULL;

-- 查询姓马的人
SELECT * FROM tbname WHERE name LIKE "马%";
-- 查询姓名中第二个字为化的人
SELECT * FROM tbname WHERE name LIKE "_化%";
-- 查询姓名中包含化的人
SELECT * FROM tbname WHERE name LIKE "%化%";
```

### 多表查询
- 内连接
    显式内连接：
            select 字段列表 from 表名1 inner join 表名2 on 条件
    隐式内连接：
            使用 where 条件消除无用数据
- 外链接
    左外连接：
        select 字段列表 from 表1 left [outer] join 表2 on 条件;
        (查询的是左表所有字段以及其交集部分)
    右外链接
        select 字段列表 from 表1 right [outer] join 表2 on 条件;
        (查询的是右表所有字段以及其交集部分)

- 子查询
**概念**：查询中嵌套查询，成嵌套查询为子查询。
1. 子查询的结果是`单行单列`
    子查询可以作为条件，使用运算符去判断。运算符：`> / >= / < / <= / =`
    eg：
    ```mysql
    select * from emp where emp.salary < (select avg(salary) from emp);
    ```
2. 子查询的结果是`多行单列`
    子查询可以作为条件，使用运算符 `IN` 去判断
    eg：
    ```mysql
    select * from emp where dept_id in (select id from dept where name in('财务部','市场部'));
    ```
3. 子查询的结果是`多行多列`
    子查询可以作为一张虚拟表参与查询
    eg：
    ```mysql
    select * from dept t1,(select  * from emp where emp.`join_date` > '2011-11-11') t2
    where t1.id = t2.dept_id;
    ```

## 约束(Constriant)
对表中的数据进行限定，保证数据的正确性，完整性和有效性。
### 非空约束
`not null`
```mysql

-- 1. 创建表时添加约束
create table tbname(
    id int;
    name varchar(20) not null
);

-- 2. 删除name的非空约束
alter table tbname modify name varchar(20);

-- 3. 创建完表后，添加非空约束
alter table tbname modify name varchar(20) not null;
```

### 唯一约束
`unique`,值不能重复
```mysql
-- 1. 创建表时，添加唯一约束
create table tbname(
    id int.
    phone_number varchar(20) unique
);

-- 2.删除唯一约束
-- 错误写法 drop table tbname modify phone_number varchar(20);
alter table tbname drop index phone_number;

-- 3. 创建表之后添加唯一约束
alter table tbname modify phone_number varchar(20);
```

### 主键约束
`primary key`，非空而且唯一，一张表只能有一个主键字段，唯一标识。
```mysql
-- 1. 创建表时，添加主键约束
create table tbname(
    id int primary key,
    name varchar(20)
);

-- 2. 删除主键约束
-- 错误写法：alter table tbname modify id int;
alter table tbname drop primary key;

-- 3. 创建完表后，添加主键约束
alter table tbname modify id int primary key;
```
### 自增长
`auto_increment`
```mysql
-- 1. 创建表时添加主键并且指定为自增长
create table tbname(
    id int primary key auto_increment,
    name varchar(20)
);
-- 2. 删除自增长
alter table tbname modify id int;
-- 3. 创建完表后添加自增长
alter table tbname modify id int auto_increment;
```
**注意**：一般情况下`auto_increment` 与`primary key` 搭配使用，但是它也是可以单独使用的。

### 外键约束
`foreign key`，让表与表产生关系，从而保证数据的正确性。
```mysql
-- 1. 在创建表时，可以添加外键。

create table tbname(
    .....
    外键列
    constriant 外键名称 foreign key(外键列名称) references 主表名称(主表列名称)
);

-- 2. 删除外键
alter table tbname drop foreign key 外键名称;

-- 3. 创建完表之后，添加外键
alter table tbname add constraint 外键名称 foreign key(外键列名) references 主表名称(主表列名);
```

### 级联操作
在操作相关联的表的数据时候，会同步更新或者删除
1. 添加级联
```mysql
alter table tbname add constraint 外键名称 foreign key(外键列名) references 主表名称(主表列名) on update cascade on delete cascade;

```
2. 分类
    级联更新：`on update cascade`
    级联删除：`on delete cascade`

## 数据库的设计
### 多表之间的关系
- 一对一 <1:1>
         eg：人和身份证
实现方式：一对一关系的实现，可以在任意一方添加唯一外键指向另一方的外键。一般还是建议合并为一张表。
- 一对多（多对一）<1:n/n:1>
         eg：部门和员工
实现方式：在多的一方建立外键，指向一的一方的主键。
- 多对多 <m:n>
         eg：学生和课程
实现方式：多对多关系实现需要借助第三章中间表，中间表至少包含两个字段，这两个字段作为第三张表的外键，分别指向两张表的主键。也可以使用联合主键。

### 数据库设计的范式
- 第一范式（1NF）：每一列都是不可分割的原子数据项。
- 第二范式（2NF）：在1NF的基础上，非码属性必须完全依赖于码（在1NF的基础上非主属性对主码的部分函数依赖）
1. `函数依赖`：A-->B，如果A属性（属性组）的值，可以确定唯一的B属性的值，则称B依赖于A。
           eg：学号-->姓名。（学号，课程名称）-->分数
2. `完全函数依赖`：A-->B，如果A是一个属性组，则B属性值的确定需要依赖于A属性组中所有的属性值。
              eg：（学号，课程名称）-->分数
3. `部分函数依赖`：A-->B，如果A是一个属性组，则B属性值的确定只需要依赖A属性组中某一些值即可。
              eg：（学号，课程名称）-->姓名
4. `传递函数依赖`：A-->B，B-->C；如果A属性（属性组），可以唯一确定B属性的值；B属性（属性组）的值可以唯一确定C属性的值，则称C传递函数依赖于A
              eg：学号-->系名，系名->系主任
5. `码`：如果在一张表中，一个属性或者属性组，被其他所有属性所完全依赖，则称这个属性（属性组）为该表的码
    `主属性`：码属性组中的所有属性
    `非主属性`：除过码属性组的所有属性
- 第三范式（3NF）：在2NF基础上，任何非主属性不依赖于其他非主属性（在2NF基础上消除传递依赖）

## 数据库的备份和还原
### 备份
```mysql
mysql> mysqldump -u用户名 -p密码 > 保存的路径
```
### 还原
1. 登录数据库 mysql -uroot -proot 
2. 创建数据库 create database dbname
3. 使用数据库 use dbname
4. 执行以下命令：
```mysql
mysql> source 文件路径
```

## 事务
### 事务的基本介绍
- 概念：如果一个包含多个步骤的业务操作，被事务管理，那么这些操作要么同时成功，要么同时失败。
- 操作：
1. 开启事务：`start transaction;`
2. 回滚：`rollback;`
3. 提交：`commit;`
4. 事务的两种提交方式
    自动：MySQL 就是自动提交的，一条DML（增删改）语句会自动提交一次事务。
    手动：Oracle 数据库默认手动提交事务。需要先开启事务再提交。
    修改事务的提交方式：
            select @@autocommit;-- 1代表自动，0代表手动
            set @@autocommit = 0;

### 事务的四大基本特征
1. 原子性：是不可分割的最小操作单位，要么同时成功，要么同时失败。
2. 持久性：当事务提交或回滚后，数据库会持久化的保存数据。
3. 隔离性：多个事务之间，相互独立。
4. 一致性：事务操作前后，数据总量不变。

### 事务的隔离级别
- 概念
多个事务之间是独立的，相互隔离的；但是如果多个事务操作同一批数据，则会引发一些问题，设置不同的隔离级别就可以解决这些问题。
- 存在问题
1. 脏读：一个事务，读取到另外一个事务中没有提交的数据。
2. 不可重复读（虚读）：在同一个事务中，两次读取到的数据不一样。
3. 幻读：一个事务操作（DML）数据表中所有事物，另一个事务添加了一条数据，则第一个事务查询不到自己的修改。
- 隔离级别
1. `read uncommitted`：读未提交；产生的问题：脏读、不可重复读、幻读
2. `read committed`：读已提交（Oracle 默认）；产生的问题：不可重复读、幻读
3. `repeatable read`：可重复读（MySQL 默认）；产生的问题：幻读
4. `serializable`：串行化；可以解决所有问题

**注意**：隔离级别从小到大，安全性越来越高，但是效率越来越低。
```mysql
-- 查询隔离级别
select @@tx_isolation;
-- 设置隔离级别
set global transaction isolation level 级别字符串;
```
## DCL 管理、授权用户
### 用户管理
1. 添加用户
```mysql
create user '用户名'@'主机名' identified by '密码';
```
2. 删除用户
```mysql
drop user '用户名'@'主机名';
```
3. 修改用户密码
```mysql
update user set password  = password('新密码') where user = '用户名';
set password for '用户名'@'主机名' = password('新密码');
```
4. 查询用户
```mysql
-- 切换到mysql数据库
use mysql;
-- 查询user表
select * from user;
```
**注意**：通配符`%`表示可以在任意主机使用该用户登录数据库

### 权限管理
1. 查询权限
```mysql
show grant for '用户名'@'主机名';
```
2. 授予权限
```mysql
grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';

-- 给张三用户授予所有权限，在任意数据库任意表上。
eg:grant all on *.* to 'zhangsan'@'localhost';
```
3. 撤销权限
```mysql
revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名';
```

## 修改 mysql 设置
### 更改默认编码
查看 mysql 数据库的默认编码命令：
```bash
mysql> show variable like 'character%';

+--------------------------+--------------------------------------------------------+
| Variable_name            | Value                                                  |
+--------------------------+--------------------------------------------------------+
| character_set_client     | utf8                                                   |
| character_set_connection | utf8                                                   |
| character_set_database   | latin1                                                 |
| character_set_filesystem | binary                                                 |
| character_set_results    | utf8                                                   |
| character_set_server     | latin1                                                 |
| character_set_system     | utf8                                                   |
| character_sets_dir       | /usr/local/mysql-5.7.10-osx10.9-x86_64/share/charsets/ |
+--------------------------+--------------------------------------------------------+
8 rows in set (0.01 sec)
```
可见 database 和 server 的字符集使用了 `latin1` 编码方式，不支持中文，即存储中文时会出现乱码。以下修改方法：

1. cd /private/etc/
2. 新增一个 my.cnf 文件
3. 然后写入以下命令保存退出
```bash
[client]
default-character-set=utf8
 
[mysql]
default-character-set=utf8
 
[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
```
4. 重启 mysql

### 忘记root用户密码
1. net stop mysql
2. mysqld --skip-grant-tables(使用无验证服务启动mysql服务)
3. 打开新的终端窗口，直接输入 `mysql` 命令
4. use mysql;
5. update user set password = password('新密码') where user = 'root';
6. 关闭两个窗口，并结束 `mysqld` 进程
7. 启动 `mysql` 服务
8. 使用新密码登录
