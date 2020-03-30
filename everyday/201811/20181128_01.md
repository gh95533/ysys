[TOC]

# greenplum table lot's record

**documents support**

ysys

**date**

2018-11-28

**label**

greenplum,table design



## background

​	同事问了我一个问题，在gp数据库中有一张表，每天的入库量在3亿左右，这张表该如何设计，我让同事给我发了一下表结构,字段不是很多，不超过30个。



## table design

​	因为数据量较大，脑海中蹦出来的是appendonly+compress,column table,partition table 这三种类型。

### appendonly+compress

​	适合场景：

​	1、业务上不对表进行update+delete操作，使用truncate + insert 方式

​	2、访问表时要求是全表扫描，就没有必要对表进行全表扫描

​	3、表结构比较固定，稳定后就不会修改

### column table 

​	适合场景

​	1、适合宽表（字段非常多的表）

​	2、查询主要是针对几个字段

​	3、压缩率适合内容相近时，更能减少空间占用

### partition table 

​	分区表只是普通表的不同表现形式，本质上就是普通表。

​	适合场景

​	1、通过某个字段分区，减小其他io(非本分区)消耗

​	2、创建索引等较为方便

​	3、可以删除某个分区数据而不影响数据库性能

**堆表、压缩表、列表size**

​	堆表

```
# create table test_heap  as select generate_series(1,100000) a,'helloworld'::varchar(50) b distributed by (a);
SELECT 100000

# select pg_size_pretty(pg_total_relation_size('test_heap'));
 pg_size_pretty 
----------------
 4512 kB
```

​	压缩表

```
# create table test_appendonly with (appendonly=true,compresslevel=5) as select generate_series(1,100000) a,'helloworld'::varchar(50) b distributed by (a);
SELECT 100000

# select pg_size_pretty(pg_total_relation_size('test_appendonly'));
 pg_size_pretty 
----------------
 1138 kB
```

​	列表

```
# create table test_column with (appendonly=true,orientation=column,compresslevel=5) as select generate_series(1,100000) a,'helloworld'::varchar(50) b distributed by (a);
SELECT 100000

# select pg_size_pretty(pg_total_relation_size('test_column'));
 pg_size_pretty 
----------------
 878 kB
```

​	所占大小一般情况下，堆表>压缩表>列表



## 遗留问题

​	明天和同事问一下，当前的业务到底是如何的。