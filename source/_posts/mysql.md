---
title: mysql
date: 2022/08/12
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

## 1.2、事务的特性
如果一个业务操作中多次访问了数据库，必须保证每条SQL语句都执行成功。如果其中有一条执行失败，那么所有已经执行过的代码必须回滚（撤销）。回到没有执行前的状态。称为事务。简单来说就是要么所有的SQL语句全部执行成功，要么全部失败

| **事务特性** | **含义** |
| --- | --- |
|  原子性（Atomicity） | 一个事务内的所有操作要么都执行，要么都执行失败 |
|  一致性（Consistency） | 事务执行前与执行后，数据库中数据应该保持相同的状态。如：转账前总金额与转账后总金额相同。 |
|  隔离性（Isolation） | 指的是一个事务在最终提交前，对其它事务是不可见的 |
|  持久性（Durability） |  如果事务执行成功，对数据库的操作是持久的。 |

## 1.3、并发访问下事务产生的问题
当同时有多个用户在访问同一张表中的记录，每个用户在访问的时候都是一个单独的事务。事务在操作时的理想状态是：事务之间不应该相互影响，实际应用的时候会引发下面三种问题：

1. 脏读： 一个事务（用户）读取到了另一个事务没有提交的数据
2. 不可重复读：一个事务多次读取同一条记录，出现读取数据不一致的情况。一般因为另一个事务更新了这条记录而引发的。
3. 幻读：在一次事务中，多次读取到的条数不一致

## 1.4、事务的隔离级别
为了尽量避免这些问题的发生。通过数据库本身的功能去避免，设置不同的隔离级别

| **级别** | **名字** | **隔离级别** | **脏读** | **不可重复读** | **幻读** | **数据库默认隔离级别** | 解释 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 读未提交 | read uncommitted | 是 | 是 | 是 |  | 事务A可以读取到事务B未提交的数据 |
| 2 | 读已提交 | read committed | 否 | 是 | 是 | Oracle和SQL Server | 事务A只能读取其它事务已提交的数据（避免了脏读） |
| 3 | 可重复读 | repeatable read | 否 | 否 | 是 | MySQL | 保证在同一个事务中多次读取同样数据的结果是一样的 |
| 4 | 串行化 | serializable | 否 | 否 | 否 |  | 事务串行化顺序执行 |

> 隔离级别越高，安全性就越高，性能越低


## 1.5、mysql索引分类

1. 按数据结构分类可分为：B+tree索引、Hash索引、Full-text索引。 
2. 按物理存储分类可分为：聚簇索引（innodb）、非聚簇索引（myisam）。 

> 聚簇索引：顾名思义，索引和数据是放在一块的。其叶子节点中存放的就是整张表的行记录数据。我们称之为聚集索引（如果创建了主键，这个主键索引就是聚簇索引，如果没有，innodb建立一个隐藏的row-id作为聚集索引）
> 非聚集索引：一般也称为二级索引或者辅助索，非聚集索引的叶子节点不存储表中的数据，而是存储索引值和数据对应主键，想要查找数据我们还需要根据主键再去聚集索引中进行查找，这个再根据聚集索引查找数据的过程，我们称为回表

3. 按字段特性分类可分为：主键索引、唯一索引、普通索引、前缀索引。

> **主键索引**：建立在主键上的索引被称为主键索引，一张数据表只能有一个主键索引，索引列值不允许有空值
> **唯一索引**：建立在UNIQUE字段上的索引被称为唯一索引，一张表可以有多个唯一索引，索引列值允许为空
> **普通索引**：建立在普通字段上的索引被称为普通索引
> **前缀索引**：文本的前几个字符建立索引（具体是几个字符在建立索引时指定），这样建立起来的索引更小，所以查询更快
> CREATEINDEX index_name ONtable_name (column_name(length));

4.  按字段个数分类可分为：单列索引、联合索引（复合索引、组合索引）。

> 单列索引：建立在单个列上的索引被称为单列索引
> 联合索引：建立在多个列上的索引被称为联合索引，又叫复合索引、组合索引
> （复合索引遵守“最左前缀”原则，即在查询条件中从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分）


## 1.6、索引设计原则
**适合建索引的场景**
1. 频繁作为where条件语句查询的字段
2. 关联字段需要建立索引，例如外键字段，student表中的classid,   classes表中的schoolid 等
3. 分组排序字段可以建立索引
4. 统计字段可以建立索引（如.count(),max()）

**不适合建索引的场景**
1. 频繁更新的字段不适合建立索引（需要重建索引）
2. where条件中用不到的字段不适合建立索引
3. 表数据可以确定比较少的不需要建索引
4. 数据重复且发布比较均匀的的字段不适合建索引（唯一性太差的字段不适合建立索引），例如性别，真假值

## 1.7、索引失效的情况

1.  or连接的条件不是每一个列都有索引，这时有索引的列也会失效 
2.  复合索引遵守“最左前缀”原则，在通过联合索引检索数据时，从索引中最左边的列开始，一直向右匹配，如果遇到范围查询(>、<、between、like等)，就停止后边的匹配
```
eg：建立复合索引index:(a,b,c) --- 实际上已经建立了三个联合索引(a)、(a,b)、(a,b,c)
select * from table where a = '1'  //走索引
select * from table where c = '1'  //不走索引
where a like 'xxx%' and b=yyy and c=zzz 
```

3. like查询是以%开头（以%结尾，索引可以使用）

> B+树，索引是有序排列的。索引的排列顺序是根据比较字符串的首字母排序的，如果首字母相同，就根据比较第二个字母进行排序，以此类推。以%开头，首字母不确定

4. 存在索引列的数据类型隐形转换，则用不上索引（比如列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引）
5. where 子句里对索引列上有数学运算，用不上索引
6. where 子句里对有索引列使用函数，用不上索引
7. 如果mysql估计使用全表扫描要比使用索引快,则不使用索引（比如数据量极少的表）

## 1.8、InnoDB和MyISAM
innoDB：innodb是mysql默认存储引擎，支持事务，行锁和外键等
MyISAM：myisam是mysql5.1版本前默认存储引擎，不支持事务，外键。默认锁的粒度为表锁

|  | InnoDB | MyISAM |
| --- | --- | --- |
| 外键 | 支持 | 不支持 |
| 事务 | 支持 | 不支持 |
| 锁 | 支持表锁和行锁 | 支持表锁 |
| 表结构 | 数据和索引集中存储 | 数据和索引分开存储 |
| 索引 | 聚簇索引 | 非聚簇索引 |

## 1.9、数据库三大范式

1. **第一范式（1NF）**：保证每列的原子性。如果数据库表中的所有字段值都是不可分解的原子值，就说明该数据库满足了第一范式

> 比如用户表里的名字字段，在中国可能直接存完整的姓名就可以了。但在国外姓和名常常需要分开使用，所以需要分成姓和名两个字段存储。所以是否满足原子性需要根据实际需求来确定

2.  **第二范式（2NF）**：满足第一范式前提，不能存在局部依赖。比如有联合主键有两个列，不能存在这样的属性，它只依赖于其中一个列，这就是不符合第二范式 

3.  **第三范式（3NF）**：消除传递依赖，每列都直接依赖于主键

> 假设存在关系模式主键1: 课程编号; 列1: 教师名; 列2: 教师家庭地址。显然满足第一范式和第二范式，但是教师家庭地址传递依赖于教师名，所以不满足第三范式


> 所谓反范式化，是一种对范式化设计的数据库的**性能优化策略**，通过在表中增加冗余或重复的数据来提供数据库的读取性能。没有冗余的数据库不一定是最好的数据库，有时为了提高查询效率，就必须降低范式标准，适当保留冗余数据。具体操作就是在一个表中增加别一个表的冗余字段，减少了两个表查询时的关联，从而提高查询效率


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

callableStatement是PreparedStatement的子类，主要是调用数据库中的存储过程/存储函数。并通过CallableStatement对象可以获取存储过程/存储函数的执行结果；

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

# 11、原理相关知识
## 11.1、 数据库为什么用B+树索引（平衡 b b+区别）

二叉查找树：二叉查找树的特点就是任何节点的左子节点的键值都小于当前节点的键值，右子节点的键值都大于当前节点的键值。顶端的节点我们称为根节点，没有子节点的节点我们称之为叶节点

平衡二叉树：平衡二叉树又称 AVL 树，在满足二叉查找树特性的基础上，要求每个节点的左右子树的高度差不能超过 1

> 平衡二叉树相比于二叉查找树来说，查找效率更稳定，总体的查找速度也更快

B树：从磁盘中读取数据时，都是按照磁盘块来读取的，并不是一条一条的读。如果我们能把尽量多的数据放进磁盘块中，那一次磁盘读取操作就会读取更多数据。平衡二叉树可是每个节点只存储一个键值和数据的，如果我们要存储海量的数据，二叉树的节点将会非常多，高度也会极其高。
图中的每个节点称为页，页就是我们上面说的磁盘块，在 MySQL 中数据读取的基本单位都是页，所以我们这里叫做页更符合 MySQL 中索引的底层数据结构

![image (29).png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311101517844.png)

B+树：B+ 树是对 B 树的进一步优化.
①B+ 树非叶子节点上是不存储数据的，仅存储键值，而 B 树节点中不仅存储键值，也会存储数据。如果不存储数据，那么就会存储更多的键值，相应的树的阶数（节点的子节点树）就会更大，树就会更矮更胖，如此一来我们查找数据进行磁盘的 IO 次数又会再次减少，数据查询的效率也会更快。
②因为 B+ 树索引的所有数据均存储在叶子节点，而且数据是按照顺序排列的，通过双向链表关联。那么 B+ 树使得范围查找，排序查找，分组查找以及去重查找变得异常简单。而 B 树因为数据分散在各个节点，要实现这一点是很不容易的。

![image (30).png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311101518699.png)


## 11.2、索引下推

索引下推是索引下推是 MySQL 5.6 及以上版本上推出的，用于对查询进行优化。索引下推是把本应该在服务层进行筛选的条件，下推到存储引擎层来进行筛选判断，这样能有效减少回表。
eg: 首先使用联合索引（name，age），现在有这样一个查询语句：

> select *  from t_user where name like 'L' and age = 17;

这条语句从最左匹配原则上来说是不符合的，只有name用的索引，但是age并没有用到。
不用索引下推的执行过程：

> 第一步：存储引擎利用索引找出name带'L'的数据行：LiLei、Lili、Lisa、Lucy 这四条索引数据
> 第二步：存储引擎再根据这四条索引数据中的 id 值，逐一进行回表扫描，从聚簇索引中找到相应的行数据，将找到的行数据返回给服务层。
> 第三步：在server层判断age = 17,进行筛选，最终只留下 Lucy 用户的数据信息。

使用索引下推的执行过程：

> 第一步：存储引擎利用索引找出name带'L'的数据行：LiLei、Lili、Lisa、Lucy 这四条索引数据
> 第二步：存储引擎根据 age = 17 这个条件，对四条索引数据进行判断筛选，最终只留下 Lucy 用户的数据信息。（注意：这一步不是直接进行回表操作，而是根据 age = 17 这个条件，对四条索引数据进行判断筛选）
> 第三步：将符合条件的索引对应的 id 进行回表扫描，最终将找到的行数据返回给 server 层。

比较二者的第二步我们发现，索引下推的方式极大的减少了回表次数。

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311101520226.png)


> Extra中 Using index condition代表使用了索引下推

## 11.3、mysql结构

![image (31).png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311101520772.png)

MySQL整体的逻辑结构可以分为4层,客户层、服务层、存储引擎层、数据层
**客户层**
客户层:进行相关的连接处理、权限控制、安全处理等操作
**服务层**
服务层负责与客户层进行连接处理、处理以及执行SQL语句等,主要包含连接器、查询缓存、优化器、执行器、存储引擎。触发器、视图等也在这一层
**存储引擎层**
存储引擎层负责对数据的存储和提取，常见的存储引擎有InnoDB、MyISAM、Memory等，在MySQL5.5之后，MySQL默认的存储引擎就是InnoDB,InnoDB默认使用的索引结构就是B+树,上面的服务层就是通过API接口与存储引擎层进行交互的
**数据层**
数据层系主要包括MySQL中存储数据的底层文件，与上层的存储引擎进行交互，是文件的物理存储层。其存储的文件主要有：日志文件、数据文件、配置文件、MySQL的进行pid文件和socket文件等。

## 11.4、数据库查询流程

1. 当客户端的查询语句为select查询语句的时候，如若再查询缓存里面已经查询到了结果，就会直接把查询结果返回给客户端
2. 在查询缓存并没有查询到结果之后，就会走到解析器，在解析器这儿，解析sql，判断是否有语法错误
3. 语法没有问题，走到执行器，执行器先会预处理（检测用户对表的权限，和相应字段有没有），优化器（对sql执行顺序，使用索引等进行优化）
4. 执行器使用引擎提供的接口与存储引擎层进行交互，执行SQL语句，并将结果返回个客户端

## 11.5、sql优化的方案有哪些？

1. 优化表结构

（1）对经常查询的列添加索引，加快查询速度。
（2）尽量避免使用大字段（如 TEXT、BLOB），因为这些字段的读写速度较慢，会影响性能。

2. 优化查询语句

（1）减少查询返回的数据量，避免不必要的 JOIN、子查询等操作。
（2）使用优化的查询方式，例如使用 EXISTS 替代 IN 或 NOT IN。
（3）避免出现索引失效的情况，如WHERE 子句中使用函数或运算符

3. explain 等工具来分析 SQL 查询语句，找出执行计划，确定是否存在性能瓶颈

## 11.6、mysql的explain的详解
EXPLAIN 命令用于SQL语句的**查询执行计划**
```java
EXPLAIN select * from person where dept_id =(select did from dept where dname ='python');
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2996398/1679319789341-8cecd090-7731-4223-9b27-ab9a3b605275.png#averageHue=%23f9f7f5&clientId=u65a23804-8145-4&from=paste&height=82&id=u45062470&originHeight=82&originWidth=816&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7645&status=done&style=none&taskId=u746f8984-6a64-4768-842a-e3ec65c59d5&title=&width=816)

1. id（查询序列号）：从 2 个表中查询，对应输出 2 行，每行对应一个表， id 列表示执行顺序，id 越大，越先执行，id 相同时，由上至下执行
2. select_type（查询类型）：最常见的值包括SIMPLE、PRIMARY、DERIVED 和UNION

> simple：简单查询，没有union和子查询
> primary ：最外层查询 (在存在子查询的语句中，最外面的select查询就是primary)
> derived  ：子查询(在FROM列表中包含的子查询)  EXPLAIN SELECT *FROM (SELECT* FROM person LIMIT 5) AS s
> subquery ：映射为子查询(在SELECT或WHERE列表中包含了子查询)

3. table： 输出的行所用的表
4. type：连接类型，访问类型，表示MySQL在访问表时所采取的方式

性能：性能： null > system/const > eq_ref > ref > ref_or_null   >  range > index >  all 

> null：优化过程中就已得到结果，不用再访问表或索引
> const：在整个查询过程中这个表最多只会有一条匹配的行，比如主键 id=1 就肯定只有一行
> eq_ref：使用有唯一性 索引查找（主键或唯一性索引）
> ref：非唯一性索引访问（select * from user where username = '张三';）
> ref_or_null：该联接类型如同ref类似,结果包含空行
> (上面这五种情况都是很理想的索引使用情况)
> range：索引范围扫描，常见于　<,<=,>,>=,between,in等操作符
> index：索引全扫描，MySQL遍历整个索引来查询匹配的行：（select username from user）
> all：全表扫描

5. possible_keys：可能使用的索引
6. key：实际使用的索引，表示MySQL在执行查询时所使用的索引
7. rows：扫描行数，表示MySQL在执行查询时所扫描的行数。
8. extra：重要的额外信息（[https://blog.csdn.net/li1325169021/article/details/113925826](https://blog.csdn.net/li1325169021/article/details/113925826)）

> （1）Using filesort：排序时没有按照建立复合索引字段的顺序进行，因此产生了外部的索引排序。效率低
> （2）Using temporary：使了用临时表保存中间结果,MySQL在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by
> （3）Using index：select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错
> （4）Using join buffer：表明使用了连接缓存，给驱动表建立索引可解决此问题


## 11.7、MVCC
多版本并发控制，数据库隔离级别读已提交、可重复读 都是基于MVCC实现的，相对于加锁简单粗暴的方式，它用更好的方式去处理读写冲突，能有效提高数据库并发性能。【注意】：只有快照读才会使用MVCC，当前读使用行锁+间隙锁（临键锁Next-Key Locks）实现
它的实现依赖于三个概念：版本链和快照读和ReadView

1. 版本链：多个事务并行操作某一行数据时，不同事务对该行数据的修改会产生多个版本，然后通过回滚指针（roll_pointer），连成一个链表，这个链表就称为版本链(最新记录+undo-log)

> undo log，**回滚日志**，用于记录数据被修改前的信息。在表记录修改之前，会先把数据拷贝到undo log里，如果事务回滚，即可以通过undo log来还原数据. 可以这样认为，当delete一条记录时，undo log 中会记录一条对应的insert记录

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311101605938.png)


2. 快照读：读取的是记录数据的可见版本，不加锁，普通的select语句都是快照读

> 当前读：读取的是记录数据的最新版本，显式加锁的都是当前读。如update，delete，select..for update
> 【注意】：只有快照读才会使用MVCC，当前读使用行锁+间隙锁（临键锁Next-Key Locks）实现

3. RedaView：读视图，其实就是一个数据结构，包含四个属性：当前活跃事务编号集合，最小活跃事务编号，预分配事务编号（当前最大事务编号+1），创建者事务编号

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311101605005.png)


> 读已提交：在每次执行快照读的时候生成ReadView
> 可重复读：（同一事务）只在第一次使用快照读的时候生成ReadView，后续复用（解决不可重复读）,（但两次快照读之间存在当前读，也会重新生成，所以存在幻读的问题）

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202311101606136.png)


> 流程：
> 1. 在执行查询操作时生成一个ReadView（注意在读已提交下，每次快照读都生成一个ReadView）
> 2. 遍历版本链，判断是否符合ReadView规则（这个过程其实就是找它最近一次事务提交的数据版本）
> 3. 返回符合规则的数据


## 11.8、行级锁
innoDB通过给索引记录加锁的方式实现行级锁，具体来说实现了三种行锁算法：
**记录锁**：锁定单个行记录的锁（RC,RR都支持）
**间隙锁**：锁定索引记录的间隙，确保索引记录的间隙不变（RR支持）
**next-key锁**：记录锁和间隙锁的组合，同时锁住数据和数据前后范围（RR支持）

> 在RR隔离级别，InnoDB对于记录加锁都是线采用next-key锁，sql中含有唯一索引时，会采用记录锁，仅锁住索引本身而非范围

各种操作加锁的特点：
（1）select...from：采用MVCC而非加锁
（2）select...from for update（在事务完成之前其他事务无法修改它，确保读到的是最新的）：采用next-key，如果有唯一索引，采用记录锁
（3）update,insert,delete同上