### 1. 数据库常见概念

db 数据库，存储数据的容器

DBMS：数据库管理系统

net start   net stop  服务开启和关闭

### 2.  MYSQL的常见命令

- 进入mysql数据库控制台，
mysql -u root -p
mysql -h localhost -P 3306 -u root -p(cmd启动)
- 使用库，说明还在使用当前这个库
mysql>use 数据库 
- 查看表
show table 
- 查看表
show tables from 数据库  
- 查看当前在那个库
select database() 
- 查看表结构
desc table 
- explain select 语句\G    ///列显示

### 3. mysql的语法规范
1.不区分大小写，但建议关键字大写，表名，列名小写
2.每条命令最好用分号结尾
3.每条命令根据需要，可以进行缩进，或换行
4.注释
​		单行 #  --

​		多行 /*  */
#### 1. 起别名

```
select s_birth AS 'nmb' from student //注意如果别名有特殊符号加''
select s_birth nmb from student
select s_birth  as nmb from student
```

#### 2.去除重复 DISTINCT
```
select DISTINCT s_id from score
```
#### 3.+号的作用
仅仅只是运算符，如果操作数都为数值型，则做加法运算

​		如果有一方为字符串，试图将字符型转换成数值型

​						如果成功，继续做加法运算， 失败的话将字符型转换成零

​		如果一方为null ，则都为null        IFNULL()函数 

CONCAT函数 //拼接两个字符型 CONCAT('a','b','c')
## 二. 查询
### 1.条件查询

1.按条件表达式筛选

​	条件运算符：> < >=  <=   !=  ， <>//不等于两个一样

```
select  s_id from score where s_id <>01  //不等于01
```

2.按逻辑表达式筛选

​	逻辑运算符：&& || ！，，，，，， and    or    not

3.模糊查询  like   between and     in      is null

​		通配符 % 任意多个字符，包含0个             _    任意单个字符

​		转移字符  \  或者标识下   '_$__ %'    ESCAPE  '$'

like

```
select  s_name from student where s_name LIKE '%雷'
select  s_name from student where s_name LIKE '_雷%'// 第二个字符为雷
```

between and 包含临界值       前面小，后面大

```
select  s_score from score where s_score BETWEEN 20 and 80
```

in  判断某字段的值是否属于in列表中的某一项,,,使用in提高语句的简洁的

```
select  s_score from score where s_score in(80,31,50)
```

### 2. 排序查询

语法 order 

​	order by 可以支持单个字段，多个字段，表达式，函数，别名

​	执行顺序再where后

​	select 查询列表  from 表  where   筛选条件   order by  排序列表    asc|desc 增|减

```
select   s_score from score  ORDER BY s_score ASC
select s_id,s_score from score  ORDER BY s_id DESC, s_score ASC
```

### 3. 常见函数 

一.字符函数

​	1. length//获取 参数值的字节个数

​	长度 一个汉字是三个字节  一个字母 一个字节

```
select LENGTH('你麻痹aaa');//12
```

​	2. concat// 拼接字符串

​	3.upper,lower大写，小写

​	4. substr    注意，索引从1开始的

```
select SUBSTR('abcdefg',1,3)  //abc
```

​	5.instr返回子串第一次出现的索引，找不到返回0

​	6.trim

​	7.lpad 用指定的字符实现左填充指定长度

```
select LPAD('df',6,'*')//****df
```

​	8.rpad 用指定的字符实现右填充指定长度

​	9.replace 替换

#### 二.数学函数

1.round 四舍五入

2.ceil向上取整

3.floor向下取整

4.truncate 截断

5.mod取余

#### 三.日期函数

1.now 返回当前系统日期+时间

2.curdate 返回当前系统日期，不包含时间

3.curtime 返回当前时间，不包含日期

4.str_to_date 将字符转换成日期

```
SELECT STR_TO_DATE('2020-9-26','%Y-%c-%d')
```

5.data_format 将日期转换成字符

```
SELECT DATE_FORMAT(NOW(),'%y年%m月%d日')
```

6.datediff相差天数

#### 流程控制函数

1.if函数

2.case函数 跟switch case效果

#### 分组函数

功能：用作统计使用，又称为聚合函数或统计函数

分类：

​	sum 求和，avg 平均值，max 最大值，min最小值， count计算个数

参数支持类型

​	特点 

​		sum，avg一般用于处理数值型

​		max，min，count，可以处理任何类型

​		都忽略null值

​		可以和distinct搭配实现去重的运算

​		和分组函数一通查询的字段要求是group by后的字段

### 4. 分组查询

group by 分组的列表

1.分组查询中的筛选条件分为两类

​								数据源								位置												关键字

​	分组前筛选 		原始表								group by句子前面						where

​	分组后筛选		分组后的结果集					group by句子后面					having

​	分组函数做条件肯定是放再having子句

​	能用分组前筛选的，就优选考虑使用分组前筛选

2.group by 句支持单个字段分组，多个字段分组（用逗号隔开）

3.也可以添加排序，排序放在最后



按表达式或函数分组

按多个字段分组，就是将多个字段放在group up后

查询各科成绩的最大值

```
SELECT MAX(s_score),c_id FROM score GROUP BY c_id
```

查询各科成绩最大值大于85的

```
SELECT MAX(s_score),c_id FROM score 
GROUP BY c_id
HAVING MAX(s_score)>85
```

```
#查询各科的平均成绩
SELECT AVG(s_score) as a,s_id FROM score 
GROUP BY s_id

SELECT AVG(s_score) a,s_id FROM score 
GROUP BY s_id
HAVING a>=60

#查询平均成绩大于等于60分的同学编号和学生姓名和平均成绩
SELECT AVG(s_score),s.s_id,s.s_name from student s JOIN score c on s.s_id =c.s_id
GROUP BY s.s_id
HAVING AVG(s_score)>60
```



### 5. 连接查询

多个表之间

内连接    innter jion

​	*等值连接**

​		连接条件=

​		多表等值连接的结果为多表的交集部分

​		n个表，至少有n-1个连接条件

​		一般为表其别名，

​	非等值连接

​		连接条件不是=，可以是between and 大于，小于 等

​	自连接 

​		就是把一张表当初两张表使用，甚至更多，连接的条件有可能

​		jion

```
查询男朋友，不在男生表中的女生名
//girl为主表
SELECT b.name,bo.* from girl b LEFT OUTER JOIN boy bo
ON b.boy_id = bo.id
WHERE bo.id is NULL
```

外连接

​	应用场景：用于查询一个表中有的，另外一个表没有的记录

​		查询的是什么，那个就是主表

​	特点

​	1.外连接的查询结果为主表中的所有记录

​					如果从表中有和它匹配的，则显示匹配的值

​					如果从表中没有和他匹配的，则显示null

​					外连接查询结果=内链接结果+主表中有而从表没有的记录 

​	2. 左外连接 left jion左边的是主表 left outer jion

​		右外连接 right  jion右边的是主表

​		更改的时候就只要把表位置改一下就行了

​	3.左外和右外交换两个表的顺序，可以实现同样的效果

​	4.全外连接 U

​	5.交叉连接 cross jion 相乘

### 6. 子查询

​	按子查询出现的位置

​		select后面

​				仅仅支持标量子查询 

​		from后面

​				支持表子查询

​		where或having后面*

​				标量子查询* 单行

​				行子查询*多行

​				列子查询

​		exists后面（相关子查询）

​			表子查询

​	按结果集的行列数不同

​		标量子查（结果集只有一行一列）

​		列子查询（结果集只有一列多行）

​		行子查询（结果集有一行多列）

​		表子查询（结果及一般为多行多列）

###### 一.	where或having后面*

特点

​	1.子查询放在小括号内

​	2.子查询一般放在条件的右侧

​	3.标量子查询，一般搭配着单行操作符使用

>  <  >=    <=    <>

​	列子查询 一般搭配着多行操作符使用

​		`in/not in 等于列表中的任意一个` 

​		`all   any/some`	

4.子查询的执行优于主查询执行

一.标量子查询* 单行

案例1.谁的工资比a高？

```
//1.查询a的工资
select salary from employees 
where name = 'a'
//2.查询员工的信息，满足salary大于1结果
select * from employees
where salary>
(select salary from employees where name= 'a')
```

案例2.查询公司工资最少的员工的 name，job_id,salary

```
1.查询公司的最低工资
select min(saraly) from employees
查询 name，job_id, saraly 要求 salary= 1.
select name,job_id,salary from employees
where saraly = (
		select min(saraly) from employees
)
```

案例四

```
查询最低工资大于50号部门最低工资的id和其最低工资
1.查询50号部门最低工资
select min(salary) from employees where department_id =50
2.查询每个部门的最低工资
select min(salary),department_id
from employees
group by department_id
3.在2基础上筛选 满足 min(salary)>1
select min(salary),department_id
from employees
group by department_id
having salary>(
select min(salary) from employees where department_id =50
)

```

二.列子查询（多行子查询）

案例1：返回location_id是1400或1700的部门中的所有员工姓名

```sql
1.查询location_id是1400或1700的部门编号
select department_id from departments
where location_id in(1400,1700)
2.查询员工姓名，要求部门号是1。列表中的某一个字段
select name from where department_id in(
	select department_id from departments
	where location_id in(1400,1700)
)
```

###### 四.exists后面（相关子查询）

/*

了解   语法：exists（完整的查询语句）结果：1或0

*/

案例:查询各部门中工资比本部门平均工资高的员工的员工号，姓名和工资

```
1.查询各个部门的平均工资
select avg(salary),department_id from employees
group by department_id
2:连接1结果集和employees表，进行筛选
select employee_id,name,salary from employees e
inner jion(
		select avg(salary) ag,department_id
		from employees
		group by department_id
)agl
on e.department_id  = agl.department_id
where salary>agl.ag
```

案例：查询和姓名中包含字母u的员工在相同部门的员工的员工号和姓名

```
1：查询姓名中包含字母u的员工的部门
select distinct department_id from employees
where name like '%u%'
2.查询部门号=1中的任意一个的员工号和姓名
select name,employee_id from employees
where department_id in(
	select distinct department_id from employees
	where name like '%u%'
)
```

### 7.分页查询

limit   【offset】起始点，size

特点 ：1.limit语句放在查询语句的最后

​			  2.公式 要显示的页数page，每页的条目数size

​				limit  (page-1)*size,size

案例1：查询前五条员工信息

```
select * from employees limit 0,5;
```

案例2：查询第11条——第25条

```
select * from employees limit 10,15;
```

案例3：有奖金的员工信息， 并且工资较高的钱10名显示出来

```
select * from employees where commission is not null
order by salary desc
limit 10;
```

```
测试
已知表 student       grade
id 学号							 id 年纪编号
name 姓名						 gradeName 年纪名称
email 邮箱
gradeId 年级编号
sex 性别
age 年龄
1.查询 所有学员的邮箱的用户名（邮箱中@前面的字符）
select substr(email,instr(email,'@')-1) 用户名 from student
2.查询年龄大于18岁的所有学生的姓名和年纪名称
select name,gradeName from student s inner jion grade g
on s.gradeId = g.id
where age>18
3.查询哪个年级的最小年龄大于20岁
	*每个年龄的最小年龄
select min(age),gradeId from student
group by gradeId
  *
select min(age),gradeId from student
group by gradeId
having min(age)>20



select 查询列表 from 表
连接类型 jion 表2
on 连接条件
where 筛选条件
group by 分组列表
having 分组后的筛选
limit 偏移，条目数
```

### 8.联合查询

union 联合 合并：将多条查询语句的结果合并成一个结果

语法：

​	查询语句1

​	union

​	查询语句2
## 三. sql优化的几种方式

为什么要对SQL进行优化

**SQL优化的一些方法**

## IV. 索引

### 1. 那些情况需要创建索引

1. 主键自动建立唯一索引
2. 频繁作为查询条件的字段应该创建索引
3. 查询中与其他表关联的字段，外键关系建立索引
4. 频繁更新的字段不适合创建索引，，因为每次更新不单单是更新了记录还会更新索引
5. where条件里用不到的字段不创建索引
6. 单键/组合索引的选择问题，who（高并发下倾向创建组合索引）
7. 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
8. 查询中统计或者分组字段

### 2. 索引能干嘛

1. 表的读取顺序
2. 数据读取操作的操作类型
3. 哪些索引可以使用
4. 哪些索引被实际使用
5. 表之间的引用
6. 每张表有多少行被优化器查询

### 3. 索引种类

- 普通索引：仅加速查询
- 唯一索引：加速查询 + 列值唯一（可以有null）
- 主键索引：加速查询 + 列值唯一（不可以有null）+ 表中只有一个
- 组合索引：多列值组成一个索引，专门用于组合搜索，其效率大于索引合并
- 全文索引：对文本的内容进行分词，进行搜索

### 4. 操作索引

```
--创建普通索引
CREATE INDEX index_name ON table_name(col_name);
--创建唯一索引CREATE UNIQUE INDEX index_name ON table_name(col_name);
--创建普通组合索引CREATE INDEX index_name ON table_name(col_name_1,col_name_2);
--创建唯一组合索引CREATE UNIQUE INDEX index_name ON table_name(col_name_1,col_name_2);

通过修改表结果创建索引
ALTER TABLE table_name ADD INDEX index_name(col_name);

删除索引
--直接删除索引DROP INDEX index_name ON table_name;
--修改表结构删除索引ALTER TABLE table_name DROP INDEX index_name;

```



```
- 查看表结构
    desc table_name;
 - 查看生成表的SQL
    show create table table_name;
 - 查看索引
    show index from  table_name;
 - 查看执行时间
    set profiling = 1;
    SQL...
    show profiles;

```

### 5. 性能分析

explain+sql语句 

表的读取顺序

###### id解释

select查询的序列号，表示查询中执行select子句或操作表的顺序

三种情况

​	1.id相同，执行顺序由上至下

​	2.如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

​	3.id相同不同，同时存在    衍生 = DERIVED

###### select_type

种类

1.SIMPLE---2.PRIMARY---3.SUBQUERY---4.DERIVED---5.UNION---6.UNION RESULT

查询的类型，主要是用于区别

普通查询，联合查询，子查询等的复杂查询

simple--简单的select查询，查询中不包含子查询或者union

primary--查询中若包含任何复杂的子部分，最外层查询则被标记为

subquery--在select或where列表中包含了子查询

derived--在from列表中包含的子查询被标记为derived（衍生），mysql会递归执行这些子查询，把结果放在临时表里

union--若第二个select出现在union之后，则被标记为union，，若union包含在from子句的子查询中，外层select表被标记为：derived

UNION RESULT--从union表获取结果的select

###### table

###### type--访问类型排列

显示查询使用了何种类型

从最好到最差依次是：system>const>eq_ref>ref>range>index>ALL

要保证查询至少达到range级别，最好能达到ref

**const-**-表示通过索引一次就找到了，const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快，如将主键置于where列表中，mysql就能将查询转换为一个常量。

**eq_ref**--唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一	索引扫描。

**ref**--非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。 

**range**--只检索给定范围的行，使用一个索引来选择行。key列显示使用了那个索引，一般就是在你的where语句中出现了between，<,> ,in 等的查询，这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束另一点，不用扫描全部索引。

**index** --虽然都是读全表的，但index是从索引中读取的

**ALL** full table scan 将遍历全表以找到匹配的行

###### possible_keys

显示**可能**应用在这张表中的索引，一个或多个。查询设计到的字段上若存在索引，则该索引将被列出。但不一定被查询实际使用

###### key

**实际**使用的索引，如果为null，则没有使用索引，

查询中若使用了**覆盖索引**这边指向using index那边对应，则该索引仅出现在key列表中

###### key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好。key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的

###### ref

显示索引的哪一列被使用，如果可能的话，是一个常数。那些列或常量被用于查找索引列上的值

###### rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

###### extra

包含不适合在其他列中显示但十分重要的额外信息

1. using filesort---说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读写，mysql中无法利用索引完成的							排序操作称为“文件排序”（性能很差，不好）

2. using temporary --使用临时表保存中间结果，mysql在对查询结果排序时使用临时表。常见于排序order by 和分组查询group by （出现这个的时候说明性能很差，不好）

3. using index--（性能不错，好）表示相应的select操作中使用了**覆盖索引**（coving index），避免访问了表的数据行，效率不错。

   ​						如果同时出现了using where，表明索引被用来执行索引键值的查找

   ​				  	如果没有同时出现using where，表面索引用来读取数据而非执行查找动作

4. using where 表示使用了where过滤

5. using jion buffer 使用了连接缓存

6. impossible where---where子句的值总是false，不能用来获取任何元组（比如一个男，是男的还是女的）

##### 覆盖索引Coving Index 

`理解方式，就是select的数据列只用从索引中就能够取得，不必读取数据行，mysql可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件，换句话说**查询列要被所建的索引覆盖**，建的是啥用的就是索引里面的`

注意：

​		如果要使用覆盖索引，一定要注意select列表中只取需要的列，不可select *，因为如果将所有字段一起做索引会导致索引文件过		大，查询性能下降

### 6. 索引优化

两个表操作

1.由于左连接特性决定的，left jion 条件用于确定如何从右表搜索行，左边一定都有，

所以右边是我们的关键点，一点需要建立索引

或者在左右连接表的位置更改了也行

2.由于右连接特性决定的，right jion 左边是我们的关键点，一点需要建立索引

![image-20200602164840869](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200602164840869.png)

![image-20200602212547876](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200602212547876.png)

无建索引

![image-20200603000815380](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200603000815380.png)

左连接有建索引

![image-20200603000330662](C:\Users\drew\AppData\Roaming\Typora\typora-user-images\image-20200603000330662.png)

结论

join语句优化

尽可能减少join语句中的nestedloop的循环总次数，“永远用小结果集驱动大的结果集”

优先优化NestedLoop内层循环

保证join语句中被驱动表上join条件字段已经被索引

当无法保证被驱动表的join条件字段被索引且内存资源充足的前提下，不要太吝啬joinbuffer的设置

#### 索引失效（应该避免）

1. 全值匹配我最爱

   1. 要遵从最左前缀法则，
   2. 比如 alter table `class` add index('a','b','c')
   3. select * from class where a="sdf";(这边的a="sdf"只要不写出来，索引就失效了)
   4. select * from class where a="sdf" and C= “dsf”不行
   5. 没有火车头，索引就失效了，中间不能断

2. 最佳最前缀法则，

   1. 指的是查询从索引的最左前列开始并且**不能跳过索引中的中间列**中间兄弟不能断

3. 不在索引列上做任何操作（计算，函数，自动or手动类型转换），会导致索引失效而转向全		表扫描

   ![image-20200603005252487](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200603005252487.png)

4. 存储引擎不能使用索引中范围条件右边的列

5. 尽量使用覆盖索引（只访问索引的查询（索引列和查询列一致）)，减少select *

6. mysql在使用不等的时候无法使用索引会导致全表扫描

7. is null is  not null 也无法使用索引

8. like以通配符开头（%abvc..）mysql索引失效会变成全表扫描的操作

9. 字符串不加单引号失效

10. 少用or，用它来连接时会索引失效

### 7. 查询截取分析

#####  查询优化

###### 	优化原则：小表驱动大表，即小的数据集驱动大的数据集

```
当b表的数据集必须小于a表的数据集时，用in优于exists
select * from A where id in (select id from b)
等价于
for select id from b
for select * from a where a.id = b.id
当a表的数据集小于b表的数据集时，用exists优于in
select * from a where exists(select 1 from b where 
b.id = a.id)
等价于
for select * from a
for select * from b where b.id = a.id

```

exists可以理解为：

​	将主查询的数据，放到子查询中做条件验证，根据验证结果（true或false）来决定主查询的数据结果是否得以保留

​	提示：exists(subquery)只返回true或false，因此子查询中的select*也可以是select 1或		select 'x' 

###### 	order by关键字优化

 尽量使用index方式排序，避免使用年FileSort方式排序，尽可能在索引列上完成排序操作，遵守索引建的最佳左前缀

优化策略--增大sort_buffer_size参数的设置

​					增大max_length_for_sort_data参数的设置

order by 能使用索引最左前缀

- order by a
- order by a,b
- order by a,bc
- order by a desc ,b desc,c desc

如果where 使用索引的最左前缀定义为常量，则order by能使用索引

​	where a = const  order by b,c

​	where a = const and b>const order by b,c

不能使用索引进行排序

order by a ASC, b DESC/* 排序不一致*/

where g = const order by b,c /* 丢失a索引*/

where  a in(...) order by b,c 

 	

###### group by关键字优化

 尽量使用index方式排序，避免使用年FileSort方式排序，尽可能在索引列上完成排序操作，遵守索引建的最佳左前缀

优化策略--增大sort_buffer_size参数的设置

​					增大max_length_for_sort_data参数的设置

​					where高于having，能卸载where限定的条件就不要去having限定了。

### 8. 慢查询日志

1. MySQL的慢查询日志是mysql提供的一种日志记录，它用来记录在mysql中响应时间超过阈		值的语句，具体指运行时间超过					long_query_time值的sql，则会被记录到慢查询日志中。
2. 具体指运行时间超过long_query_time值的sql，则会被记录到慢查询日志中。			long_query_time的默认值为10，意思是运行10秒以	上的语句
3. 由他来查看哪些sql超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢sql

查看是否开启慢查询

​		默认-show vaariables like '%show_query_log%';

​		开启-set global slow_query_log=3; 设置为3秒//重新进去了就可以刷新

### 9. 批量数据脚本

### show profile

show variables like 'profiling'

set profiling = on;

### 10. 全局查询日志

#### 分析

1. 慢查询的开启并捕获
2. explain+慢sql分析
3. show profile查询sql在mysql服务器里面的执行细节和生命周期情况
4. sql数据库服务器的参数调优



## mysql锁机制



## 主从复制

































































