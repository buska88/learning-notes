# hive

博客

https://www.hadoopdoc.com/hive/hive-tutorial

## sql解析

### 1 join时on和where

Hive中，A left join B时，B表(右表)的过滤条件写在on和where中，会导致不同的查询结果。

右表过滤条件在On：

select * from a  left join b  on a.id=b.id and b.dt='2012';  

对右表的过滤条件在on中，实际执行时会先根据b.dt过滤b表，然后再跟a表join



过滤条件在where：

select * from a  left join b  on a.id=b.id where b.dt='2012';  

对右表的过滤条件在where中，实际执行时会先根据on把ab连接起来，然后根据b.dt过滤，

有什么区别？

```
a表
id
1
2
b表
id   dt
1    2013
```

过滤条件在on时，首先拿b.dt='2012';  对b进行过滤，得到一个null记录，那么join的时候结果为：

```
a.id   b.id    b.dt
1      null     null
2      null    null
```

最终结果有两条记录

过滤条件在where时，先连起来：

```
a.id   b.id    b.dt
1      1       2013
2      null    null
```

最后按照b,dt过滤，最后结果一条记录都没了

可见，放在where中比放在on中少记录。

on中先执行b.dt的行为是谓词下推，不对保护表成立(outer join中保留连接不上行的表，例如left join的左表，right join的右表，full outer join的左右表),只对补空表(outer join中需要补NULL的表，例如left join的右表，right join的左表，full outer join的左右表)

```
select * from a left join b on a.id=b.id and a.id=1
```

这种情况会对a进行过滤吗？

不会，而是以"a.id=b.id and a.id=1"作为a与b的笛卡尔积能否连接上的条件，对于a表中连接不上的行，还需要补null，最终结果还是

```
a.id   b.id    b.dt
1      null     null
2      null    null
```

会保留a.id=2的字段。

