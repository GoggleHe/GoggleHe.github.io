---
layout: post
title: MySQL存储过程——入参与出参
categories: [Blog]
description: MySQL存储过程中变量的使用
keywords: MySQL, 存储过程,参数
---

MySQL存储过程的参数的小案例和一些小总结

### 入参``IN``

###### 声明

```mysql
DROP PROCEDURE IF EXISTS FUNC;
CREATE PROCEDURE FUNC(IN param INT,IN param2 INT) 
BEGIN
	SELECT param,param2;
	SET param := param2;
	SELECT param,param2;
END;
```

######  值调用

```mysql
CALL FUNC(1,2);
```

![mysql-procedure-parameters-in](/images/blog/mysql-procedure-parameters-in.png)

###### 变量调用

```mysql
select @in1,@in2;
call func(@in1,3);
select @in1,@in2;
```

![mysql-procedure-parameters-in2](/images/blog/mysql-procedure-parameters-in2.png)

- 可见存储过程的``in``类型入参仅解析了变量的值，而不会变量本身产生副作用

### 出参``OUT``

###### 声明

````mysql
DROP PROCEDURE IF EXISTS FUNC;
CREATE PROCEDURE FUNC(OUT out_var1 INT,OUT out_var2 INT) 
BEGIN
	SELECT out_var1,out_var2;
	SET out_var1 := 123;
	SET out_var2 := 456;
	SELECT out_var1,out_var2;
END;
````

###### 调用

```mysql
select @out1 := 888,@out2 := 999; -- 初始化
CALL FUNC(@out1,@out2); -- 执行存储过程
select @out1,@out2; -- 查询变量调用存储过程后的值
```

###### 输出结果

![mysql-procedure-parameters-out](/images/blog/mysql-procedure-parameters-out.png)

- ``out``类型的参数只能传入变量
- ``out``类型的参数并不会解析变量的值，如果存储过程有对``out``类型的参数赋值，该赋值会影响作为参数的变量

### 出入参``INOUT``

###### 声明

```mysql
DROP PROCEDURE IF EXISTS FUNC;
CREATE PROCEDURE FUNC(INOUT in_out_var1 INT,INOUT in_out_var2 INT) 
BEGIN
	SELECT in_out_var1,in_out_var2;
	SET in_out_var1 := 123;
	SET in_out_var2 := 456;
	SELECT in_out_var1,in_out_var2;
END;
```

###### 调用

```mysql
select 888,999 into @inout1,@inout2; -- 初始化
call func(@inout1,@inout2); -- 调用
select @inout1,@inout2; -- 验证结果
```

###### 输出结果

![mysql-procedure-parameters-inout](/images/blog/mysql-procedure-parameters-inout.png)

- 可见``inout``类型参数结和了``in``和``out``类型，既会解析变量的值也会修改变量的值
- ``inout``类型的参数也只能传入变量

### 默认

###### 声明

```mysql
DROP PROCEDURE IF EXISTS FUNC;
CREATE PROCEDURE FUNC(in_out_var1 INT,in_out_var2 INT) 
BEGIN
	SELECT in_out_var1,in_out_var2;
	SET in_out_var1 := 123;
	SET in_out_var2 := 456;
	SELECT in_out_var1,in_out_var2;
END;
```
###### 调用

```mysql
select 888,999 into @inout1,@inout2; -- 初始化
call func(@inout1,@inout2); -- 调用
select @inout1,@inout2; -- 验证结果
```
###### 执行结果

![mysql-procedure-parameters-default](/images/blog/mysql-procedure-parameters-default.png)

- 解析了入参，但没有对变量产生副作用，说明默认是``in``类型参数

### 总结

| 类型  | 参数类型（值、变量） | 是否会修改变量的值（对变量是否有副作用） |
| ----- | -------------------- | ----------------------------- |
| in    | 值、变量             | 否                                       |
| out   | 变量                 | 是                                       |
| inout | 变量                 | 是                                       |

- MySQL的存储过程没有Java方法中的return，出参实际上都利用修改参数中的变量的值进行模拟