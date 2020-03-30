[TOC]

# pg 在同一个页面对一条数据做update后，最早出现的数据lp_off变为0

​	更新的字段为索引字段不会支持Hot技术

```
HOT updates only work when the changed columns are not involved in indexes in any way because the indexes pointing the the old tuples need to point to the new version of it as of transaction id. 
```





```
tutorial=# create table test1(id int,info text);
CREATE TABLE
tutorial=# create index idx_test1_id on test1(id);
CREATE INDEX
tutorial=# insert into test1 values(1,'gh');
INSERT 0 1
tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/2BCA3A10 |        0 |     0 |    28 |  8160 |    8192 |     8192 |       4 |         0
(1 row)

tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     31 |   2134 |      0 |        0 | (0,1)  |           2 |       2050 |     24 |        |      
(1 row)

tutorial=# update test1 set info = 'guo';
UPDATE 1
tutorial=# update test1 set info = 'guo';
UPDATE 1
tutorial=# update test1 set info = 'guo';
UPDATE 1
tutorial=# update test1 set info = 'guo';
UPDATE 1
tutorial=# update test1 set info = 'guo';
UPDATE 1
tutorial=# update test1 set info = 'guo';
UPDATE 1
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     31 |   2134 |   2135 |        0 | (0,2)  |       16386 |       1282 |     24 |        |      
  2 |   8128 |        1 |     32 |   2135 |   2136 |        0 | (0,3)  |       49154 |       9474 |     24 |        |      
  3 |   8096 |        1 |     32 |   2136 |   2137 |        0 | (0,4)  |       49154 |       9474 |     24 |        |      
  4 |   8064 |        1 |     32 |   2137 |   2138 |        0 | (0,5)  |       49154 |       9474 |     24 |        |      
  5 |   8032 |        1 |     32 |   2138 |   2139 |        0 | (0,6)  |       49154 |       9474 |     24 |        |      
  6 |   8000 |        1 |     32 |   2139 |   2140 |        0 | (0,7)  |       49154 |       8450 |     24 |        |      
  7 |   7968 |        1 |     32 |   2140 |      0 |        0 | (0,7)  |       32770 |      10242 |     24 |        |      
(7 rows)

tutorial=# vacuum test1;
VACUUM
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      7 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |   8160 |        1 |     32 |   2140 |      0 |        0 | (0,7)  |       32770 |      10498 |     24 |        |      
(7 rows)

tutorial=# update test1 set id = 1 where id = 1;
UPDATE 1
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      7 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |   8128 |        1 |     32 |   2141 |      0 |        0 | (0,2)  |       32770 |      10242 |     24 |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |   8160 |        1 |     32 |   2140 |   2141 |        0 | (0,2)  |       49154 |       8450 |     24 |        |      
(7 rows)

tutorial=# vacuum test1;
VACUUM
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      2 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |   8160 |        1 |     32 |   2141 |      0 |        0 | (0,2)  |       32770 |      10498 |     24 |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)

tutorial=# update test1 set id = 2;
UPDATE 1
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      2 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |   8160 |        1 |     32 |   2141 |   2142 |        0 | (0,3)  |       32770 |       8450 |     24 |        |      
  3 |   8128 |        1 |     32 |   2142 |      0 |        0 | (0,3)  |           2 |      10242 |     24 |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)

tutorial=# vacuum test1;
VACUUM
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  2 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  3 |   8160 |        1 |     32 |   2142 |      0 |        0 | (0,3)  |           2 |      10498 |     24 |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)

```

在最后一步update后，对其做vacuum，发现lp为1的lp_off变为0，可是之前的都是lp为1的lp_off都不是零，这是为什么？



原因很简单，hot技术其实针对是没有索引字段的数据更新

如果取消索引，测试效果就不一样了

```
tutorial=# drop index idx_test1_id;
DROP INDEX
tutorial=# truncate table test1;
TRUNCATE TABLE
tutorial=# insert into test1 values(1,'gh');
INSERT 0 1
tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/2BCB2988 |        0 |     0 |    28 |  8160 |    8192 |     8192 |       4 |         0
(1 row)

tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     31 |   2145 |      0 |        0 | (0,1)  |           2 |       2050 |     24 |        |      
(1 row)

tutorial=# update test1 set info = 'guo';
UPDATE 1
tutorial=#  update test1 set info = 'guo';
UPDATE 1
tutorial=#  update test1 set info = 'guo';
UPDATE 1
tutorial=#  update test1 set info = 'guo';
UPDATE 1
tutorial=#  update test1 set info = 'guo';
UPDATE 1
tutorial=#  update test1 set info = 'guo';
UPDATE 1
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     31 |   2145 |   2146 |        0 | (0,2)  |       16386 |       1282 |     24 |        |      
  2 |   8128 |        1 |     32 |   2146 |   2147 |        0 | (0,3)  |       49154 |       9474 |     24 |        |      
  3 |   8096 |        1 |     32 |   2147 |   2148 |        0 | (0,4)  |       49154 |       9474 |     24 |        |      
  4 |   8064 |        1 |     32 |   2148 |   2149 |        0 | (0,5)  |       49154 |       9474 |     24 |        |      
  5 |   8032 |        1 |     32 |   2149 |   2150 |        0 | (0,6)  |       49154 |       9474 |     24 |        |      
  6 |   8000 |        1 |     32 |   2150 |   2151 |        0 | (0,7)  |       49154 |       8450 |     24 |        |      
  7 |   7968 |        1 |     32 |   2151 |      0 |        0 | (0,7)  |       32770 |      10242 |     24 |        |      
(7 rows)

tutorial=# vacuum test1;
VACUUM
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      7 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |   8160 |        1 |     32 |   2151 |      0 |        0 | (0,7)  |       32770 |      10498 |     24 |        |      
(7 rows)

tutorial=# update test1 set id = 1 where id = 1;
UPDATE 1
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      7 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |   8128 |        1 |     32 |   2152 |      0 |        0 | (0,2)  |       32770 |      10242 |     24 |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |   8160 |        1 |     32 |   2151 |   2152 |        0 | (0,2)  |       49154 |       8450 |     24 |        |      
(7 rows)

tutorial=#  vacuum test1;
VACUUM
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      2 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |   8160 |        1 |     32 |   2152 |      0 |        0 | (0,2)  |       32770 |      10498 |     24 |        |      
  3 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)

tutorial=# update test1 set id = 2;
UPDATE 1
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      2 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |   8160 |        1 |     32 |   2152 |   2153 |        0 | (0,3)  |       49154 |       8450 |     24 |        |      
  3 |   8128 |        1 |     32 |   2153 |      0 |        0 | (0,3)  |       32770 |      10242 |     24 |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)

tutorial=# vacuum test1;
VACUUM
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      3 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  3 |   8160 |        1 |     32 |   2153 |      0 |        0 | (0,3)  |       32770 |      10498 |     24 |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  7 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
(7 rows)

tutorial=# 

```



## 链接地址

[外文资料](https://git.postgresql.org/gitweb/?spm=a2c4e.11153940.blogcont59251.96.79cd7fc3I37D46&p=postgresql.git;a=blob;f=src/backend/access/heap/README.HOT;h=4cf3c3a0d4c2db96a57e73e46fdd7463db439f79;hb=f2dba881a5e13abc957f0e692749f89c9288134d)

