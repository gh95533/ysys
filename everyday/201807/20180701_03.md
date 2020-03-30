- [TOC]


# pg_技术内幕

​	摘自《POSTGRESQL 从小工到专家》



## 遗留问题

​	1、事务回滚，之前的变更都不会生效，但是xmax反而发生变化。

​	可能解决这个疑问:

​	https://www.postgresql.org/docs/9.5/static/pageinspect.html

​	2、冻结事务frezid是否超过2^31后自动执行，是最大和最小事务id吗？还是根据是表的最大id和最小id,还是						数据库的最小id和最大id

​	3、	是否只能update才会引起多版本？





## 说明

​	本章节第一小节主要讲述了oid,tableoid,xmin,xmax,cmin,cmax,ctid在数据行的隐藏或者没有开启的字段，第二小节讲述了多版本并发控制(这个需要好好了解)，第三部分讲述了物理存储结构，第四部分讲述了部分技术并解密相关原理。

## 表中的系统字段

​	表中系统字段主要讲述如下几个，分别是oid,xmin,xmax,cmin,cmax,ctid

​	这里先说一下tableoid，tableoid指的时所属表的oid,那么什么场景可能会用到，在9.x系列中经常出现继承表，当子表继承父表，如果在查询语句中查询父表，默认情况下会出现子表数据，那么我们如何区分到底是子表数据还是父表独有数据时，不需要非得使用select * from only father_table来查看父表独有数据，可以添加tableoid字段来区分是否是子表，还是父表，还是其他一系列继承表。



### OID

​	接触oid最早是在pg_class表中，这个pg_class表存储表，索引，视图等一系列对象。在pg_class的表结构(使用\d展现)都没有找到oid字段，当时就觉得好神奇。

​	小技巧:查看这张表是否有oid字段,可以使用\d+ tabname，查看最后一行是否拥有"HASH OIDS:YES"

```
osdba=# \d+ pg_class;
                          Table "pg_catalog.pg_class"
     Column     |   Type    | Modifiers | Storage  | Stats target | Description 
----------------+-----------+-----------+----------+--------------+-------------
 relname        | name      | not null  | plain    |              | 
 relnamespace   | oid       | not null  | plain    |              | 
 reltype        | oid       | not null  | plain    |              | 
 reloftype      | oid       | not null  | plain    |              | 
 relowner       | oid       | not null  | plain    |              | 
 relam          | oid       | not null  | plain    |              | 
 relfilenode    | oid       | not null  | plain    |              | 
 reltablespace  | oid       | not null  | plain    |              | 
 relpages       | integer   | not null  | plain    |              | 
 reltuples      | real      | not null  | plain    |              | 
 relallvisible  | integer   | not null  | plain    |              | 
 reltoastrelid  | oid       | not null  | plain    |              | 
 relhasindex    | boolean   | not null  | plain    |              | 
 relisshared    | boolean   | not null  | plain    |              | 
 relpersistence | "char"    | not null  | plain    |              | 
 relkind        | "char"    | not null  | plain    |              | 
 relnatts       | smallint  | not null  | plain    |              | 
 relchecks      | smallint  | not null  | plain    |              | 
 relhasoids     | boolean   | not null  | plain    |              | 
 relhaspkey     | boolean   | not null  | plain    |              | 
 relhasrules    | boolean   | not null  | plain    |              | 
 relhastriggers | boolean   | not null  | plain    |              | 
 relhassubclass | boolean   | not null  | plain    |              | 
 relispopulated | boolean   | not null  | plain    |              | 
 relreplident   | "char"    | not null  | plain    |              | 
 relfrozenxid   | xid       | not null  | plain    |              | 
 relminmxid     | xid       | not null  | plain    |              | 
 relacl         | aclitem[] |           | extended |              | 
 reloptions     | text[]    |           | extended |              | 
Indexes:
    "pg_class_oid_index" UNIQUE, btree (oid)
    "pg_class_relname_nsp_index" UNIQUE, btree (relname, relnamespace)
    "pg_class_tblspc_relfilenode_index" btree (reltablespace, relfilenode)
Has OIDs: yes

```

postgresql在内部使用对象标识符(oid)作为各种系统的主键。

系统不会给用户创建的表增加一个oid系统字段，当然用户可以在创建表时添加“with oids"就可以创建出来，并且可以查询添加oid就可以了。

```
tutorial=# create table t_1(id int4,name text) with oids;
CREATE TABLE
tutorial=# \d+ t_1;
                          Table "public.t_1"
 Column |  Type   | Modifiers | Storage  | Stats target | Description 
--------+---------+-----------+----------+--------------+-------------
 id     | integer |           | plain    |              | 
 name   | text    |           | extended |              | 
Has OIDs: yes

tutorial=# select * from t_1;
 id | name 
----+------
(0 rows)

tutorial=# select oid,id,name from t_1;
 oid | id | name 
-----+----+------
(0 rows)

tutorial=# insert into t_1 values(1,'gh');
INSERT 70142 1
tutorial=# select * from t_1;
 id | name 
----+------
  1 | gh
(1 row)

tutorial=# select oid,id,name from t_1;
  oid  | id | name 
-------+----+------
 70142 |  1 | gh
(1 row)

tutorial=# 

```

再看如下一条语句

```
tutorial=# create table t_2(id int4,name text);
CREATE TABLE
tutorial=# insert into t_2 values(1,'gh');
INSERT 0 1

```

为什么同时让表内插入数据，第一个返回"INSERT 70142 1",而第二个返回"INSERT 0 1"这就是因为前者表中拥有oid,后者表中没有该字段，第二列数据返回oid值，如果没有oid字段就变为0，第三列代表数据量。

```
tutorial=# insert into t_1 select generate_series(2,1000),'gh';
INSERT 0 999
tutorial=# select oid,* from t_1 limit 10;
  oid  | id | name 
-------+----+------
 72150 |  2 | gh
 72151 |  3 | gh
 72152 |  4 | gh
 72153 |  5 | gh
 72154 |  6 | gh
 72155 |  7 | gh
 72156 |  8 | gh
 72157 |  9 | gh
 72158 | 10 | gh
 72159 | 11 | gh
(10 rows)


```

那么当前情况下，为何在第二列返回0,而不是返回相对性oid呢，我的猜测是因为返回的数据量太多，将所有的oid展现出来，不是很美观，所以当多条数据一次性插入是，就将第二列置为0



oid 是否可以作为用户创建表时默认的主键？

​	目前oid类型用一个四字节的无符号整数实现，它不能提供大数据范围内的唯一性保证，甚至在单个大表中也不行(达到它的上限，可能oid又会从0开始向最大值移动)



表(包括toast表)、索引，视图的对象标识符就是系统表pg_class的oid字段的值。



表1-1 对象标识符类型

| 类型名称      | 引用         | 描述                 | 数值列子           |
| ------------- | ------------ | -------------------- | ------------------ |
| oid           | 任意         | 数字化的对象标识符   | 36544              |
| regproc       | pg_proc      | 函数名字             | sum                |
| regprocedure  | pg_proc      | 带参数类型的函数     | sum(int)           |
| regoper       | pg_operator  | 操作符名             | +                  |
| regoperetor   | pg_operator  | 带参数类型的操作符名 | *(integer,integer) |
| regclass      | pg_class     | 表名或者索引名       | pg_type            |
| regtype       | pg_type      | 数据类型名           | integer            |
| regconfig     | pg_ts_config | 全文检索配置         | English            |
| regdictionary | pg_ts_dict   | 全文检索路径         | Simple             |

**上面还不是太明白之间的关系与逻辑**

```
tutorial=# select oprname,oprleft::regtype,oprright::regtype,oprresult::regtype,oprcode from pg_operator limit 2;
 oprname | oprleft | oprright | oprresult | oprcode 
---------+---------+----------+-----------+---------
 =       | integer | bigint   | boolean   | int48eq
 <>      | integer | bigint   | boolean   | int48ne
```



### CTID



​	在oracle中使用的rowid来表示表内的物理位置。ctid字段是postgresql数据库表示他所在的物理位置，ctid字段类型是tid，尽管ctid可以快速定位数据所在的位置，但每次vacuum full之后，数据行在块内的物理位置就会移动，ctid不能作为长期的行标识符的。

​	查看ctid示例

```
tutorial=# select ctid, * from t_1 limit 10;
  ctid  | id |   info   
--------+----+----------
 (0,1)  |  1 | tutorial
 (0,2)  |  2 | tutorial
 (0,3)  |  3 | tutorial
 (0,4)  |  4 | tutorial
 (0,5)  |  5 | tutorial
 (0,6)  |  6 | tutorial
 (0,7)  |  7 | tutorial
 (0,8)  |  8 | tutorial
 (0,9)  |  9 | tutorial
 (0,10) | 10 | tutorial

```

​	从上例可以看出，ctid是由两个数字组成，第一个数字代表物理块号，第二个数字代表物理块的行号

​	使用ctid查看数据

```
tutorial=# select ctid,* from t_1 where ctid ='(0,5)';
 ctid  | id |   info   
-------+----+----------
 (0,5) |  5 | tutorial
(1 row)

```

​	利用ctid来删除重复数据

```
tutorial=# insert into t_1 select generate_series(1,4),'tu';
INSERT 0 4
tutorial=# insert into t_1 select generate_series(1,10),'tutorial';
INSERT 0 10
tutorial=# select * from t_1 order by id;
 id |   info   
----+----------
  1 | tu
  1 | tutorial
  2 | tu
  2 | tutorial
  3 | tutorial
  3 | tu
  4 | tu
  4 | tutorial
  5 | tutorial
  6 | tutorial
  7 | tutorial
  8 | tutorial
  9 | tutorial
 10 | tutorial
(14 rows)

删除语句
tutorial=# delete from t_1 a where a.ctid <>(select min(b.ctid) from t_1 b where b.id = a.id);
DELETE 4

数据量多删除重复数据，上面语句执行起来非常慢

tutorial=# \timing
Timing is on.
tutorial=# truncate table t_1;
TRUNCATE TABLE
Time: 35.226 ms
tutorial=# insert into t_1 select generate_series(1,10),'tutorial';
INSERT 0 10
Time: 2.035 ms
tutorial=# insert into t_1 select generate_series(1,100000),'ghysys';
INSERT 0 100000
Time: 364.714 ms
tutorial=# explain delete from t_1 a where a.ctid <>(select min(b.ctid) from t_1 b where b.id = a.id);
                                 QUERY PLAN                                 
----------------------------------------------------------------------------
 Delete on t_1 a  (cost=0.00..179133452.50 rows=99510 width=6)
   ->  Seq Scan on t_1 a  (cost=0.00..179133452.50 rows=99510 width=6)
         Filter: (ctid <> (SubPlan 1))
         SubPlan 1
           ->  Aggregate  (cost=1791.13..1791.14 rows=1 width=6)
                 ->  Seq Scan on t_1 b  (cost=0.00..1791.12 rows=1 width=6)
                       Filter: (id = a.id)
(7 rows)

tutorial=# delete from t_1 a where a.ctid <>(select min(b.ctid) from t_1 b where b.id = a.id);
^CCancel request sent
ERROR:  canceling statement due to user request
Time: 352148.550 ms
由于执行时间较长，将sql停掉

tutorial=# DELETE FROM t_1 where ctid =ANY(ARRAY(select ctid from (select ctid,row_number() over (partition by id) from t_1) x where x.row_number>1));
DELETE 10
Time: 108.667 ms
```

因为第一条语句没有执行完成，手动停止，就这样比较一下执行时间发现都是几百倍的差距

### XMIN,XMAX,CMIN,CMAX



​	分析一下数据入库更新删除时XMIN,XMAX,CMIN,CMAX分别会发生什么变化。

​	1、假设场景如下：

​	首先将所有表的数据truncate，新启一个事务A，向表不同时间插入两条数据，检查xmin,xmax的情况，现在新启一个事务B查看当前表是否有数据，然后在A事务内更新数据多次，查看xmin,xmax变化情况，同时在事务B中查看数据是否存在，是否发生变化。

```
删除数据并新启一个事务A插入两条数据
tutorial=# truncate table t_1;
TRUNCATE TABLE
tutorial=# begin;
BEGIN
tutorial=# select txid_current();
 txid_current 
--------------
         6881
(1 row)

tutorial=# insert into t_1 values(1,'ysys');
INSERT 0 1
tutorial=# insert into t_1 values(2,'ysys');
INSERT 0 1
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
(2 rows)




事务B
tutorial=# BEGIN;
BEGIN
tutorial=# select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select * from t_1;
 id | info 
----+------
(0 rows)

tutorial=# 


```

说明一点：在A事务没有提交时，B不能看到数据情况。

```
事务A提交数据
tutorial=# select txid_current();
 txid_current 
--------------
         6881
(1 row)

tutorial=# COMMIT;
COMMIT
tutorial=# 


事务B查看数据情况
tutorial=# select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
(2 rows)


```

A事务提交后，B事务可以看到数据了。

现在新启一个事务A',开始更新添加数据，看看事务B是否能够追踪到变化。

```
事务A'
tutorial=# begin;
BEGIN
tutorial=#  select txid_current();
 txid_current 
--------------
         6883
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
(2 rows)

tutorial=# insert into t_1 values(3,'ysys');
INSERT 0 1
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
 6883 |    0 | (0,3) |  3 | ysys
(3 rows)



事务B

tutorial=# select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys


```

A'新入一条数据没有提交,B依然看不到。



在A'中删除id=1的数据，看看A',B的情况是否发生变化

```
事务A'中
tutorial=#  select txid_current();
 txid_current 
--------------
         6883
(1 row)
tutorial=# delete from t_1 where id = 1;
DELETE 1
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,2) |  2 | ysys
 6883 |    0 | (0,3) |  3 | ysys
(2 rows)


事务B
tutorial=#  select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6883 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
(2 rows)


```

如上图，A'中数据id=1,居然就真的没有了，在B事务中，id=1的数据存在，但是它的xmax发生变化,

变为了6883，就是事务A'的当前值。

在事务A‘中是否还可以访问到id=1的数据呢？

尝试使用ctid=(0,1)来访问

```
事务A'中
tutorial=# select txid_current();
 txid_current 
--------------
         6883
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1 where ctid = '(0,1)';
 xmin | xmax | ctid | id | info 
------+------+------+----+------
(0 rows)


```

在A'中还是无法访问已经删除，而在事务B中，依然可以访问没有提交的A'之前的数据:即(id=1)，如果此时回滚，id=1数据是存在，那么xmin,xmax又是如何变化的。

```
事务A'回滚
tutorial=# select txid_current();
 txid_current 
--------------
         6883
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1 ;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,2) |  2 | ysys
 6883 |    0 | (0,3) |  3 | ysys
(2 rows)

tutorial=# rollback;
ROLLBACK

事务B查看数据清空
tutorial=#  select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6883 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
(2 rows)


```

即便是事务A'回滚，xmax依然存在，表示这条数据被回滚，而之前在事务A'插入的数据随着回滚也不会出现在数据中。



开启新的事务A'',这次测试删除id=1成功，事务B是否查看有变化。

```
事务A''中
tutorial=# begin;
BEGIN
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6883 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
(2 rows)

tutorial=# select txid_current();
 txid_current 
--------------
         6884
(1 row)

tutorial=# delete from t_1 where id = 1;
DELETE 1
tutorial=# select txid_current();
 txid_current 
--------------
         6884
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,2) |  2 | ysys
(1 row)


事务B

tutorial=#  select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6884 | (0,1) |  1 | ysys
 6881 |    0 | (0,2) |  2 | ysys
(2 rows)


```

在B中id=1的xmin事务还是没有发生变化，而xmax的值确实发生了变化，首先符合规则中的更新或者删除时，xmax为当前操作的事务值，只是理解方面吗?后面执行update后，统一做说明。



```
事务A''
tutorial=# commit;
COMMIT

事务B

tutorial=#  select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,2) |  2 | ysys
(1 row)


```

事务A''提交后，事务B发现id=1被删除，后只是展现id=2的数据



测试更新情况

新启事务前，先执行命令入库一条新数据

```
tutorial=# insert into t_1 values(4,'df');
INSERT 0 1
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,2) |  2 | ysys
 6885 |    0 | (0,4) |  4 | df
(2 rows)

```

新启事务A''',更新id=2的数据，将其info字段值变为’gh‘，但不提交，通过事务B来查看变化情况

```
事务A'''

tutorial=# begin;
BEGIN
tutorial=# select txid_current();
 txid_current 
--------------
         6886
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 |    0 | (0,2) |  2 | ysys
 6885 |    0 | (0,4) |  4 | df
(2 rows)

tutorial=# update t_1 set info ='gh' where id = 2;
UPDATE 1
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6885 |    0 | (0,4) |  4 | df
 6886 |    0 | (0,5) |  2 | gh
(2 rows)

事务B

tutorial=#  select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6886 | (0,2) |  2 | ysys
 6885 |    0 | (0,4) |  4 | df
(2 rows)


```

通过B来看，这一条id=2的数据应该是被删除了，而通过A'''事务来看，id=2数据删除后就不存在了。如果此时事务A'''回滚，看看数据情况

```
事务A'''
tutorial=# select txid_current();
 txid_current 
--------------
         6886
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6885 |    0 | (0,4) |  4 | df
 6886 |    0 | (0,5) |  2 | gh
(2 rows)

tutorial=# rollback;
ROLLBACK

事务B

tutorial=#  select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6886 | (0,2) |  2 | ysys
 6885 |    0 | (0,4) |  4 | df
(2 rows)

```

想不想数据被删除时的场景，只不过多了一条新插入的数据。

如果事务A'''提交成功呢？

```
事务A'''
tutorial=# begin;
BEGIN
tutorial=# select txid_current();
 txid_current 
--------------
         6887
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6886 | (0,2) |  2 | ysys
 6885 |    0 | (0,4) |  4 | df
(2 rows)

tutorial=# update t_1 set info ='gh' where id = 2;
UPDATE 1
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6885 |    0 | (0,4) |  4 | df
 6887 |    0 | (0,6) |  2 | gh
(2 rows)

事务B
tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6881 | 6887 | (0,2) |  2 | ysys
 6885 |    0 | (0,4) |  4 | df
(2 rows)

事务A'''提交

tutorial=# commit;

事务B 
tutorial=#  select txid_current();
 txid_current 
--------------
         6882
(1 row)

tutorial=# select xmin,xmax,ctid,* from t_1;
 xmin | xmax | ctid  | id | info 
------+------+-------+----+------
 6885 |    0 | (0,4) |  4 | df
 6887 |    0 | (0,6) |  2 | gh
(2 rows)

```

​	update 可以解析成为delete then insert ,如果数据被update后回滚，那么数据就像delete方案使得，数据会保留原始，但是会出现xmax从一个值变成执行时的txid,而数据的xmin永远是数据插入xmid。



​	演示了这么多的insert ,update ,delete在不同的事务下的场景，导致数据的xmax发生变化，都忘记了cmin,cmax的字段信息。

​	

```
tutorial=# begin;
BEGIN
tutorial=# select txid_current();
 txid_current 
--------------
         6893
(1 row)

tutorial=# insert into t_1 values(1,'gh');
INSERT 0 1
tutorial=# select xmin,xmax,cmin,cmax,ctid,* from t_1;
 xmin | xmax | cmin | cmax | ctid  | id | info 
------+------+------+------+-------+----+------
 6893 |    0 |    0 |    0 | (0,1) |  1 | gh
(1 row)

tutorial=# insert into t_1 values(2,'gh');
INSERT 0 1
tutorial=# insert into t_1 values(3,'gh');
INSERT 0 1
tutorial=# select xmin,xmax,cmin,cmax,ctid,* from t_1;
 xmin | xmax | cmin | cmax | ctid  | id | info 
------+------+------+------+-------+----+------
 6893 |    0 |    0 |    0 | (0,1) |  1 | gh
 6893 |    0 |    1 |    1 | (0,2) |  2 | gh
 6893 |    0 |    2 |    2 | (0,3) |  3 | gh
(3 rows)

tutorial=# update t_1 set info = 'gh' where id = 1;
UPDATE 1
tutorial=# select xmin,xmax,cmin,cmax,ctid,* from t_1;
 xmin | xmax | cmin | cmax | ctid  | id | info 
------+------+------+------+-------+----+------
 6893 |    0 |    1 |    1 | (0,2) |  2 | gh
 6893 |    0 |    2 |    2 | (0,3) |  3 | gh
 6893 |    0 |    3 |    3 | (0,4) |  1 | gh
(3 rows)

tutorial=# update t_1 set info = 'gh2' where id = 1;
UPDATE 1
tutorial=# select xmin,xmax,cmin,cmax,ctid,* from t_1;
 xmin | xmax | cmin | cmax | ctid  | id | info 
------+------+------+------+-------+----+------
 6893 |    0 |    1 |    1 | (0,2) |  2 | gh
 6893 |    0 |    2 |    2 | (0,3) |  3 | gh
 6893 |    0 |    4 |    4 | (0,5) |  1 | gh2
(3 rows)

tutorial=# commit;
COMMIT
tutorial=# select xmin,xmax,cmin,cmax,ctid,* from t_1;
 xmin | xmax | cmin | cmax | ctid  | id | info 
------+------+------+------+-------+----+------
 6893 |    0 |    1 |    1 | (0,2) |  2 | gh
 6893 |    0 |    2 |    2 | (0,3) |  3 | gh
 6893 |    0 |    4 |    4 | (0,5) |  1 | gh2
(3 rows)

tutorial=# update t_1 set info ='gh1' where id = 2;
UPDATE 1
tutorial=# select xmin,xmax,cmin,cmax,ctid,* from t_1;
 xmin | xmax | cmin | cmax | ctid  | id | info 
------+------+------+------+-------+----+------
 6893 |    0 |    2 |    2 | (0,3) |  3 | gh
 6893 |    0 |    4 |    4 | (0,5) |  1 | gh2
 6894 |    0 |    0 |    0 | (0,6) |  2 | gh1
(3 rows)

tutorial=# 

```



​	在同一个事务内，每次变更命令都要加1，如果事务结束，那么当前保留最新的cmin,cmax

​	当新的 事务开启后，如果对于这条数据有变更，cmin,cmax是从0开始的,函数{x-[x]}有的类似，经过一个正变量后从0开始，一直到1,(这里认为1为一个上限值)

​	





## 多版本并发控制



​	MVCC(multi-version concurrency control)，缩写为MVCC,是指在数据库中并发访问数据时保持数据一致性的方法。

​	

### 多版本并发控制原理

​	在并发操作中，当正在写时，如果有用户在读，可能会产生数据不一致的问题。解决这个问题的办法使用读写锁。写的时候不允许读，正在读的时候不允许写，这种方式导致读和写不能并发执行。MVCC的方案就是写数据时，旧的版本数据并不删除，并发的读还能读取旧版本的数据。这样就不会有问题了，事先mvcc的方式主要有两种：

​	第一种：写数据时，把旧的数据移动到一个单独的地方，如回滚段中，其他人读数据时，从回滚段中把旧的数据读取出来

​	第二种：写新数据时，旧的数据不删除，而把新数据插入。

​	postgresql数据库使用的是第二种方式，而oracle数据库使用的是第一种方式。



### postgresql多版本并发控制

​	postgresql是通过把旧数据保留在数据文件中，新插入一条数据来实现多版本功能的，而且为了实现该功能，每张表上都添加了xmin,xmax,cmin,cmax,当两个事务同时访问这条记录时，通过参考xmin,xmax的标记来判断记录的版本，然后根据版本号与自己当前事务标识进行比较，确定自己的数据权限。当删除数据时，需要记住记录并没有从数据块中删除，空间也没有立即释放。

​	postgresql的多版本实现中首先要解决的是旧数据的空间释放问题。postgresql通过运行vacuum进程来回收之前的存储空间，默认postgresql数据跨库中的autovacuum是打开的，也就是当一个表的更新达到一定数量时，autovacuum自动回收空间。当然也可以关闭autovacuum，然后在业务低峰期手工运行vacuum来回收空间。

​	在postgresql中，若一个事务失败，在数据文件中这个事务产生的数据并不会在事务回滚时被清理掉。为何要这么做呢？为何不在事务提交时把数据标记成有效，而在事务回滚时，把这些数据标记为无效呢？这是处于效率的考虑。若事务提交或回滚时再次标记了数据，那这些数据就有可能被刷新到磁盘中，而再次标记会导致另一次I/O,从而降低了性能，那么如何知道这些数据是有效还是无效呢？postgresql数据库事通过记录事务的状态来实现的，因为数据行有记录xmin和xmax，所以只需要了解xmin和xmax对应的事务是成功提交了还是回滚了，就知道这些数据行是否有效。postgresql把事务状态记录在commit log中，简称clog,10后改名了，clog在数据目录的pg_clog子目录中

​	事务的状态有以下四种：

​	TRANSACTION_STATUS_IN_PROGRESS=0x00:表示事务正在进行中

​	TRANSACTION_STATUS_COMMITTED=0x01:表示事务已提交

​	TRANSACTION_STATUS_ABORTED=0x02:表示事务已回滚

​	TRANSACTION_STATUS_SUB_COMMITTED=0x03:表示子事务已提交

​	事务ID在POSTGRESQL中有时会缩写成xid,是一个32bit的数字，有以下三个特殊的事务ID是给系统内部使用的，它们代表特殊的意思

​	InvalidTransactionId=0;表示无效的事务ID

​	BootstrapTransactionId=1:表示系统表初始化时的事务ID

​	FrozenTransactionId=2:冻结的事务ID

​	所以数据库系统第一个正常的事务ID是从3开始的，然后不停递增，达到最大值后，再从3开始，事务ID为0,1,2的始终保留。

​	通常，使用值为0的事务代表一个无效的事务ID.

​	值为1的事务ID是initdb初始化系统表时，在表上所填写的事务ID,虽然此时数据库还没有启动，但在系统表中cmax也需要一个有效的事务ID,这个事务ID的值就是1。

```
osdba=# select cmin,cmax,relname from pg_class where relname in ('pg_type','pg_attribute');
 cmin | cmax |   relname    
------+------+--------------
    1 |    1 | pg_type
    1 |    1 | pg_attribute

```

​	若事务ID一直递增，总会到达4字节整数的最大值，达到最大值后在从头开始，以前的事务ID都会比当前的事务ID大，而在比较是，就会任务以前的事务ID是将来事务ID,这会导致严重问题，即事务ID回卷的问题。另外,postgresql多版本实现中经常需要判断事务之间的新旧关系，例如：如果数据行中已提交的事务比当前事务要旧，则在当前事务中这条数据是可见的。在事务ID 没有回卷时，简单比较两个事务ID的大小就可以知道事务的先后关系。为了解决事务回卷和满足比较事务新旧的需求，在postgresql中规定，最旧和最新的两个事务之前的年龄差最多是2^31，而不是无符号整数的最大值2^32,也就是说只有该范围的一般，当要超过2^31时，就把旧的事务换成一个特殊的事务ID,也就是Frezid，当正常的事务与frezid比较时，会认为正常事务ID比冻结ID新，做了上述规定后，对两个普通的事务ID比较新旧就可以使用下面的公式了：

```
((int32) (id1-id2))<0
id1代表旧的，id2代表新的
```

​	如果该公式的结果为真，则表明事务id1比事务id2旧，从这个公式可以看出，当事务ID没有回卷时，上面的公式相等于直接比较大小，在事务id回卷后，例如id1=4294967295,id2=5,那么id1-id2=4294967290,这是一个正数，但是转换成有符号的int32时，由于超出了有符号数的范围(2^31-1),因此会转换成一个负数，这样一来，其结果对于事务ID回滚后的情况也就可以试用了。

​	

### postgresql 多版本的优劣分析

​	oracle数据库实现了多版本的功能是将旧版本的数据记录到回滚端中，如果是读取旧版本的数据，需要根据回滚端的数据重构旧版本数据。

​	postgresql的多版本机制类似java虚拟机的垃圾回收机制有点像。事务提交前，只需要访问原始数据即可，而事务提交后，系统更新元祖的存储标识，知道vacuum进程收回为止。

​	相对于oracle数据库，postgresql多版本的优势在于：

- 事务回滚可以立即完成，无论事务进行多少操作。

- 数据可以进行很多更新，不会遇到回滚端不会被用完

  相对与oracle数据库，postgresql多版本的劣势在于：

- 旧版本的数据需要清理

- 旧版本的数据会导致查询变慢一些

  ​	

	​	

## 物理存储结构



​	postgresql数据库目前不支持使用裸设备(原始分区fdisk-l)和块设备(?),在postgresql数据库中，表里的数据总是存放在一个或者多个物理的数据文件之中。而相应的数据文件又分为多个固定大小的数据块(8kb)，数据就放在这些数据块中。



### postgresql中的术语

- relation

  指的是表或者索引

- tuple

  指的是行

- page

  表示在磁盘中的数据块

- buffer

  表示在内存中的数据块

### 数据块结构

​	数据块的大小默认是8kb,最大32kb,一个数据块中可存放多行的数据。块中的结构

![_](..\img_src\2018-07-04_pg_1.png)



是先有一个块头，其后记录了这个块中各个数据行的指针，行指针是向后按顺序排列的，而实际的数据行内容是从块尾向前反向排列的。行数据指针与行数据之间的部分则是空闲空间。

​	块头记录了如下一些信息

​	1、块的checksum值

​	2、空闲空间的起始位置和结束位置

​	3、特殊数据的起始位置

​	4、其他一些信息



### 行结构

​	行的物理结构是先有一个行头，后面跟了各项数据。行头中记录了一下重要信息。

​	oid,ctid,xmin,xmax,cmin,xmax

​	natts&infomask2:字段数，其中低11位表示这行有多少列

​	infomask:用于标识行当前的状态，比如行是否具有oid,是否有空属性

​        boff:表示行头的长度

​	bits:是一个数组，用于标识该行上那些字段为空

​	行上的xmin,xmax,cmin,cmax和clog日志一起用于控制行的可见性。每个事务在clog中占用两个bit,数据库运行一段时间后，如几年，就可能产生20亿事务，clog可能占用480M的空间，文件这么大，效率可能不高，于是postgresql对查询行的可见性做了优化，把一些可见性信息记录在infomask字段上，该字段的t_infomask中有以下与可见性相关的标识位

```
	define HEAP_XMIN_COMMITTED 0x0100

	define HEAP_XMIN_INVALID 0x0200

	define HEAP_XMAX_COMMITTED 0x0400

	define HEAP_XMAX_INVALID 0x0800

	define HEAP_XMAX_IS_MULTI 0x1000

```

​	如果t_infomask中HEAP_XMIN_COMMITTED为真，而HEAP_XMAX_INVALID为假，则说明这行是新插入的数据,是可见的，这是就不需要到clog中查询xmin,xmax的事务状态了。

​	如果没有设置HEAP_XMIN_COMMITTED,并不是代表这行数据没有提交，而是说不知道xmin是否提交，需要到clog中去判断xmin的状态，HEAP_XMAX_COMMITTED也是如此

​	第一次插入数据时，t_infomask中的HEAP_XMIN_COMMITTED和HEAP_XMAX_INVALID并没有设置，当事务提交后，有人再读取这个数据块时，会通过clog判断这些行的事务已提交，从而设置了t_infomask中HEAP_XMIN_COMMITTED和HEAP_XMAX_INVALID标志位，下次查询这行时，直接使用t_infomask中HEAP_XMIN_COMMITTED和HEAP_XMIN_INVALID标志位就可以判断行的可见性，不需要再到clog中查询。



### 数据块空闲空间管理

​	

​	postgresql数据库使用一个FSM的文件来记录每个数据块的空闲空间.

​	

### 可见性映射表文件



​	为了加快VACUUM清理的速度和降低对系统I/O性能的影响，postgresql从8.4后为每个数据文件增加了一个后缀名为"_vm"的文件，这个文件又被称为可见性映射文件。

​	vacuum清理数据两种方式，一种被称为lazy vacuum，另外一种被称为full vacuum,vm文件只在lazy vacuum中使用，full vacuum操作则需要对整个数据文件进行扫描。



## 技术解密



### index-only scan

​	

​	在postgresql数据库索引并没有所谓多版本信息，因此即使是SELECT 的列都是索引列，在postgresql之前的版本中，还需要到表上在查询一遍，但是在postgresql9.2后，就不需要了，这种扫描方式被称为“index-only sacns“，实际上9.2版本中也没有在索引中添加多版本信息，那它是如何实现这个功能的呢？它是通过可见映射表(visibility map)来实现的，可见映射表文件中记录了一个数据块中的行对于全部事务是否可见，当然还存在数据块中的行对部分事务可见，部分不可见的情况，但是前一种情况下，postgresql就能判断出从索引中返回的数据是否对当前事务可见，这样就不需要访问表中的行。



#### Heap-Only Tuples



​	在postgresql中因为多版本功能的原因，当更新一行时，实际上旧行并没有被删除，支持插入了一条新行，如果在表上有索引，而更新的字段不是索引的键值时，由于新行的物理位置发生了变化，因此仍然需要更新索引，这将导致性能下降，为了解决这个问题，postgresql从8.3版本后，引入了一个名为Heap-Only Tuple的新技术，简称HOT。使用HOT技术后，如果更新后的旧行与新行都在同一个数据块内，旧行会有一个指针，指向新行，这样就不必更新索引了



​	













​	











