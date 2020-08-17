[TOC]

# pg dba 5

​	

​	本章节主要讲述了执行计划，成本公式解说，代价因此校准，自动追踪sql执行计划



## explain

### explain 语法



explain:显示sql的执行计划

analyze:通过实际执行的sql来获取相应的执行计划

verbose:显示计划的附加信息，如计划树中每个节点输出的各个列。

costs：每个节点的启动成本和总成本，以及估计行数和每行的宽度。

buffers：显示缓存区使用的信息，该参数只能和analyze参数一起使用

format:指定的输出格式，text,xml,json,yaml

timing:输出时间开销



**需要注意analyze的使用，analyze会真的执行相关sql语句**

可以通过 begin; explain analyze sql; rollback;方式解决



### explain 输出

```
tutorial=# explain(analyze,verbose,costs,buffers,timing) select count(1) from test1;
                                                        QUERY PLAN                                                   
     
---------------------------------------------------------------------------------------------------------------------
-----
 Aggregate  (cost=1887.00..1887.01 rows=1 width=0) (actual time=24.904..24.904 rows=1 loops=1)
   Output: count(1)
   Buffers: shared hit=637
   ->  Seq Scan on public.test1  (cost=0.00..1637.00 rows=100000 width=0) (actual time=0.009..12.563 rows=100000 loop
s=1)
--seq scan on table:这个节点的路径
--0.00 表示第一行输出的成本，后面一个是总的成本，actual表示真实时间
         Output: id, info, crt_time
         Buffers: shared hit=637 --从shared buffer命中637page,如果在shared buffer中没有命中，就需要到os缓存或者硬盘中查找了read
 Planning time: 0.030 ms--计划执行时间
 Execution time: 24.936 ms--总的执行时间
(8 rows)

```



### 嵌套连接

nested loop join: The right relation is scanned once for every row found in the left relation. This strategy is easy to implement but can be very time consuming. (However, if the right relation can be scanned with an index scan, this can be a good strategy. It is possible to use values from the current row of the left relation as keys for the index scan of the right.)

嵌套循环

```
for  tuple in 左表查询 loop
右表查询(根据左表查询得到的行作为右表查询的条件依次输出最终结果)
end loop;
```



适合右表的关联列发生在唯一键值列或者主键上的情况



```
tutorial=# explain(analyze ,verbose,costs,buffers,timing) select t1.* from test1 t1,test2 t2 where t1.id = t2.id 
and t1.id < 100;
                                                                QUERY PLAN                                           
                     
---------------------------------------------------------------------------------------------------------------------
---------------------
 Nested Loop  (cost=0.43..39.86 rows=1 width=19) (actual time=0.048..0.270 rows=99 loops=1)
   Output: t1.id, t1.info, t1.crt_time
   Buffers: shared hit=200 read=1
   ->  Index Scan using idx_test1_id on public.test1 t1  (cost=0.29..10.10 rows=103 width=19) (actual time=0.009..0.0
29 rows=99 loops=1)
         Output: t1.id, t1.info, t1.crt_time
         Index Cond: (t1.id < 100)
         Buffers: shared hit=3
   ->  Index Only Scan using idx_test2_id on public.test2 t2  (cost=0.14..0.28 rows=1 width=4) (actual time=0.001..0.
002 rows=1 loops=99)
         Output: t2.id
         Index Cond: (t2.id = t1.id)
         Heap Fetches: 99
         Buffers: shared hit=197 read=1
 Planning time: 0.346 ms
 Execution time: 0.321 ms
(14 rows)


```



### HASH连接

hash join: the right relation is first scanned and loaded into a hash table, using its join attributes as hash keys. Next the left relation is scanned and the appropriate values of every row found are used as hash keys to locate the matching rows in the table

首先右表扫描加载到内存HASH表, hash key为JOIN列

然后左表扫描, 并与内存中的HASH表进行关联, 输出最终结果

```
tutorial=# explain(analyze ,verbose,costs,buffers,timing) select t1.* from test1 t1,test2 t2 where t1.id = t2.id 
and t1.id < 100;
                                                               QUERY PLAN                                                                
-----------------------------------------------------------------------------------------------------------------------------------------
 Hash Join  (cost=3.54..13.74 rows=1 width=19) (actual time=0.034..0.106 rows=99 loops=1)
   Output: t1.id, t1.info, t1.crt_time
   Hash Cond: (t1.id = t2.id)
   Buffers: shared hit=4
   ->  Index Scan using idx_test1_id on public.test1 t1  (cost=0.29..10.10 rows=103 width=19) (actual time=0.003..0.028 rows=99 loops=1)
         Output: t1.id, t1.info, t1.crt_time
         Index Cond: (t1.id < 100)
         Buffers: shared hit=3
   ->  Hash  (cost=2.00..2.00 rows=100 width=4) (actual time=0.027..0.027 rows=100 loops=1)
         Output: t2.id
         Buckets: 1024  Batches: 1  Memory Usage: 4kB
         Buffers: shared hit=1
         ->  Seq Scan on public.test2 t2  (cost=0.00..2.00 rows=100 width=4) (actual time=0.000..0.011 rows=100 loops=1)
               Output: t2.id
               Buffers: shared hit=1
 Planning time: 0.148 ms
 Execution time: 0.135 ms
(17 rows)

```

### MERGE连接

merge join: Each relation is sorted on the join attributes before the join starts. Then the two relations are scanned in parallel, and matching rows are combined to form join rows. This kind of join is more attractive because each relation has to be scanned only once. The required sorting might be achieved either by an explicit sort step, or by scanning the relation in the proper order using an index on the join key.



首先两个JOIN的表根据join key进行排序,然后根据join key的排序顺序并行扫描两个表进行匹配输出最终结果,适合大表并且索引列进行关联的情况 



```
tutorial=# explain(analyze ,verbose,costs,buffers,timing) select t1.* from test1 t1,test2 t2 where t1.id = t2.id 
and t1.id < 100;
                                                               QUERY PLAN                                                                
-----------------------------------------------------------------------------------------------------------------------------------------
 Merge Join  (cost=5.63..6.22 rows=1 width=19) (actual time=0.025..0.079 rows=99 loops=1)
   Output: t1.id, t1.info, t1.crt_time
   Merge Cond: (t1.id = t2.id)
   Buffers: shared hit=4
   ->  Index Scan using idx_test1_id on public.test1 t1  (cost=0.29..10.10 rows=103 width=19) (actual time=0.004..0.022 rows=99 loops=1)
         Output: t1.id, t1.info, t1.crt_time
         Index Cond: (t1.id < 100)
         Buffers: shared hit=3
   ->  Sort  (cost=5.32..5.57 rows=100 width=4) (actual time=0.020..0.023 rows=100 loops=1)
         Output: t2.id
         Sort Key: t2.id
         Sort Method: quicksort  Memory: 29kB
         Buffers: shared hit=1
         ->  Seq Scan on public.test2 t2  (cost=0.00..2.00 rows=100 width=4) (actual time=0.002..0.009 rows=100 loops=1)
               Output: t2.id
               Buffers: shared hit=1
 Planning time: 0.136 ms
 Execution time: 0.098 ms
(18 rows)


```



## explain 成本计算

​	成本计算相关的参数和系统表或视图

​	表或视图

- pg_stats

- pg_class --用到relpages和reltuples

  参数

- seq_page_cost --全表扫描的单个数据块的代价因子

- random_page_cost --索引扫描的单个数据块的代价因子

- cpu_tuple_cost --处理每条记录的CPU开销代价因子

- cpu_index_tuple_cost --索引扫描时每个索引条目的CPU开销代价因子

- cpu_operator_cost --操作符或函数的开销代价因子



### pg_stats

| 名字                     | 类型       | 引用                   | 描述                                                         |
| ------------------------ | ---------- | ---------------------- | ------------------------------------------------------------ |
| `schemaname`             | `name`     | `pg_namespace.nspname` | 包含此表的模式名字                                           |
| `tablename`              | `name`     | `pg_class.relname`     | 表的名字                                                     |
| `attname`                | `name`     | `pg_attribute.attname` | 这一行描述的字段的名字                                       |
| `inherited`              | `bool`     |                        | 如果为真，那么这行包含继承的子字段，不只是指定表的值。       |
| `null_frac`              | `real`     |                        | 记录中字段为空的百分比                                       |
| `avg_width`              | `integer`  |                        | 字段记录以字节记的平均宽度                                   |
| `n_distinct`             | `real`     |                        | 如果大于零，就是在字段中独立数值的估计数目。如果小于零，    就是独立数值的数目被行数除的负数。用负数形式是因为`ANALYZE`    认为独立数值的数目是随着表增长而增长；    正数的形式用于在字段看上去好像有固定的可能值数目的情况下。比如，    -1 表示一个唯一字段，独立数值的个数和行数相同。 |
| `most_common_vals`       | `anyarray` |                        | 一个字段里最常用数值的列表。如果看上去没有啥数值比其它更常见，则为 null |
| `most_common_freqs`      | `real[]`   |                        | 一个最常用数值的频率的列表，也就是说，每个出现的次数除以行数。    如果`most_common_vals`是 null ，则为 null。 |
| `histogram_bounds`       | `anyarray` |                        | 一个数值的列表，它把字段的数值分成几组大致相同热门的组。    如果在`most_common_vals`里有数值，则在这个饼图的计算中省略。    如果字段数据类型没有`<`操作符或者`most_common_vals`    列表代表了整个分布性，则这个字段为 null。 |
| `correlation`            | `real`     |                        | 统计与字段值的物理行序和逻辑行序有关。它的范围从 -1 到 +1 。    在数值接近 -1 或者 +1 的时候，在字段上的索引扫描将被认为比它接近零的时候开销更少，    因为减少了对磁盘的随机访问。如果字段数据类型没有`<`操作符，那么这个字段为null。 |
| `most_common_elems`      | `anyarray` |                        | 经常在字段值中出现的非空元素值的列表。（标量类型为空。）     |
| `most_common_elem_freqs` | `real[]`   |                        | 最常见元素值的频率列表，也就是，至少包含一个给定值的实例的行的分数。    每个元素频率跟着两到三个附加的值；它们是在每个元素频率之前的最小和最大值，    还有可选择的null元素的频率。（当`most_common_elems`    为null时，为null） |
| `elem_count_histogram`   | `real[]`   |                        | 该字段中值的不同非空元素值的统计直方图，跟着不同非空元素的平均值。（标量类型为空。） |

### 全表扫描

​	全表扫描的成本计算

```
tutorial=# analyze test1;
ANALYZE
tutorial=# explain select * from test1; 
                          QUERY PLAN                          
--------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..1637.00 rows=100000 width=17)
(1 row)
```

cost等于1637是怎么得来的，全表计算的成本只需要计算pg_class

```
tutorial=# select relpages,reltuples from pg_class where oid='test1'::regclass;
 relpages | reltuples 
----------+-----------
      637 |    100000
(1 row)
tutorial=# show seq_page_cost;
 seq_page_cost 
---------------
 1
(1 row)

tutorial=# show cpu_tuple_cost;
 cpu_tuple_cost 
----------------
 0.01
(1 row)

tutorial=# select 637*1+100000*0.01;
 ?column? 
----------
  1637.00
(1 row)

tutorial=# 

```

### 行数评估

#### 柱状图

```

tutorial=# explain select * from test1 where id < 2000;
                         QUERY PLAN                         
------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..1887.00 rows=1903 width=17)
   Filter: (id < 2000)
(2 rows)

```

rows等于1903是怎么得来的呢？

```
tutorial=# select * from pg_stats where tablename ='test1' and attname ='id';
-[ RECORD 1 ]----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
schemaname             | public
tablename              | test1
attname                | id
inherited              | f
null_frac              | 0
avg_width              | 4
n_distinct             | -1
most_common_vals       | 
most_common_freqs      | 
histogram_bounds       | {1,1017,2105,3251,4331,5285,6345,7319,8366,9317,10292,11254,12263,13307,14302,15333,16369,17328,18273,19272,20191,21184,22150,23182,24218,25229,26259,27228,28217,29110,30072,31065,32113,33017,34024,34994,35983,36977,37946,38875,39749,40665,41686,42651,43702,44636,45636,46590,47611,48608,49635,50655,51633,52660,53602,54593,55642,56551,57489,58473,59551,60514,61561,62556,63583,64657,65730,66650,67615,68716,69680,70654,71660,72723,73775,74757,75666,76693,77621,78610,79620,80655,81745,82675,83703,84689,85797,86802,87798,88772,89812,90768,91767,92809,93792,94862,95902,96883,97945,98990,100000}
correlation            | 1
most_common_elems      | 
most_common_elem_freqs | 
elem_count_histogram   | 

```



一共有100个柱面，其中2000位于1017~2105之间占比 之后占据总的占比

select (1+(2000-1017)/(2105-1017))/100.0;

```
tutorial=# select 0.01903492647058823529*100000;
-[ RECORD 1 ]-----------------------
?column? | 1903.49264705882352900000

```

#### MCV(most common values)

​	条件中查询的数据是最多的统计

```
tutorial=# explain select * from test1 where id = 1;
                         QUERY PLAN                          
-------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..3774.00 rows=99940 width=17)
   Filter: (id = 1)
(2 rows)

```

​	查看pg_class的reltuples和pg_stats的most_common_vals和most_common_freqs

```
tutorial=# select * from pg_stats where tablename='test1' and attname ='id';
-[ RECORD 1 ]----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
schemaname             | public
tablename              | test1
attname                | id
inherited              | f
null_frac              | 0
avg_width              | 4
n_distinct             | -0.13058
most_common_vals       | {1}
most_common_freqs      | {0.4997}
histogram_bounds       | {3,1027,1993,3159,4069,5268,6187,7245,8226,9380,10451,11377,12419,13349,14359,15457,16445,17442,18350,19398,20416,21424,22381,23408,24310,25260,26220,27221,28240,29306,30347,31292,32257,33190,34231,35219,36302,37284,38164,39186,40193,41272,42285,43235,44248,45269,46232,47131,48297,49305,50293,51294,52347,53355,54365,55409,56289,57315,58415,59271,60345,61369,62308,63160,64186,65366,66314,67370,68449,69398,70361,71485,72516,73536,74676,75740,76780,77697,78621,79534,80575,81489,82498,83344,84282,85234,86151,87237,88186,89135,90085,91106,92072,93028,94133,95091,96178,97171,98075,99022,100000}
correlation            | -0.499999
most_common_elems      | 
most_common_elem_freqs | 
elem_count_histogram   | 

tutorial=# select 200000*0.4997;
  ?column?  
------------
 99940.0000
(1 row)

```



#### MCV和distinct值个数评估行数

​	条件中查询的数据为一个distinct数据

```
tutorial=# explain select * from test1 where id = 1000;
                       QUERY PLAN                        
---------------------------------------------------------
 Seq Scan on test1  (cost=0.00..3774.00 rows=4 width=17)
   Filter: (id = 1000)
(2 rows)
```



```
tutorial=# select * from pg_stats where tablename='test1' and attname ='id';
-[ RECORD 1 ]----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
schemaname             | public
tablename              | test1
attname                | id
inherited              | f
null_frac              | 0
avg_width              | 4
n_distinct             | -0.128015
most_common_vals       | {1}
most_common_freqs      | {0.5054}
histogram_bounds       | {5,1007,2003,2962,3910,4930,5881,6787,7747,8757,9765,10748,11684,12562,13575,14598,15581,16612,17632,18701,19638,20693,21755,22786,23728,24670,25666,26718,27759,28654,29580,30387,31476,32669,33701,34730,35746,36713,37751,38790,39869,40693,41771,42853,43737,44821,45883,46827,47820,48753,49756,50729,51718,52835,53809,54911,55818,56774,57704,58902,59839,60822,61750,62775,63886,64868,65874,66752,67712,68733,69718,70863,71890,72876,73864,74775,75790,76938,77951,78982,80002,80903,81916,82912,84017,84957,86041,87090,88177,89154,90210,91136,92115,93071,94210,95267,96244,97118,98160,99121,99999}
correlation            | -0.499825
most_common_elems      | 
most_common_elem_freqs | 
elem_count_histogram   | 

```



```
(1-sum(mcf))(sumn_distinct-count(mcv))sum
```

```
tutorial=# select (1-0.5054)/(25603-1)*200000;
-[ RECORD 1 ]------------------------
?column? | 3.863760643699710960000000

```



#### mcv和柱状图

​	条件中既包含MCV有落在柱状图的情况，柱状图的统计不包括MCV的值，所以从柱状图中计算航的选择性时，要乘以一个系数，这个系数是1减去MCF的总和

```
tutorial=# explain select * from test1 where id <99121;
                          QUERY PLAN                          
--------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..3774.00 rows=199011 width=17)
   Filter: (id < 99121)
(2 rows)

```

```
tutorial=# select * from pg_stats where tablename='test1' and attname ='id';
-[ RECORD 1 ]----------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
schemaname             | public
tablename              | test1
attname                | id
inherited              | f
null_frac              | 0
avg_width              | 4
n_distinct             | -0.128015
most_common_vals       | {1}
most_common_freqs      | {0.5054}
histogram_bounds       | {5,1007,2003,2962,3910,4930,5881,6787,7747,8757,9765,10748,11684,12562,13575,14598,15581,166,32669,33701,34730,35746,36713,37751,38790,39869,40693,41771,42853,43737,44821,45883,46827,47820,48753,49756,50729,512,67712,68733,69718,70863,71890,72876,73864,74775,75790,76938,77951,78982,80002,80903,81916,82912,84017,84957,86041,8
correlation            | -0.499825
most_common_elems      | 
most_common_elem_freqs | 
elem_count_histogram   | 

tutorial=# select (1-0.5054)/100;
        ?column?        
------------------------
 0.00494600000000000000
(1 row)

tutorial=# select 1-(1-0.5054)/100;
        ?column?        
------------------------
 0.99505400000000000000
(1 row)

tutorial=# select  0.99505400000000000000*200000;
          ?column?           
-----------------------------
 199010.80000000000000000000
(1 row)

tutorial=# 

```

#### 多列条件



​	相互独立事件

```
tutorial=# explain select * from test1 where id = 1 and info ='guose';
                         QUERY PLAN                          
-------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..4274.00 rows=49962 width=18)
   Filter: ((id = 1) AND (info = 'guose'::text))
(2 rows)

```

```
tutorial=# select * from pg_stats where tablename='test1' and attname ='id';
-[ RECORD 1 ]----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
schemaname             | public
tablename              | test1
attname                | id
inherited              | f
null_frac              | 0
avg_width              | 4
n_distinct             | -0.12921
most_common_vals       | {1}
most_common_freqs      | {0.502733}
histogram_bounds       | {3,989,2005,3061,3991,4923,5894,6949,7921,8857,9831,10793,11698,12696,13674,14720,15668,16768,17806,18720,19774,20763,21783,22629,23713,24835,25807,26892,27938,28940,29834,30759,31845,32824,33822,34701,35766,36800,37993,39038,40044,41015,41947,43018,44125,45155,46116,47084,48072,49102,50113,51209,52150,53228,54222,55133,56214,57189,58238,59200,60319,61279,62324,63350,64286,65202,66097,67096,68066,69045,69995,71109,72208,73218,74159,75099,76166,77112,78147,79106,80109,81113,82048,83107,84063,85011,86116,87156,88286,89235,90171,91189,92076,93079,94044,95230,96138,97043,98105,99036,99989}
correlation            | 0.999444
most_common_elems      | 
most_common_elem_freqs | 
elem_count_histogram   | 

tutorial=# select * from pg_stats where tablename='test1' and attname ='info';
-[ RECORD 1 ]----------+----------------
schemaname             | public
tablename              | test1
attname                | info
inherited              | f
null_frac              | 0
avg_width              | 6
n_distinct             | 2
most_common_vals       | {guoqi,guose}
most_common_freqs      | {0.5031,0.4969}
histogram_bounds       | 
correlation            | 1
most_common_elems      | 
most_common_elem_freqs | 
elem_count_histogram   | 


```



```
tutorial=# select 200000*(0.502733)*(0.4969);
     ?column?     
------------------
 49961.6055400000
(1 row)

```



### explain 成本计算



​	索引扫描时，和全表扫描不同，扫描PAGE的开销是pages*random_page_cost

​	另外，索引扫描一般都涉及到操作符，例如大于小于等于

​	这些操作符对应的函数的coast乘以cpu_operator_cost就得到这个操作符的代价因子，乘以实际操作的行数就得到CPU操作符开销，rows explain中可能无输出

​	索引的CPU开销则是实际扫描的索引条目乘以cpu_index_tuple_cost,row explain中无输出

​	最后一个开销是实际返回或丢给上层的TUPLE带来的CPU开销，cpu_tuple_cost*实际扫描的行数。Rows Explain中有输出



## explain 代价因子校准

​	略过



## auto_explain插件的使用

​	auto_explain的目的是给数据库中执行的sql语句一个执行时间阈值。超过这个阈值的话，记录下当时这个sql的执行计划到日志中，便于未来查看这个sql执行计划有没有问题



### 编译运行

​	到/contrib/auto_explain下

```
[osdba@mysql45 auto_explain]$ make
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -O2 -fpic -I. -I. -I../../src/include -D_GNU_SOURCE   -c -o auto_explain.o auto_explain.c
mgcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Wendif-labels -Wmissing-format-attribute -Wformat-security -fno-strict-aliasing -fwrapv -O2 -fpic -shared -o auto_explain.so auto_explain.o -L../../src/port -L../../src/common -Wl,--as-needed -Wl,-rpath,'/usr/local/pgsql/lib',--enable-new-dtags  
[osdba@mysql45 auto_explain]$ make install
/bin/mkdir -p '/usr/local/pgsql/lib'
/usr/bin/install -c -m 755  auto_explain.so '/usr/local/pgsql/lib/auto_explain.so'
[osdba@mysql45 auto_explain]$ 

```

​	

### 会话使用

```
tutorial=# load 'auto_explain';
LOAD
tutorial=# set auto_explain.log_min_duration=0;--设置sql执行时间执行阈值
SET
tutorial=# select * from test1 limit 1;
 id | info  |      crt_time       
----+-------+---------------------
  2 | guoqi | 2018-08-09 09:14:22
(1 row)

```

```
$ tail -200f postgresql-*.csv
Query Text: select * from test1 limit 1;
Limit  (cost=0.00..0.02 rows=1 width=18)
  ->  Seq Scan on test1  (cost=0.00..3274.00 rows=200000 width=18)",,,,,,,,,"psql"

```



​	

### 数据库使用



​	vi $PGDATA/postgresql.conf

​	shared_preload_libraries = 'auto_explain' #shared_preload_libraries = 'pg_stat_statments,auto_explain' # 在本次配置并没有安装pg_stat_statments文件，所以添加它会报错
	auto_explain.log_min_duration = 100ms   

​	

​	修改后需要重启数据库



