---
title: mysql
date: 2023/08/12
categories:
  - coding
tags:
  - mysql
  - 数据库
  - 编程基础
abbrlink: 13153
---

# 1、数据库基础概念
## 1.1、mysql数据类型

| **分类** | **类型名称** | **类型说明** |
| --- | --- | --- |
| 整数 | tinyInt | 微整型：占8位二进制，1个字节 （-128-127） |
| | smallint | 小整型：占16位二进制，2个字节（-32768-32767） |
|  | mediumint | 中整型：占24位二进制，3个字节（-8388608-8388607） |
|  | **int**(integer) | 整型：占32位二进制，4个字节（-2147483648-2147483647） |
|  | **bigint** | 大整型：占64位二进制，8个字节（-9223372036854775808-9223372036854775807） |
| 小数 | **float** | 单精度浮点数，占4个字节 |
|  | **double** | 双精度浮点数，占8个字节 |
|  | decimal(m,n) | 高精度小数型，最大精度为65，最大占用空间30字节 |
| 日期 | **time** | 表示时间类型 yyyy-MM-DD ，3字节|
|  | **date** | 表示日期类型 hh:mm:ss ，3字节 |
|  | **datetime** | 同时可以表示日期和时间类型，8字节 |
| 字符串 | char(m) | **固定长度的字符串，无论使用几个字符都占满全部**，M为0~255之间的整数 如：char(20)，实际使用只用了1个字符，也占用20个字符 |
|  | **varchar**(m) | **可变长度的字符串，使用几个字符就占用几个**，M为0~65535之间的整数。 如：varchar(20)，这个字符串最长是20，大于20会报错。 使用几个，占几个字符。 |

# 2、数据库基本操作
## 2.1、DDL数据定义语言
### 2.1.1、数据库相关

```sql
-- 创建数据库并指定字符集
create database db01 character set utf8;

-- 查看所有的数据库
show databases;

-- 查看某个数据库的DDL语句
show create database db01;

-- 修改数据库编码
alter database db01 character set utf8;

-- 删除数据库
drop database db01;

-- 切换数据库
drop database db01;

```
### 1.1.2、表相关

```sql
-- 创建表
create table emp (
    id int,
  	-- ` 是 MySQL 的转义符，避免和 mysql 的本身的关键字冲突，只要你不在列名、表名中使用 mysql 的保留字或中文，就不需要转义。
    sex char(1),
    `name` varchar(20), 
	address varchar(20)
);

-- 查看数据库所有表
show tables;

-- 查看表结构
desc 表名;

-- 查看创建表的sql语句
show create table emp;

-- 复制表结构
CREATE TABLE 新表 LIKE 旧表;

-- 删除表
DROP TABLE 表名; 

-- 判断表是否存在，存在删除表
DROP TABLE IF EXIST 表名;

-- 添加列
alter table emp add age int;

-- 删除列
alter table emp drop age;

-- 修改字段类型
alter table emp modify address varchar(100);

-- 修改字段名和字段类型
alter table emp change address addr varchar(80);

-- 修改表名（MySQL中没有直接修改库名的语句）
rename table emp to employee;

-- 修改表字符集
alter table emp character set utf8;
```
## 2.2、DML数据操纵语言

```sql
-- 插入全部字段
insert into emp values(1,'张三','男','河南汤阴',39);

-- 插入指定字段
insert into emp(id,name,sex) values(2,'李四','男');

-- 插入多条数据
INSERT INTO emp
VALUES
	( 4, '小红', '女', '四川南充', 25 ),
	( 5, '小明', '女', '河南平顶山', 24 ),
	( 6, '小李', '男', '陕西榆林', 20 );

-- 更新数据
update emp set sex='男',addr='陕西忻州' where id=5;

-- 删除表数据
delete from emp;

-- 清空表（先删除表结构,再创建一个相同的表结构 相当于drop table emp，再create table emp）表中有自增长，会把自增长id 重置成1开始
truncate emp;
```
## 2.3、DQL数据查询语言
### 2.3.1、基础查询

```sql
-- 查询指定列
select id,name,addr from emp;

-- 指定列的别名
select id as 编号,name as 姓名,addr 地址 from emp;

-- 去重id相同的行
select distinct id from emp;

-- 查询时进行列运算
select id,name,addr,id+age from emp;

-- 范围查询
select * from student where english between 60 and 90;

-- 模糊查询
select * from student where address like '%西%'; -- 地址中带西的数据
select * from student where address like '_____';-- 查询五个字地址城市的学生

-- 查询列为空的数据
select * from student where sex is null;

-- limit查询offset：跳过多少条记录，默认是0 length：返回多少条记录
select * from table LIMIT offset,length
select * from student limit 2, 3; -- 从第二条开始查，显示三条

-- case when（Case When语句用于选择判断，在执行时先对条件进行判断，然后根据判断结果做出相应的操作）
SELECT
	*,
CASE
		sex 
		WHEN '男' THEN
		'man' 
		WHEN '女' THEN
		'woman' 
		ELSE '其他'
	END AS 'other' 
FROM
	student;

```

| **通配符** | **说明** |
| --- | --- |
| % | 匹配零个或多个字符 |
| _ | 匹配一个字符 |

### 2.3.2、排序

```sql
-- 升序排列
select * from student order by age asc;

-- 降序排列
select * from student order by age desc;

-- 组合排序（查询所有数据大于20岁的学生,在年龄降序排序的基础上,如果年龄相同再以数学成绩升序排序）
select * from student where age > 22 order by age desc, math asc;
```

### 2.3.3、聚合函数

| **SQL中的聚合函数** | **作用** |
| --- | --- |
| count | 统计个数，如果这一列有NULL，null不会参与统计 |
| max | 找这一列中的最大值，一般是数值类型进行操作。 |
| min | 找这一列中的最小值 |
| sum | 求这一列的总和 |
| avg | 求这一列的平均，返回值小数average |


### 2.3.4、分组

分组一般搭配聚合函数一起使用

```sql
-- 查询年龄大于23岁的人，按性别分组，统计每组的人数
select sex, count(*) from student where age>23 group by sex;

-- 查询年龄大于23岁的人，按性别分组，统计每组的人数，并只显示性别人数大于2的数据
SELECT sex, COUNT(*) FROM student WHERE age > 23 GROUP BY sex having COUNT(*) >2;


```

| **子名** | **作用** |
| --- | --- |
| where 子句 | 先过滤掉行上的一些数据，再进行分组操作。(先过滤再分组) |
| having子句 | 先分组后得到的结果上再进行过滤的操作。(先分组再过滤) |

> **where子句后不能使用分组函数，having子句后可以使用聚合函数**

### 2.3.5、连接查询
#### 2.3.5.1、表的关系

| **表与表的关系** | **示例** | **关系的维护** |
| --- | --- | --- |
| 一对多 | 班级合学生 | 通过从表中外键来维护 |
| 多对多 | 学生和课程。拆分成学生，选课，课程 | 通过中间表，将两个一对多加到一起变成了一个多对多 |
| 一对一 | 在实际的开发中应用不多，因为一对一可以创建成一张表 | （1）外键添加约束 （2）从表的主键又是外键 可以简化成一张表 |

#### 2.3.5.2、笛卡尔积

简单的说就是两个集合相乘的结果。笛卡尔（Descartes）乘积又叫直积。假设集合A={a,b}，集合B={0,1,2}，则两个集合的笛卡尔积为{(a,0),(a,1),(a,2),(b,0),(b,1), (b,2)}

```sql
-- 笛卡尔查询示例
SELECT * FROM table1 , table2
```

笛卡尔积产生，有两种情况：
（1）表连接缺少关联条件，这个是必须要改的；
（2）表连接有关联条件，但是数据库判断用笛卡尔积更快，也会出现笛卡尔积，这个时候要看实际执行速度；数据库这样判断，**一般是表比较小**，这个时候要特别检查表的数据量是不是真的很少，以免oracle因为统计信息错误而误判。

#### 2.3.5.3、内连接

用左边表的记录去匹配右边表的记录，如果符合条件的则显示

1. 隐式内连接：看不到join关键字，条件使用where指定

```sql
select * from dept,emp where dept.id=emp.dept_id;
```

2. 显示内连接：**使用INNER JOIN ... ON语句, 可以省略INNER

```sql
select * from dept d inner join emp e on d.id = e.dept_id;
```

> 隐式内连接只能使用where来进行表的关联；显示内连接可以使用on和where来关联，推荐使用on


#### 2.3.5.4、左外连接

左外连接：使用LEFT OUTER JOIN ... ON，OUTER可以省略(查询的数据以左表为准，即使在其他表中没有匹配的记录也会显示出来)
```sql
-- 需要查询所有的部门和员工，无论这个部门下有没有员工
select * from dept left join emp on dept.id = emp.dept_id;
```

> 在使用left jion时，on和where条件的区别如下：
> 1、on条件是在生成临时表时使用的条件，它不管on中的条件是否为真，都会返回左边表中的记录。
> 2、where条件是在临时表生成好后，再对临时表进行过滤的条件。

#### 2.3.5.5、右外连接

右外连接：使用RIGHT OUTER JOIN ... ON，OUTER可以省略（查询的数据以右表为准，即使在其他表中没有匹配的记录也会显示出来）

```sql
select * from dept right join emp on dept.id = emp.dept_id;
```
#### 2.3.5.6、全连接
全连接：左表和右表的数据都能够显示全面呢（在对方表中没有匹配的数据就以null补齐）

> MySQL并没有提供全连接，但Oracle支持；虽然MySQL不支持全连接，但是我们可以利用MySQL提供的其它功能来完成全连接的功能：

```sql
select * from dept d left join emp e on d.id=e.dept_id
union 
select * from dept d right join emp e on d.id=e.dept_id;

```

> union关键字可以将两个或多个SQL语句的结果集拼接成一个结果集，前提是这些SQL语句的结果集列数必须相同；（union关键字自带去重功能，即去除重复的数据）


#### 2.3.5.7、子查询
一个查询语句结果做为另一个查询语句的条件查询语句有嵌套，里面的查询称为子查询，外面的查询称为父查询子查询要使用括号括起来

1. 子查询结果为单行单列

```sql
select * from emp where age = (select max(age) from emp);
```

2. 子查询结果为多行单列

查询结果是多行单列的时候，子查询的结果相当于一个集合或数组。父查询要使用in/any/all这些关键字

```sql
-- 采用in 取结果集中的数据 in (1,2,3)
select * from dept where id in (select dept_id from emp where age > 23);
-- 采用all 查询年龄大于1号部门所有员工的人
select * from emp where age > all (select age from emp where dept_id=1);
-- 采用any 比1号部门任意一个大就行
select * from emp where age > any (select age from emp where dept_id=1);
```

3. 子查询结果为多行多列

如果子查询的结果是多行多列，父查询可以将这个查询结果做为一个虚拟表，进行第2次查询。不是放在where后面，而是放在from的后面
```sql
-- 询出年龄大于23岁的员工信息和部门名称
select e.*,d.name 部门名称 from dept d, (select * from emp where age > 23) e where d.id = e.dept_id
```

## 2.4、DCL数据控制语言
DCL是数据控制语言,主要是用来设置或更改数据库用户或角色权限的语句

```sql
-- 创建用户
CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';

-- 查询数据库用户
select * from mysql.user;

-- 授权
grant create,alter,insert,update,select on db03.* to 'zhangsan'@'localhost';
grant all on *.* to 'lisi'@'%';

-- 撤销权限
revoke select on db03.* from 'zhangsan'@'localhost';

-- 查看用户权限
show grants for 'zhangsan'@'localhost';

-- 删除用户
drop user 'zhangsan'@'localhost';

-- 修改管理员密码
mysqladmin -uroot -p password 新密码

```

# 3、数据库约束

一般在创建表的时候给表的字段添加各种约束，从而保证输入到表中的数据是正确的。保证数据的正确性，完整性和有效性。违反约束的数据是不能添加到表中去的。如果表已经存在，并且表中已经有数据，添加约束的时候如果表中的数据已经违反了现在要添加的约束，约束会添加失败。

| **约束名** | **关键字** | **说明** |
| --- | --- | --- |
| 主键 | primary key | 唯一，非空 |
| 默认 | default | 没有输入值，使用默认值 |
| 非空 | not null | 必须输入 |
| 唯一 | unique | 不能重复 |
| 外键 | foregin key (外键) references 主表(主键) | 外键在从表 主表：1方 从表：多方 |

## 3.1、主键约束
用来唯一标识表中的每一行记录，在创建表的时候，每张表都应该创建一个主键，每个表只能有一个主键约束，只要有主键就有主键约束。

> 特点：非空，唯一，一张表最多一个主键

```sql
create table t1(
	id int primary key auto_increment,				-- 指定id列为主键
    city varchar(20)
)

-- 在已有的表中添加主键
alter table t1 add primary key(id);

-- 删除主键
alter table t1 drop primary key;

-- 联合主键
create table test(
	id int,
	id2 int,
	city varchar(20),
	primary key(id,id2)			-- id和id2列组合为联合主键
);
```
## 3.2、唯一约束

这一列的值不能重复

> 和主键约束区别：唯一性约束允许在该列上存在NULL值，主键约束允许

```sql
create table t3 (
	id int primary key auto_increment,
	city varchar(20) unique
);
```
## 3.3、非空约束
这一列的值必须输入，不能为空

```sql
create table t4(
	id int,
	city varchar(20) not null			-- 非空约束
);
```
## 3.4、检查约束
检查约束可以使用一定的范围条件来约束我们的列的值，例如年龄应该在0~120岁之间，性别只能有男或女等；

```sql
create table t6(
	id int,
	name varchar(30),
	age int check(age>0 and age<120),			-- 年龄只在0~120之间
	sex char(1) check('男' or '女') 			-- 性别只能在'男' 和 '女'之间选择
);
```
## 3.5、外键约束
外键出现在从表中，被主表的主键约束的那一列外键

```sql
create table employee(
	id int primary key auto_increment,
	dept_id int, -- 外键的数据类型与主表中的主键相同
	CONSTRAINT `employee_ibfk_1` foreign key (dept_id) references dept(id)		-- 本表的dept_id列依赖于dept表的id列
);

-- 新增外键约束
ALTER TABLE 表名 ADD CONSTRAINT 约束名 FOREIGN KEY (外键字段) REFERENCES 主表(主键)
```
# 4、数据库事务

如果一个业务操作中多次访问了数据库，必须保证每条SQL语句都执行成功。如果其中有一条执行失败，那么所有已经执行过的代码必须回滚（撤销）。回到没有执行前的状态。称为事务。简单来说就是要么所有的SQL语句全部执行成功，要么全部失败

| **事务特性** | **含义** |
| --- | --- |
|  原子性（Atomicity） | 一个事务内的所有操作要么都执行，要么都执行失败 |
|  一致性（Consistency） | 事务执行前与执行后，数据库中数据应该保持相同的状态 |
|  隔离性（Isolation） | 指的是一个事务在最终提交前，对其它事务是不可见的 |
|  持久性（Durability） |  如果事务执行成功，对数据库的操作是持久的。 |


## 4.1、事务提交

1. **手动提交事务**

| **功能** | **SQL语句** |
| --- | --- |
| 开启事务 | start transaction/begin |
| 提交事务 | commit |
| 回滚事务 | rollback |


```sql
-- 开启事务
start transaction;

-- a账号-500元
update account set money=money-500 where name='a';

-- b账号+500元
update account set money=money+500 where name='b';

-- 查询账号信息
select * from account;

-- 提交事务
commit;

```

2. **自动提交事务，默认是自动提交**

MySQL默认每一条DML(增删改)语句都是一个单独的事务，每条语句都会自动开启一个事务，执行完毕自动提交事务，MySQL默认开始自动提交事务

```sql
-- 取消自动提交事务
set @@autocommit = 0;

select @@autocommit;

```

## 4.2、并发访问下事务产生的问题
当同时有多个用户在访问同一张表中的记录，每个用户在访问的时候都是一个单独的事务。事务在操作时的理想状态是：事务之间不应该相互影响，实际应用的时候会引发下面三种问题：

1. 脏读： 一个事务（用户）读取到了另一个事务没有提交的数据
2. 不可重复读：一个事务多次读取同一条记录，出现读取数据不一致的情况。一般因为另一个事务更新了这条记录而引发的。
3. 幻读：在一次事务中，多次读取到的条数不一致

## 4.3、事务的隔离级别
为了尽量避免这些问题的发生。通过数据库本身的功能去避免，设置不同的隔离级别

| **级别** | **名字** | **隔离级别** | **脏读** | **不可重复读** | **幻读** | **数据库默认隔离级别** | 解释 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 读未提交 | read uncommitted | 是 | 是 | 是 |  | 事务A可以读取到事务B未提交的数据 |
| 2 | 读已提交 | read committed | 否 | 是 | 是 | Oracle和SQL Server | 事务A只能读取其它事务已提交的数据（避免了脏读） |
| 3 | 可重复读 | repeatable read | 否 | 否 | 是 | MySQL | 保证在同一个事务中多次读取同样数据的结果是一样的 |
| 4 | 串行化 | serializable | 否 | 否 | 否 |  | 事务串行化顺序执行 |

> 隔离级别越高，安全性就越高，性能越低

```sql
-- 设置隔离级别
set global transaction isolation level read uncommitted;
```

# 5、视图

视图其实就是一个select返回的结果集，用于方便我们查询而创建的"临时表"，简化我们的查询语句

```sql
create view test4 as select * from student where class_id=1 with local check option;
```
# 6、触发器

触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性 , 日志记录 , 数据校验等操作 。

| **触发器类型** | **NEW和OLD的使用** |
| --- | --- |
| insert触发器 | NEW 表示将要或者已经新增的数据 |
| update触发器 | OLD 表示修改之前的数据 , NEW 表示将要或已经修改后的数据 |
| delete触发器 | OLD 表示将要或者已经删除的数据 |

> 使用别名 OLD 和 NEW 来引用触发器中发生变化的记录内容

```sql
-- 基本语法
create trigger trigger_name [after/before] [insert/update/delete] 
on table_name
for each row
begin
	......
end;
```

```sql
-- insert触发器
create trigger test1 after insert
on student
for each row
begin
	insert into log values(null,new.name,now());
end;

-- update触发器
create trigger test2 after update
on student
 for each row
begin
	insert into log values(null,concat('之前的值: ',old.name,';之后的值: ',new.name),now());
end;

-- delete触发器
create trigger test3 after delete
on student
 for each row
begin
	insert into log values(null,concat('删除的值: ',old.name),now());
end;

-- 查看当前数据库中的触发器
show triggers;

-- 删除触发器
drop trigger test1;
```
# 7、存储过程和存储函数

MySQL中提供存储过程与存储函数机制，我们先将其统称为存储程序，一般的SQL语句需要先编译然后执行，存储程序是一组为了完成特定功能的SQL语句集，**经编译后存储在数据库中**，当用户通过指定存储程序的名字并给定参数（如果该存储程序带有参数）来调用才会执行

## 7.1、存储过程语法
### 7.1.1、变量

1. declare：声明变量

```sql
CREATE PROCEDURE test2 ()
begin
	
	declare num int default 0;		-- 声明变量,赋默认值为0
	select num+10;
	
end ;

call test2();			-- 调用存储过程

```

2. set：赋值操作

```sql
CREATE PROCEDURE test3 ()
begin
	
	declare num int default 0;
	set num =20;			-- 给num变量赋值
	select num;
	
end ;

call test3();

```

3. into：赋值

```sql
CREATE PROCEDURE test4 ()
begin
	
	declare num int default 0;			
	select count(1) into num from student;
	select num;
	
end ;

call test4();

```
### 7.1.2、if语句


```sql
-- 根据class_id判断是Java还是UI还是产品
CREATE PROCEDURE test5 ()
begin
	
	declare id int default 1;			
	declare class_name varchar(30);
	
	if id=1 then
		set class_name='哇塞，Java大佬！';
	elseif id=2 then
		set class_name='原来是UI的啊';
	else
		set class_name='不用想了，肯定是产品小样';
	end if;
	
	select class_name;
	
end ;

call test5();

```
### 7.1.3、传递参数

```sql
create procedure procedure_name([in/out/inout] 参数名  参数类型)
```

> in： 该参数可以作为输入，也就是需要调用方传入值 , 默认
> out： 该参数作为输出，也就是该参数可以作为返回值
> inout： 既可以作为输入参数，也可以作为输出参数



```sql
-- 定义一个输入参数和一个输出参数
CREATE PROCEDURE test7 (in id int,out class_name varchar(100))
begin
	if id=1 then
		set class_name='哇塞，Java大佬！';
	elseif id=2 then
		set class_name='原来是UI的啊';
	else
		set class_name='不用想了，肯定是产品小样';
	end if;
	
end ;


call test7(1,@class_name);	-- 创建会话变量		

select @class_name;		-- 引用会话变量

```

> @xxx：代表定义一个会话变量，整个会话都可以使用，当会话关闭（连接断开）时销毁
> @@xxx：代表定义一个系统变量，永久生效，除非服务器重启


### 7.1.4、case语句

```sql
-- 传递一个月份，返回所在季节
CREATE PROCEDURE test8 (in month int,out season varchar(10))
begin
	
	case 
		when month >=1 and month<=3 then
			set season='spring';
		when month >=4 and month<=6 then
			set season='summer';
		when month >=7 and month<=9 then
			set season='autumn';
		when month >=10 and month<=12 then
			set season='winter';
	end case;
end ;

call test8(9,@season);			-- 定义会话变量来接收test8存储过程返回的值

select @season;

```
### 7.1.5、while循环

```sql
-- 计算任意数的累加和
CREATE PROCEDURE test10 (in count int)
begin
	declare total int default 0;
	declare i int default 1;
	
	while i<=count do
		set total=total+i;
		set i=i+1;
	end while;
	select total;
end ;

call test10(10);

```

### 7.1.6、repeat循环（while）

```sql
CREATE PROCEDURE test11 (count int)		-- 默认是输入(in)参数
begin
	declare total int default 0;
	repeat 
		set total=total+count;
		set count=count-1;
		until count=0				-- 结束条件,注意不要打分号
	end repeat;
	select total;
end ;

call test11(10);

```
### 7.1.7、loop循环

```sql
CREATE PROCEDURE test12 (count int)		-- 默认是输入(in)参数
begin
	declare total int default 0;	
	sum:loop							-- 定义循环标识
		set total=total+count;
		set count=count-1;
		
		if count < 1 then
			leave sum;					-- 跳出循环
		end if;
	end loop sum;						-- 标识循环结束
	select total;
	
end ;

call test12(10);

```
### 7.1.8、游标

游标是用来存储查询结果集的数据类型，可以帮我们保存多条行记录结果，我们要做的操作就是读取游标中的数据获取每一行的数据。
```sql
CREATE PROCEDURE test13 ()		-- 默认是输入(in)参数
begin
	
	declare id int(11);
	declare `name` varchar(20);
	declare class_id int(11);
	-- 定义游标结束标识符
	declare has_data int default 1;
	
	declare stu_result cursor for select * from student;
	-- 监测游标结束
	declare exit handler for not found set has_data=0;
	
	-- 打开游标
	open stu_result;
	
	repeat 
		fetch stu_result into id,`name`,class_id;
		
		select concat('id: ',id,';name: ',`name`,';class_id',class_id);
		until has_data=0		-- 退出条件,注意不要打分号
	end repeat;
	
	-- 关闭游标
	close stu_result;
	
end ;

call test13();

```
## 7.2、存储函数
### 7.2.1、存储函数和存储过程区别

1. 存储函数的限制比较多，例如不能用临时表、不能执行查询语句、只能用表变量等；而存储过程的限制较少，存储过程的实现功能要复杂些,而函数的实现功能针对性比较强。
2. 返回值不同。存储函数必须有返回值,且仅返回一个结果值；存储过程可以没有返回值,但是能返回结果集(out,inout)。
3. 调用时的不同。存储函数select 存储函数名(变量值)；存储过程通过call语句调用 call 存储过程名。
4. 参数的不同。存储函数的参数类型类似于IN参数，没有类似于OUT和INOUT的参数。存储过程的参数类型有三种，in、out和inout：in： 数据只是从外部传入内部使用(值传递),可以是数值也可以是变量

### 7.2.2、创建方式

```sql
-- 无参函数
create function test1()
returns int			
begin

	declare num int default 0;
	
	set num=10;
	
	return num;
end;

select test1()

-- 带参函数
create function test2(num int)
returns int			
begin
	return num;
end;

select test2(22);
```
# 8、数据库范式
## 8.1、三大范式

1. **第一范式（1NF）**：保证每列的原子性。如果数据库表中的所有字段值都是不可分解的原子值，就说明该数据库满足了第一范式
> 比如用户表里的名字字段，在中国可能直接存完整的姓名就可以了。但在国外姓和名常常需要分开使用，所以需要分成姓和名两个字段存储。所以是否满足原子性需要根据实际需求来确定


2.  **第二范式（2NF）**：满足第一范式前提，不能存在局部依赖。比如有联合主键有两个列，不能存在这样的属性，它只依赖于其中一个列，这就是不符合第二范式 

3.  **第三范式（3NF）**：消除传递依赖，每列都直接依赖于主键
> 假设存在关系模式主键1: 课程编号; 列1: 教师名; 列2: 教师家庭地址。显然满足第一范式和第二范式，但是教师家庭地址传递依赖于教师名，所以不满足第三范式


## 8.2、反范式化

所谓反范式化，是一种对范式化设计的数据库的**性能优化策略**，通过在表中增加冗余或重复的数据来提供数据库的读取性能。没有冗余的数据库不一定是最好的数据库，有时为了提高查询效率，就必须降低范式标准，适当保留冗余数据。具体操作就是在一个表中增加别一个表的冗余字段，减少了两个表查询时的关联，从而提高查询效率。

# 9、数据库备份与恢复
mysqldump命令主要用于数据库的备份

> options：
> -h：mysql服务器的IP
> -P：mysql服务器的端口
> -u：mysql用户名
> -p：mysql密码
> -n（--no-create-db）：不包含创建数据库语句（包含建表语句和数据）
> -t（--no-create-info）：不包含创建表语句（只要插入语句）
> -d（--no-data）：不包含数据
> -B（--database）：导出数据库（也包含建库语句也包含数据）
> -A（--all-databases）：导出所有数据库

```sql
-- 导出指定表（没有指定表则导出所有）
mysqldump -h127.0.0.1 -P3306 -uroot -padmin db02 student > D:/test.sql
-- 导出数据库（比导表增加了 加了create database db_name语句）
mysqldump -uroot -padmin --all-databases > D:/test.sql
-- 恢复（执行sql文件）
source d:/test.sql;
```
# 10、JDBC

是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了操作数据库的所有方法。JDBC是一组接口，没有具体的实现。核心功能也就是实现类由各 **数据库厂商去实现**这些实现类也被成为数据库的驱动。
## 10.1、JDBC常用接口

| **接口或类** | **作用** |
| --- | --- |
| Driver | 驱动接口，定义建立链接的方式 |
| DriverManager | 1. 加载和注册第三方厂商的驱动程序 2. 创建一个数据库的连接对象 |
| Connection | 与数据库的一个连接对象 |
| Statement | SQL语句对象，用于封装SQL语句发送给MySQL服务器 |
| PreparedStatement | 是Statement接口的子接口，功能更加强大 |
| ResultSet | 封装从数据库中查询到的结果集 |

## 10.2、PreparedStatement
PreparedStatement 是 Statement 的子类，也能执行Statement之前的所有操作，其中最主要的功能就是提供了占位符传参处理、预编译等功能；我们实际开发中PreparedStatement会使用的更多
### 10.2.1、sql注入问题
**用户输入的内容作为了SQL语句语法的一部分，改变了原有SQL真正的意义**，以上问题称为SQL注入。要解决SQL注入就不能让用户输入的密码和我们的SQL语句进行简单的字符串拼接
如 用户输入密码：

> abc' or '1'='1

执行的sql为：select * from user where name='admin' and password='abc' or '1'='1';

```java
// PreparedStatement解决sql注入问题：

//创建预编译的SQL语句
ps = conn.prepareStatement("select * from user where name=? and password=?");

//替换占位符
ps.setString(1, name);
ps.setString(2, password);

//执行SQL语句，查询
rs = ps.executeQuery();

```
### 10.2.2、CallableStatement 

allableStatement是PreparedStatement的子类，主要是调用数据库中的存储过程/存储函数。并通过CallableStatement对象可以获取存储过程/存储函数的执行结果；

```java
	// 定义存储过程
    CallableStatement cs = connection.prepareCall("call test3(?,?,?)");

    // 输入参数
    cs.setInt(1,2);

    // 输出参数
    cs.registerOutParameter(2, Types.VARBINARY);
    cs.registerOutParameter(3, Types.VARBINARY);

    // 执行存储过程
    cs.execute();

    // 根据参数名获取值
    String str1 = cs.getString("str1");
    String str2 = cs.getString("str2");

```
### 10.2.3、JDBC事务处理

```java
public class Demo15_事务 {
    @Test
    public void test1() throws Exception{
        //创建连接对象
        Connection conn = null;
        Statement stmt = null;
        try {
            conn = JdbcUtils.getConnection();

            // 设置事务不要自动提交(手动提交,默认情况下,事务是自动提交的)
            conn.setAutoCommit(false);

            //创建语句对象
            stmt = conn.createStatement();

            //a扣钱
            stmt.executeUpdate("update account set money=money-500 where name='a'");

            // b加钱
            stmt.executeUpdate("update account set money=money+500 where name='b'");

            // 提交事务
            conn.commit();

            System.out.println("转账成功");
        } catch (Exception e) {
            try {
                // 回滚事务
                conn.rollback();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            System.out.println("转账失败");
        } finally {
            JdbcUtils.close(conn, stmt);
        }
    }
}

```
