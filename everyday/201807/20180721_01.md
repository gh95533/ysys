[TOC]

# PG pageinspect

​	在学习数据页和数据行是发现pg提供一些函数能够更具体查看相关信息，就百度了一些文档，参考文档和视频来学习这方面知识



## 安装

​	1、检查函数是否存在

```
osdba=# create extension pageinspect
```

​	如果提示缺少文件，则说明还没有安装，或者提示已经安装过。

​	2、首先编译数据库时尽量使用源码安装并且安装地址保持不变

```
cd pageinspcet
make
make install
```

​	3、创建extension

```
create extension pageinspect
```



## 函数



### get_raw_page(relname text, fork text, blkno int) returns bytea

函数：`get_raw_page(relname text, fork text, blkno int) returns bytea` 

`get_raw_page` reads the specified block of the named relation and returns a copy as a `bytea` value. This allows a single time-consistent copy of the block to be obtained. *fork* should be `'main'` for the main data fork, `'fsm'` for the free space map, `'vm'` for the visibility map, or `'init'` for the initialization  

```
get_raw_page读取命名关系指定的块并返回一个拷贝作为bytea值。 这允许获得一个该块的时间一致的拷贝。*fork*应该是主数据分支的 'main'，自由空间映射的'fsm'， 可见映射的'vm'，或初始fork的'init'。 
```

函数：`get_raw_page(relname text, blkno int) returns bytea`

A shorthand version of `get_raw_page`, for reading from the main fork. Equivalent to `get_raw_page(relname, 'main', blkno)`

```
 get_raw_page的短版本，从主分支上读取。 等于get_raw_page(relname, 'main', blkno)
```



当然直接执行函数get_raw_page(relname,0），默认返回是main的byeta

```
osdba=# set bytea_output='hex';
SET
osdba=# select * from get_raw_page('test_1',0);

\x03000000c0580451000000001c00e01f0020042000000000e09f38000000000000000000000000000000000000000000000000
000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000006c1b00000000000000000000000000000100010000081800010000000
0000000

osdba=# set bytea_output='escape';
SET
osdba=# select * from get_raw_page('test_1',0);
 \003\000\000\000\300X\004Q\000\000\000\000\034\000\340\037\000 \004 0\000\000\000\000\000\000\000\000\000\000\000\000\000\00
\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\000\001\000\001\000\000\010\030\000\001\000\000\000\000\000\000\000
(1 row)


```

​	看不懂上面具体的意思，需要其他函数来解析信息



### page_header(page bytea) returns record

函数：`page_header(page bytea) returns record` 

`page_header` shows fields that are common to all PostgreSQL heap and index pages.

```
page_header显示了所有PostgreSQL 堆和索引页共通的字段
```



A page image obtained with `get_raw_page` should be passed as argument. For example:

```
get_raw_page获得的页面镜像应该作为一个参数传送。例如： 
```

```
test=# SELECT * FROM page_header(get_raw_page('pg_class', 0));
    lsn    | checksum | flags  | lower | upper | special | pagesize | version | prune_xid
-----------+----------+--------+-------+-------+---------+----------+---------+-----------
 0/24A1B50 |        0 |      1 |   232 |   368 |    8192 |     8192 |       4 |         0
```

The returned columns correspond to the fields in the `PageHeaderData` struct. See `src/include/storage/bufpage.h` for details. 

```
返回的列符合PageHeaderData结构中的字段。 参阅src/include/storage/bufpage.h获取详细信息。 
```

[bugpage.h](../20180807/postgresql_bufpage.h.md)



### heap_page_items(page bytea) returns setof record

函数：`heap_page_items(page bytea) returns setof record` 



`heap_page_items` shows all line pointers on a heap page. For those line pointers that are in use, tuple headers are also shown. All tuples are shown, whether or not the tuples were visible to an MVCC snapshot at the time the raw page was copied. 

```
heap_page_items显示了一个堆页面中所有的行指针。 对于那些还在使用中的行指针，也显示元组头。所有的元组都被显示， 不管在拷贝裸页面时这些元组对MVCC快照是否可见。
```

A heap page image obtained with `get_raw_page` should be passed as argument. For example: 

```
get_raw_page获得的堆页面镜像应该作为一个参数传递。例如：
```

```
test=# SELECT * FROM heap_page_items(get_raw_page('pg_class', 0));
```

See `src/include/storage/itemid.h` and `src/include/access/htup_details.h` for explanations of the fields returned. 

```
参阅src/include/storage/itemid.h和src/include/access/htup_details.h 获取返回的字段的解释。
```

[itemid.h](../20180807/pg_itemid.h.md)

[htup_details.h](../20180807/pg_htup_details.h.md)



### bt_metap(relname text) returns record

函数：`bt_metap(relname text) returns record` 

`bt_metap` returns information about a B-tree index's metapage. For example: 

```
bt_metap返回关于B-tree索引的元数据页的信息。例如：
```

```
test=# SELECT * FROM bt_metap('pg_cast_oid_index');
-[ RECORD 1 ]-----
magic     | 340322
version   | 2
root      | 1
level     | 0
fastroot  | 1
fastlevel | 0
```



### bt_page_stats(...) returns record

函数：`bt_page_stats(relname text, blkno int) returns record` 

```
bt_page_stats返回关于B-tree索引的单个页面的摘要信息。例如：
```

```
test=# SELECT * FROM bt_page_stats('pg_cast_oid_index', 1);
-[ RECORD 1 ]-+-----
blkno         | 1
type          | l
live_items    | 256
dead_items    | 0
avg_item_size | 12
page_size     | 8192
free_size     | 4056
btpo_prev     | 0
btpo_next     | 0
btpo          | 0
btpo_flags    | 3
```





### bt_page_items(...) returns setof record

函数：`bt_page_items(relname text, blkno int) returns setof record` 

`bt_page_items` returns detailed information about all of the items on a B-tree index page. For example: 

```
bt_page_items返回关于B-tree索引页中的所有条目的详细信息。例如： 
```

```
test=# SELECT * FROM bt_page_items('pg_cast_oid_index', 1);
 itemoffset |  ctid   | itemlen | nulls | vars |    data
------------+---------+---------+-------+------+-------------
          1 | (0,1)   |      12 | f     | f    | 23 27 00 00
          2 | (0,2)   |      12 | f     | f    | 24 27 00 00
          3 | (0,3)   |      12 | f     | f    | 25 27 00 00
          4 | (0,4)   |      12 | f     | f    | 26 27 00 00
          5 | (0,5)   |      12 | f     | f    | 27 27 00 00
          6 | (0,6)   |      12 | f     | f    | 28 27 00 00
          7 | (0,7)   |      12 | f     | f    | 29 27 00 00
          8 | (0,8)   |      12 | f     | f    | 2a 27 00 00
```

### fsm_page_contents(page bytea) returns text

函数:`fsm_page_contents(page bytea) returns text` 

`fsm_page_contents` shows the internal node structure of a FSM page. The output is a multiline string, with one line per node in the binary tree within the page. Only those nodes that are not zero are printed. The so-called "next" pointer, which points to the next slot to be returned from the page, is also printed. 

```
fsm_page_contents显示了FSM页的内部节点结构。输出是多行字符串， 在该页中的二叉树上每个节点一行。这些节点中只有不是零的才输出。也输出被称为"next"的指针， 这个指针指向下一个从页面中返回的位置。
```









## 链接地址

https://yq.aliyun.com/articles/2291

https://www.postgresql.org/docs/current/static/pageinspect.html?spm=a2c4e.11153940.blogcont2291.5.36292b2bYnZkIy

http://blog.163.com/digoal@126/blog/static/16387704020114273265960/



