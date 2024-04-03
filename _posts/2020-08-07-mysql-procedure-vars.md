---
layout: post
title: MySQL存储过程——变量
categories: [MySQL]
description: MySQL存储过程中变量的使用
keywords: MySQL, 存储过程, 变量
---

工作需要使用MySQL的存储过程批量修改一些数据，被MySQL的变量坑了好久，特此记录，以防再次踩坑。

#### 用户变量

###### 声明

```plsql
declare @var_name [,variable_name...] datatype [DEFAULT value]; --[] 表示可选
declare @var_name int default 0; 
declare @var_name varchar(32) default null;
```

###### 赋值

```sql
set  @var_name = 1;
set  @var_name := 1;
```

```sql
select  @var_name, @var_name1:=2;
select  col1 := @var_name ,col2 := @var_name1 from table1;
select col1, col2 into @var_name, @var_name1 from table1;  
```

- 使用``set``方式赋值可使用 ``= ``或者 ``:=``
- 使用``select``方式赋值只能使用`` := ``.，否则不会报错，但无法赋值成功
- 用户变量可以不声明直接赋值即可使用

###### 使用

```sql
select  @var_name1; -- 或直接使用@var_name1即可
```

###### 作用域

- 当前会话，在同一个终端窗口重复执行一个定义了用户变量的存储过程，可以拿到上一次执行时的值

#### 局部变量

###### 声明

```plsql
declare variable_name [,variable_name...] datatype [DEFAULT value]; --[] 表示可选
declare var_name int default 0; 
declare var_name varchar(32) default null; 
```

- 局部变量必须声明类型，一些类型如``varchar``还必须定义长度

###### 赋值

```sql
set  var_name = 1;
set  var_name := 1;
```

```sql
select  var_name, var_name1:=2;
select  col1 := var_name ,col2 := var_name1 from table1;
select col1, col2 into var_name, var_name1 from table1;  
```

###### 使用

```sql
select  var_name1; -- 或直接使用var_name1即可
```

###### 作用域

- 当前的存储过程，类似Java的局部变量，仅在当前存储过程中有效

#### 系统变量

###### 使用

		分全局变量和会话变量，会话变量只在当前会话中有效，全局变量在所有会话中均有效

- 查询会话变量

```sql
show session variables;
show variables；
```

- 查询全局变量

```sql
show global variables;
```



###### 自定义系统变量

```sql
set GLOBAL sort_buffer_size=value;
set @@global.sort_buffer_size=value;
```

- 系统变量没有声明
- 系统许多的配置项均为系统变量
- 个人建议，编程尽量避免自定义系统变量

#### 总结
- 系统变量均为MySQL系统配置，不建议自定义系统变量
- 除了声明和使用方式等硬性区别，各个变量的主要差异在于作用域