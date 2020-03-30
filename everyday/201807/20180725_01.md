

[TOC]

# PG TOAST

​	

​	TOAST是“The Oversized-Attribute Storage  Technique”的缩写，主要用于存储一个大字段的值。在PG中，页是数据在文件存储中的基本单位，其大小是固定的且只能在编译期指定，之后无法修改，默认的大小为8KB。同时，PG不允许一行数据跨页存储，那么对于超长的行数据，PG就会启动TOAST，具体就是采用压缩和切片的方式。如果启用了切片，实际数据存储在另一张系统表的多个行中，这张表就叫TOAST表，这种存储方式叫行外存储。 

​	在深入细节之前，我们要先了解，在PG中每个表字段有四种TOAST的策略：

- PLAIN：避免压缩和行外存储。只有那些不需要TOAST策略就能存放的数据类型允许选择（例如int类型），而对于text这类要求存储长度超过页大小的类型，是不允许采用此策略的

- EXTENDED：允许压缩和行外存储。一般会先压缩，如果还是太大，就会行外存储

- EXTERNAL：允许行外存储，但不许压缩。类似字符串这种会对数据的一部分进行操作的字段，采用此策略可能获得更高的性能，因为不需要读取出整行数据再解压。

- MAIN：允许压缩，但不许行外存储。不过实际上，为了保证过大数据的存储，行外存储在其它方式（例如压缩）都无法满足需求的情况下，作为最后手段还是会被启动。

  因此理解为：尽量不使用行外存储更贴切。 现在我们通过实际操作来研究TOAST的细节



​	

##  variable-length type

​	只有变长类型的字段类型才会触发到toast表,常态下，int,boolean,date都不会触发toast表

```
utorial=# create table test1(id int);
CREATE TABLE
tutorial=# insert into test1 values(1);
INSERT 0 1
tutorial=# select pg_column_size(id) from test1;
 pg_column_size 
----------------
              4
(1 row)

tutorial=# \d+ test1;
                        Table "public.test1"
 Column |  Type   | Modifiers | Storage | Stats target | Description 
--------+---------+-----------+---------+--------------+-------------
 id     | integer |           | plain   |              | 

tutorial=# drop table test1;
DROP TABLE
tutorial=# create table test1(id int,name varchar(100),info text);
CREATE TABLE
tutorial=# \d+ test1;
                                Table "public.test1"
 Column |          Type          | Modifiers | Storage  | Stats target | Description 
--------+------------------------+-----------+----------+--------------+-------------
 id     | integer                |           | plain    |              | 
 name   | character varying(100) |           | extended |              | 
 info   | text                   |           | extended |              | 

tutorial=# select oid,relname,reltoastrelid from pg_class where relname = 'test1';
  oid  | relname | reltoastrelid 
-------+---------+---------------
 81704 | test1   |         81707
(1 row)

```

​	

## 修改 默认存储方式

​	alter table  alter set storage

```
tutorial=# alter table test1 alter name set storage external;
ALTER TABLE
tutorial=# \d+ test1;
                                Table "public.test1"
 Column |          Type          | Modifiers | Storage  | Stats target | Description 
--------+------------------------+-----------+----------+--------------+-------------
 id     | integer                |           | plain    |              | 
 name   | character varying(100) |           | external |              | 
 info   | text                   |           | extended |              | 
```

​	数据库为每种字段类型都有默认的存储方式，修改后只是优先考虑当前修改的存储方式，当某个字段存储方式改为`main`,如果是在压缩不下去，只能使用`extended`方式



## EXTENDED

​	当字段长度接近2kb时，首先是压缩，后面就是行外存储

```
tutorial=# insert into test1 values(1,'guo h','0123456789qwertyuiopasdfghjklzxcvbnm');
INSERT 0 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
             37
(1 row)

tutorial=# select pg_relation_size(81704);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# select pg_relation_size(81707);
 pg_relation_size 
------------------
                0
(1 row)



```



​	经过多次UPDATE后,发现空间变小，而pg_toast却并没有发生变化，说明当前应该是经过压缩了

```


tutorial=# update test1 set info = info||info;
UPDATE 1


tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
             77
(1 row)

tutorial=# select pg_relation_size(81707);
 pg_relation_size 
------------------
                0
(1 row)
```

​	经过多次UPDATE后，发现所占字节数一直在变大，但是数据页没有变化，而且toast表也有了空间,说明数据已经是行外存储了。

```
tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
            471
(1 row)

tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
           1736
(1 row)

tutorial=# select pg_relation_size(81704);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# select pg_relation_size(81707);
 pg_relation_size 
------------------
                0
(1 row)

tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_relation_size(81704);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# select pg_relation_size(81707);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
           3423
(1 row)

tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
           6798
(1 row)

tutorial=# select pg_relation_size(81707);
 pg_relation_size 
------------------
            16384
(1 row)

tutorial=# select pg_relation_size(81704);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# 

```



## external





```
tutorial=# drop table test1;
DROP TABLE
tutorial=# create table test1(id int,info text);
CREATE TABLE
tutorial=# \d+ test1;
                         Table "public.test1"
 Column |  Type   | Modifiers | Storage  | Stats target | Description 
--------+---------+-----------+----------+--------------+-------------
 id     | integer |           | plain    |              | 
 info   | text    |           | extended |              | 

tutorial=# select oid,relname,reltoastrelid from pg_class where relname ='test1';
  oid  | relname | reltoastrelid 
-------+---------+---------------
 81715 | test1   |         81718
(1 row)
tutorial=# alter table test1 alter info set storage external;
ALTER TABLE
tutorial=# \d+ test1;
                         Table "public.test1"
 Column |  Type   | Modifiers | Storage  | Stats target | Description 
--------+---------+-----------+----------+--------------+-------------
 id     | integer |           | plain    |              | 
 info   | text    |           | external |              | 

tutorial=# insert into test1 values(1,
tutorial(# '111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111');
INSERT 0 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
            112
(1 row)

tutorial=# select pg_relation_size(81715);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# select pg_relation_size(81718);
 pg_relation_size 
------------------
                0
(1 row)

tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
            226
(1 row)

tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
            226
(1 row)

tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
            892
(1 row)

tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
           1780
(1 row)

tutorial=# select pg_relation_size(81718);
 pg_relation_size 
------------------
                0
(1 row)

tutorial=# select pg_relation_size(81715);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# update test1 set info = info||info;
UPDATE 1
tutorial=# select pg_column_size(info) from test1;
 pg_column_size 
----------------
           3552
(1 row)

tutorial=# select pg_relation_size(81715);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# select pg_relation_size(81718);
 pg_relation_size 
------------------
             8192
(1 row)

tutorial=# select count(1) from pg_toast.pg_toast_81715;
 count 
-------
     2
(1 row)


```

因为external没有压缩这个策略，导致数据接近2kb，就开始将数据迁移到行外存储pg_toast表中，而默认的extended既有压缩，又有行外存储，就是当数据块接近2kb,首先进行压缩，当数据块再次接近2kb时，开始行外存储。



## MAIN

​	猜测是压缩后，空间如果还是不够用，将会使用行外存储，迁移出来。







## pg_toast 查询

​	查询toast表名其实就是pg_toast_oid(oid指的时原表的oid)

```
tutorial=# \d pg_toast.pg_toast_81704;
TOAST table "pg_toast.pg_toast_81704"
   Column   |  Type   
------------+---------
 chunk_id   | oid
 chunk_seq  | integer
 chunk_data | bytea

tutorial=# select chunk_id,chunk_seq from pg_toast.pg_toast_81704;
 chunk_id | chunk_seq 
----------+-----------
    81711 |         0
    81711 |         1
    81711 |         2
    81711 |         3
(4 rows)

```











摘自官网说明

```
This section provides an overview of TOAST (The Oversized-Attribute Storage Technique).

PostgreSQL uses a fixed page size (commonly 8 kB), and does not allow tuples to span multiple pages. Therefore, it is not possible to store very large field values directly. To overcome this limitation, large field values are compressed and/or broken up into multiple physical rows. This happens transparently to the user, with only small impact on most of the backend code. The technique is affectionately known as TOAST (or "the best thing since sliced bread").

Only certain data types support TOAST — there is no need to impose the overhead on data types that cannot produce large field values. To support TOAST, a data type must have a variable-length (varlena) representation, in which the first 32-bit word of any stored value contains the total length of the value in bytes (including itself). TOAST does not constrain the rest of the representation. All the C-level functions supporting a TOAST-able data type must be careful to handle TOASTed input values. (This is normally done by invoking PG_DETOAST_DATUM before doing anything with an input value, but in some cases more efficient approaches are possible.)

TOAST usurps two bits of the varlena length word (the high-order bits on big-endian machines, the low-order bits on little-endian machines), thereby limiting the logical size of any value of a TOAST-able data type to 1 GB (230 - 1 bytes). When both bits are zero, the value is an ordinary un-TOASTed value of the data type, and the remaining bits of the length word give the total datum size (including length word) in bytes. When the highest-order or lowest-order bit is set, the value has only a single-byte header instead of the normal four-byte header, and the remaining bits give the total datum size (including length byte) in bytes. As a special case, if the remaining bits are all zero (which would be impossible for a self-inclusive length), the value is a pointer to out-of-line data stored in a separate TOAST table. (The size of a TOAST pointer is given in the second byte of the datum.) Values with single-byte headers aren't aligned on any particular boundary, either. Lastly, when the highest-order or lowest-order bit is clear but the adjacent bit is set, the content of the datum has been compressed and must be decompressed before use. In this case the remaining bits of the length word give the total size of the compressed datum, not the original data. Note that compression is also possible for out-of-line data but the varlena header does not tell whether it has occurred — the content of the TOAST pointer tells that, instead.

If any of the columns of a table are TOAST-able, the table will have an associated TOAST table, whose OID is stored in the table's pg_class.reltoastrelid entry. Out-of-line TOASTed values are kept in the TOAST table, as described in more detail below.

The compression technique used is a fairly simple and very fast member of the LZ family of compression techniques. See src/backend/utils/adt/pg_lzcompress.c for the details.

Out-of-line values are divided (after compression if used) into chunks of at most TOAST_MAX_CHUNK_SIZE bytes (by default this value is chosen so that four chunk rows will fit on a page, making it about 2000 bytes). Each chunk is stored as a separate row in the TOAST table for the owning table. Every TOAST table has the columns chunk_id (an OID identifying the particular TOASTed value), chunk_seq (a sequence number for the chunk within its value), and chunk_data (the actual data of the chunk). A unique index on chunk_id and chunk_seq provides fast retrieval of the values. A pointer datum representing an out-of-line TOASTed value therefore needs to store the OID of the TOAST table in which to look and the OID of the specific value (its chunk_id). For convenience, pointer datums also store the logical datum size (original uncompressed data length) and actual stored size (different if compression was applied). Allowing for the varlena header bytes, the total size of a TOAST pointer datum is therefore 18 bytes regardless of the actual size of the represented value.

The TOAST code is triggered only when a row value to be stored in a table is wider than TOAST_TUPLE_THRESHOLD bytes (normally 2 kB). The TOAST code will compress and/or move field values out-of-line until the row value is shorter than TOAST_TUPLE_TARGET bytes (also normally 2 kB) or no more gains can be had. During an UPDATE operation, values of unchanged fields are normally preserved as-is; so an UPDATE of a row with out-of-line values incurs no TOAST costs if none of the out-of-line values change.

The TOAST code recognizes four different strategies for storing TOAST-able columns:

    PLAIN prevents either compression or out-of-line storage; furthermore it disables use of single-byte headers for varlena types. This is the only possible strategy for columns of non-TOAST-able data types.

    EXTENDED allows both compression and out-of-line storage. This is the default for most TOAST-able data types. Compression will be attempted first, then out-of-line storage if the row is still too big.

    EXTERNAL allows out-of-line storage but not compression. Use of EXTERNAL will make substring operations on wide text and bytea columns faster (at the penalty of increased storage space) because these operations are optimized to fetch only the required parts of the out-of-line value when it is not compressed.

    MAIN allows compression but not out-of-line storage. (Actually, out-of-line storage will still be performed for such columns, but only as a last resort when there is no other way to make the row small enough to fit on a page.)

Each TOAST-able data type specifies a default strategy for columns of that data type, but the strategy for a given table column can be altered with ALTER TABLE SET STORAGE.

This scheme has a number of advantages compared to a more straightforward approach such as allowing row values to span pages. Assuming that queries are usually qualified by comparisons against relatively small key values, most of the work of the executor will be done using the main row entry. The big values of TOASTed attributes will only be pulled out (if selected at all) at the time the result set is sent to the client. Thus, the main table is much smaller and more of its rows fit in the shared buffer cache than would be the case without any out-of-line storage. Sort sets shrink also, and sorts will more often be done entirely in memory. A little test showed that a table containing typical HTML pages and their URLs was stored in about half of the raw data size including the TOAST table, and that the main table contained only about 10% of the entire data (the URLs and some small HTML pages). There was no run time difference compared to an un-TOASTed comparison table, in which all the HTML pages were cut down to 7 kB to fit.
```







## 链接地址

https://www.postgresql.org/docs/9.4/static/storage-toast.html

https://www.cnblogs.com/qcloud1001/p/6641088.html

https://my.oschina.net/Kenyon/blog/113026