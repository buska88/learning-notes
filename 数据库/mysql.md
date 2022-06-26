# mysql

## mysql语法

https://www.begtut.com/mysql/mysql-tutorial.html

这篇教程里写的比较详细。

## 窗口函数

https://zhuanlan.zhihu.com/p/92654574

https://segmentfault.com/a/1190000040088969

http://www.vldb.org/pvldb/vol8/p1058-leis.pdf

使用场景：排名+topn

```text
window_function (expression) OVER (
   [ PARTITION BY part_list ]#按照part_list分组
   [ ORDER BY order_list ]#按照order_list排序
   [ { ROWS | RANGE } BETWEEN frame_start AND frame_end ]# 当前窗口frame包括哪些rows/range)
```

* frame_start&frame_end的取值：current row、unbounded preceding、unbounded following、preceding 、following
* 不指定 `ORDER BY`，默认使用分区内所有行 “rows BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING【第二篇博客写的range，不过我觉得应该是rows】
* 若指定了 `ORDER BY`，默认使用分区内第一行到当前值 `rows BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`

* ROWS:rows mode directly specifies how many rows before or after the current row belong to the frame. rows between 5 preceding and 2 preceding.表示当前行前5行到当前行前两行范围
* Ranges：当前行的值（比如当前行的值是7.5）那么ranges between 5 preceding and 2 preceding.表示值在2.5到5.5的所有行，一般ranges在指定了order by之后才能用



窗口函数计算流程：

1. 按窗口定义，将所有输入数据分区、再排序（如果需要的话）
2. 对每一行数据，计算它的 Frame 范围
3. 将 Frame 内的行集合输入窗口函数，计算结果填入当前行



位置：窗口函数是对where或者group by子句处理后的结果进行操作，执行位置在selelct后,order by之前，所以**窗口函数原则上只能写在select子句中**。

<img src="https://pic3.zhimg.com/80/v2-3285d1d648de9f90864000d58847087a_1440w.jpg" alt="img" style="zoom:25%;" />

给出每个班级内的成绩排名(topn)

```text
select *,
   rank() over (partition by 班级
                 order by 成绩 desc) as ranking
from 班级表
```

首先对班级分区，然后按照成绩排序，frame为第一行到当前行，然后计算rank函数





group by vs 窗口函数：group by分组汇总后改变了表的行数，一行只有一个类别；而partiition by和rank函数不会减少原表中的行数

## 谓词

https://www.developerastrid.com/sql/sql-predicates/

就是sql中返回bool值的函数，包括：

- `LIKE`
- `BETWEEN`
- `IS NULL`、`IS NOT NULL`
- `IN`
- `EXISTS`







# 极客时间-sql必知必会

## 1基础

DataBase System=DataBase Management System+DBA=多个数据库（DB） + 管理程序 + DBA

DB=多个数据表

主流dbms和数据库模式：

<img src="https://static001.geekbang.org/resource/image/a7/91/a7237ddbe4ca69353bd21a6eff35d391.png" alt="img" style="zoom:50%;" />

NoSQL泛指非关系型数据库，包括了榜单上的键值型数据库、文档型数据库、搜索引擎和列存储等，除此以外还包括图形数据库。

## 2 sql执行过程

mysql中sql的执行过程：

MySQL是典型的C/S架构，即Client/Server架构，服务器端程序使用的mysqld，mysql分层架构为：

1. 连接层：客户端和服务器端建立连接，客户端发送SQL至服务器端；
2. SQL层：对SQL语句进行查询处理；
3. 存储引擎层：与数据库文件打交道，负责数据的存储和读取。

其中SQL层与数据库文件的存储方式无关，我们来看下SQL层的结构：

<img src="mysql.assets/30819813cc9d53714c08527e282ede79.jpg" alt="img" style="zoom:50%;" />

1. 查询缓存：Server如果在查询缓存中发现了这条SQL语句，就会直接将结果返回给客户端；如果没有，就进入到解析器阶段。需要说明的是，因为查询缓存往往效率不高，所以在MySQL8.0之后就抛弃了这个功能。
2. 解析器：在解析器中对SQL语句进行语法分析、语义分析。
3. 优化器：在优化器中会确定SQL语句的执行路径，比如是根据全表检索，还是根据索引来检索等。
4. 执行器：在执行之前需要判断该用户是否具备权限，如果具备权限就执行SQL查询并返回结果。在MySQL8.0以下的版本，如果设置了查询缓存，这时会将查询结果进行缓存。

你能看到SQL语句在MySQL中的流程是：SQL语句→缓存查询→解析器→优化器→执行器。在一部分中，MySQL和Oracle执行SQL的原理是一样的。

MySQL的存储引擎采用了插件的形式，比如Innodb、myisam

MySQL中对一条SQL语句的执行时间进行分析：

```
set profiling=1;//开启profiling可以让MySQL收集在SQL执行时所使用的资源情况
select xxxx;//执行某个sql语句，通过show profiles可以查看当前sql会话中所有query的id
show profile for queryId//展示这个查询的执行时间
```

## 3 sql语法

创建数据库、创建数据表、修改表结构的sql命令。

常见约束（主外key+unique、not null、check、default）

**Select**：

* 查询常数：select 1,name from a//在a表中查出name这个列，同时在查询结果集中新增一个全是1的列（查询常数常用于在结果集中新增一列用作标记之类的）

* Distinct：DISTINCT需要放到所有列名的前面，select distinct id, name from a(正确)、select id, distinct name from a(错误)；DISTINCT其实是对后面所有列名的组合进行去重【不会有相同的(id, name)】
* Orderby:ORDER BY可以使用非选择列进行排序，所以即使在SELECT后面没有这个列名，你同样可以放到ORDER BY后面进行排序。甚至可以用多个字段之间计算出的表达式排序。ORDER BY通常位于SELECT语句的最后一条子句，否则会报错。
* select执行顺序：FROM > WHERE > GROUP BY > HAVING > SELECT的字段 > DISTINCT > ORDER BY > LIMIT

在SELECT语句执行这些步骤的时候，每个步骤都会产生一个虚拟表，然后将这个虚拟表传入下一个步骤中作为输入。

**内置函数**：

sql内置了一些函数，这里介绍下转换函数：

<img src="https://static001.geekbang.org/resource/image/5d/59/5d977d747ed1fddca3acaab33d29f459.png" alt="img" style="zoom:50%;" />

`SELECT CAST(123.123 AS INT)`，运行结果会报错。

`SELECT CAST(123.123 AS DECIMAL(8,2))`，运行结果为123.12。

`SELECT COALESCE(null,1,2)`，运行结果为1。

CAST函数在转换数据类型的时候，不会四舍五入，如果原数值有小数，那么转换为整数类型的时候就会报错。在MySQL中，可以用 `DECIMAL(a,b)`来指定。

**聚集函数**

5大聚集函数：多行返回一个统计值。

`COUNT(字段名)`会忽略值为NULL的数据行，而COUNT(*)只是统计数据行数，不管某个字段是否为NULL；AVG、MAX、MIN等聚集函数会自动忽略值为NULL的数据行

Having&where:对于分组的筛选，我们一定要用HAVING，而不是WHERE。另外你需要知道的是，HAVING支持所有WHERE的操作，因此所有需要WHERE子句实现的功能，你都可以使用HAVING对分组进行筛选。

## 4 子查询

分为非关联子查询和关联子查询：

子查询从数据表中查询了数据结果，如果这个数据结果只执行一次，然后这个数据结果作为主查询的条件进行执行，那么这样的子查询叫做非关联子查询：

```
SELECT player_name, height FROM player WHERE height = (SELECT max(height) FROM player)
```

子查询需要执行多次，即采用循环的方式，先从外部查询开始，每次都传入子查询进行查询，然后再将结果反馈给外部，这种嵌套的执行方式就称为关联子查询【一般可以和in、exists一起使用以便传入条件】

* 关联子查询通常也会和EXISTS一起来使用，EXISTS子查询用来判断条件是否满足，满足的话为True，不满足为False[关联子查询中用到了不在子查询中表player_score的字段player.player_id]。对于exists来说，只要子查询返回记录了就返回true，一条记录都没返回就返回false:

```sql
#在这个统计中，是否出场是通过player_score这张表中的球员出场表现来统计的，要看出场过的球员都有哪些，并且显示他们的姓名、球员ID和球队ID
SELECT player_id, team_id, player_name FROM player WHERE EXISTS (SELECT player_id FROM player_score WHERE player.player_id = player_score.player_id)
```

```sql
#使用In实现上面功能
SELECT player_id, team_id, player_name FROM player WHERE player_id in (SELECT player_id FROM player_score WHERE player.player_id = player_score.player_id)#这里没必要加上后面的where吧
```

IN VS EXIST，某些情况下in可以实现和EXIST一样的效果：

```sql
SELECT * FROM A WHERE cc IN (SELECT cc FROM B)#非关联形式
SELECT * FROM A WHERE EXIST (SELECT cc FROM B WHERE B.cc=A.cc)#这里子查询如果返回行了，
```

https://segmentfault.com/a/1190000023825926

实际上在查询过程中，在我们对cc列建立索引的情况下，我们还需要判断表A和表B的大小。在这里例子当中，表A指的是player表，表B指的是player_score表。如果表A比表B大，那么IN子查询的效率要比EXIST子查询效率高，因为这时B表中如果对cc列进行了索引，那么IN子查询的效率就会比较高。

原因是小表驱动大表，先查小表再查询大表，两个表做关联时，需要把一张表join到另一张表的每一条记录上，所以需要让大的表去join到小表的每一条记录上：

```
for（i：200）{for (j:200000){...}}
```

20万条记录的表join到200张表的每一条记录上，主动join去匹配记录上的表是被驱动表，被匹配的是驱动表，那小表驱动大表就是让小表成为被join的表，A驱动B:=>A表的一条记录和B表做笛卡尔积，先对驱动表做循环

1.当使用left join时，左表是驱动表，右表是被驱动表 ;
2.当使用right join时，右表是驱动表，左表是被驱动表 ;
3.当使用inner join时，mysql会选择数据量比较小的表作为驱动表，大表作为被驱动表 ;

- 如果小的循环在外层，对于表连接来说就只连接200次 ;
- 如果大的循环在外层，则需要进行20万次表连接，从而浪费资源，增加消耗 ;

所以，对于IN来说是先执行完子查询，得一个表B，相当于SELECT * FROM A WHERE A.cc =B.cc【拿整个A表的记录去匹配B.cc】，是用B去驱动A，所以B越小越好。

同样，如果表A比表B小，那么使用EXISTS子查询效率会更高，相当于SELECT * FROM B WHERE B.cc =A.cc拿出a的一条记录去匹配整个b的记录，a驱动b，a越小效率越高

ANY【与子查询返回的任意值比较】 & ALL【与子查询返回的所有值比较】：需要使用比较符

```
SELECT player_id, player_name, height FROM player WHERE height > ALL (SELECT height FROM player WHERE team_id = 1002)

SELECT player_id, player_name, height FROM player WHERE height > ANY (SELECT height FROM player WHERE team_id = 1002)

```

## 5 连接

两张表的等值连接就是用两张表中都存在的列进行连接。我们也可以对多张表进行等值连接。

```
SELECT player_id, player.team_id, player_name, height, team_name FROM player, team WHERE player.team_id = team.team_id
```

当我们进行多表查询的时候，如果连接多个表的条件是等号时，就是等值连接，其他的运算符连接就是非等值查询。

```
SELECT p.player_name, p.height, h.height_level
FROM player AS p, height_grades AS h
WHERE p.height BETWEEN h.height_lowest AND h.height_highest
```

外连接：外连接还可以查询某一方不满足条件的记录。两张表的外连接，会有一张是主表，另一张是从表。如果是多张表的外连接，那么第一张表是主表，即显示全部的行，而第剩下的表则显示对应连接的信息。

```
SELECT * FROM player, team where player.team_id = team.team_id(+)#SQL92中采用（+）代表从表所在的位置
SELECT * FROM player LEFT JOIN team on player.team_id = team.team_id#sql99
```

自连接：可以对多个表进行操作，也可以对同一个表进行操作。也就是说查询条件使用了当前表的字段。

```
#查看比布雷克·格里芬高的球员都有谁，以及他们的对应身高：
SELECT b.player_name, b.height FROM player as a , player as b WHERE a.player_name = '布雷克-格里芬' and a.height < b.height
```

SQL99中提供的连接：

* CROSS JOIN：相当于sql92中的笛卡尔积

* NATURAL JOIN：自动查询两张连接表中所有相同的字段，然后进行等值连接；代替了sql92的等值连接

* 当我们进行连接的时候，可以用USING指定数据表里的同名字段进行等值连接。比如：

  ```
  SELECT player_id, team_id, player_name, height, team_name FROM player JOIN team USING(team_id)
  等价于
  SELECT player_id, player.team_id, player_name, height, team_name FROM player JOIN team ON player.team_id = team.team_id
  ```

* 全外连接：FULL JOIN 或 FULL OUTER JOIN ，OUTER一般可省略

Sql92/99是sql语法层面的东西，而具体的dbms比如mysql就不支持FULL OUTER JOIN

## 6 视图

视图【虚拟表】，相当于是一张表或多张表的数据结果集

创建视图：

```
CREATE VIEW 视图名 AS (sql查询)
```

当视图创建之后，它就相当于一个虚拟表，可以直接使用，在这个视图上select

## 7 存储过程

SQL语句的封装。一旦存储过程被创建出来，使用它就像使用函数一样简单

