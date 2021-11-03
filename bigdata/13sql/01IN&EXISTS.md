# IN & EXISTS

## in和exists

- in是把外表和内表作hash连接
- exists是对外表作loop循环，每次loop循环再对内表进行查询
- 如果查询的两个表大小相当，那么用in和exists差别不大
- 如果两个表中一个较小一个较大，则子查询表大的用exists，子查询表小的用in

```sql
-- 适用于A大表 B小表
select * from A where cc in(select cc from B)　　-->用到了A表上cc列的索引

-- 适用于A小表 B大表
select * from A where exists(select cc from B where cc=A.cc)　　-->用到了B表上cc列的索引

```

## not in 和not exists

- 如果查询语句使用了not in，那么**对内外表都进行全表扫描，没有用到索引**
- 而not exists的子查询依然能用到表上的索引。**所以无论哪个表大，用not exists都比not in 要快**

```sql
create table t1(c1 int,c2 int);

create table t2(c1 int,c2 int);

insert into t1 values(1,2);

insert into t1 values(1,3);

insert into t2 values(1,2);

insert into t2 values(1,null);

 
-- 执行结果为空
-- 如果子查询中返回的任意一条记录含有空值，则查询将不返回任何记录!
select * from t1 where c2 not in(select c2 from t2);　

-- 执行结果为 1 3
select * from t1 where not exists(select 1 from t2 where t2.c2=t1.c2)　　-->执行结果：1　　3

```

