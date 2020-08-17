[TOC]

# PG PAGEINSPECT 探索数据页信息



​	上一章节中主要讲述了pageinspect的主要函数，现在就学以致用，看看postgres数据页到底是怎样一个分部



​	先来贴一张图片

![_](../img_src/000/2018-08-08_193430.png)

​	



##  page header

​	

​	从上图中可以看出page header在数据页的头部，那么它的结构又是什么？

​	测试往新表插入一条数据

```
tutorial=# create table test1(id int);
CREATE TABLE
tutorial=# insert into test1 values(1);
INSERT 0 1
```

​	查看pageheader信息

```
tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/416997B8 |        0 |     0 |    28 |  8160 |    8192 |     8192 |       4 |         0
(1 row)
```

​	那么列lsn,checksum,flags,lower,upper,special,pagesize,version,prune_xid字段的含义是什么意思

​	将参考信息贴出来



```
/*
 * disk page organization
 *
 * space management information generic to any page
 *
 *		pd_lsn		- identifies xlog record for last change to this page.
 *		pd_checksum - page checksum, if set.
 *		pd_flags	- flag bits.
 *		pd_lower	- offset to start of free space.
 *		pd_upper	- offset to end of free space.
 *		pd_special	- offset to start of special space.
 *		pd_pagesize_version - size in bytes and page layout version number.
 *		pd_prune_xid - oldest XID among potentially prunable tuples on page.
 *
 * The LSN is used by the buffer manager to enforce the basic rule of WAL:
 * "thou shalt write xlog before data".  A dirty buffer cannot be dumped
 * to disk until xlog has been flushed at least as far as the page's LSN.
 *
 * pd_checksum stores the page checksum, if it has been set for this page;
 * zero is a valid value for a checksum. If a checksum is not in use then
 * we leave the field unset. This will typically mean the field is zero
 * though non-zero values may also be present if databases have been
 * pg_upgraded from releases prior to 9.3, when the same byte offset was
 * used to store the current timelineid when the page was last updated.
 * Note that there is no indication on a page as to whether the checksum
 * is valid or not, a deliberate design choice which avoids the problem
 * of relying on the page contents to decide whether to verify it. Hence
 * there are no flag bits relating to checksums.
 *
 * pd_prune_xid is a hint field that helps determine whether pruning will be
 * useful.  It is currently unused in index pages.
 *
 * The page version number and page size are packed together into a single
 * uint16 field.  This is for historical reasons: before PostgreSQL 7.3,
 * there was no concept of a page version number, and doing it this way
 * lets us pretend that pre-7.3 databases have page version number zero.
 * We constrain page sizes to be multiples of 256, leaving the low eight
 * bits available for a version number.
 *
 * Minimum possible page size is perhaps 64B to fit page header, opaque space
 * and a minimal tuple; of course, in reality you want it much bigger, so
 * the constraint on pagesize mod 256 is not an important restriction.
 * On the high end, we can only support pages up to 32KB because lp_off/lp_len
 * are 15 bits.
 */




typedef struct PageHeaderData
{
	/* XXX LSN is member of *any* block, not only page-organized ones */
	PageXLogRecPtr pd_lsn;		/* LSN: next byte after last byte of xlog
								 * record for last change to this page */
	uint16		pd_checksum;	/* checksum */
	uint16		pd_flags;		/* flag bits, see below */
	LocationIndex pd_lower;		/* offset to start of free space */
	LocationIndex pd_upper;		/* offset to end of free space */
	LocationIndex pd_special;	/* offset to start of special space */
	uint16		pd_pagesize_version;
	TransactionId pd_prune_xid; /* oldest prunable XID, or zero if none */
	ItemIdData	pd_linp[1];		/* beginning of line pointer array */
} PageHeaderData;

```





| 字段                | 类型          | 长度  | 描述                                                    |
| ------------------- | ------------- | ----- | ------------------------------------------------------- |
| pd_lsn              | XLogRecPtr    | 8字节 | LSN: 该页上最后的变化对应的xlog记录的最后字节的下一字节 |
| pd_checksum         | uint16        | 2字节 | 页面校验和                                              |
| pd_flags            | uint16        | 2字节 | 标志位                                                  |
| pd_lower            | LocationIndex | 2字节 | 到空闲空间开始处的偏移量                                |
| pd_upper            | LocationIndex | 2字节 | 到空闲空间结尾处的偏移量                                |
| pd_special          | LocationIndex | 2字节 | 到专用空间开始处的偏移量                                |
| pd_pagesize_version | uint16        | 2字节 | 页大小和布局版本号信息                                  |
| pd_prune_xid        | TransactionId | 4字节 | 页上最旧的未修整的XMAX，如果没有则为零。                |



### pg_lsn

​	The LSN is used by the buffer manager to enforce the basic rule of WAL:"thou shalt write xlog before data".  A dirty buffer cannot be dumped to disk until xlog has been flushed at least as far as the page's LSN.

```
缓存管理器通过lsn来wal基础规则生效：在写入数据之前需要写入到xlog文件中。脏数据不能被转储到磁盘中除非xlog已经将页面的lsn刷新进去
```

函数:pg_xlogfile_name定位lsn所在的xlog日志

```
tutorial=#  select pg_xlogfile_name('0/416997B8');
     pg_xlogfile_name     
--------------------------
 000000010000000000000041
(1 row)
```

检查当前的xlog日志

```
[osdba@mysql45 pg_xlog]$ ls -lt
total 131076
-rw------- 1 osdba osdba 16777216 Aug  8 11:41 000000010000000000000041
-rw------- 1 osdba osdba 16777216 Aug  8 04:50 000000010000000000000048
-rw------- 1 osdba osdba 16777216 Aug  8 04:50 000000010000000000000046
-rw------- 1 osdba osdba 16777216 Aug  8 04:50 000000010000000000000047
-rw------- 1 osdba osdba 16777216 Aug  8 04:50 000000010000000000000043
-rw------- 1 osdba osdba 16777216 Aug  8 04:50 000000010000000000000045
-rw------- 1 osdba osdba 16777216 Aug  8 04:50 000000010000000000000044
-rw------- 1 osdba osdba 16777216 Aug  8 04:50 000000010000000000000042
drwx------ 2 osdba osdba     4096 Jul 30 02:22 archive_status
```



### pd_checksum



pd_checksum stores the page checksum, if it has been set for this page; zero is a valid value for a checksum. If a checksum is not in use then we leave the field unset. This will typically mean the field is zero though non-zero values may also be present if databases have been pg_upgraded from releases prior to 9.3, when the same byte offset was used to store the current timelineid when the page was last updated. Note that there is no indication on a page as to whether the checksum is valid or not, a deliberate design choice which avoids the problem of relying on the page contents to decide whether to verify it. Hence there are no flag bits relating to checksums.

```
pd_checksum存储页面校验和，如果已经为该页设置了校验和;0是校验和的有效值。如果校验和没有被使用，那么我们就不设置字段。这通常意味着字段为0，但如果数据库在9.3之前的版本中升级了pg_upgrade(当页面上一次更新时使用相同的字节偏移量来存储当前的timelineid)，则可能会出现非零值。注意，在页面上没有显示校验和是否有效，这是一个经过深思熟虑的设计选择，它避免了依赖页面内容来决定是否验证校验的问题。因此，没有与校验和相关的标记位。
```

​	说实话没有看懂到底是什么意思？

### pd_flags

```
/*
 * pd_flags contains the following flag bits.  Undefined bits are initialized
 * to zero and may be used in the future.
 *
 * PD_HAS_FREE_LINES is set if there are any LP_UNUSED line pointers before
 * pd_lower.  This should be considered a hint rather than the truth, since
 * changes to it are not WAL-logged.
 *
 * PD_PAGE_FULL is set if an UPDATE doesn't find enough free space in the
 * page for its new tuple version; this suggests that a prune is needed.
 * Again, this is just a hint.
 */
#define PD_HAS_FREE_LINES	0x0001		/* are there any unused line pointers? */
#define PD_PAGE_FULL		0x0002		/* not enough free space for new
										 * tuple? */
#define PD_ALL_VISIBLE		0x0004		/* all tuples on page are visible to
										 * everyone */

#define PD_VALID_FLAG_BITS	0x0007		/* OR of all valid pd_flags bits */
```

​	说实话没有看懂到底是什么意思？



### pd_lower pd_upper

​	上面的位移，下面的位移



### pd_special

​	特殊标记，不需要了解



### pd_pagesize_version

​	pd_pagesize_version， 存储页大小和版本指示符。 从PostgreSQL 8.3开始版本编号是4；  PostgreSQL 8.1和8.2使用版本编号3； PostgreSQL 8.0使用版本编号2； PostgreSQL 7.3和7.4使用版本编号1； 先前发布版本使用版本编号0 



### pd_prune_xid



​	pd_prune_xid:当页面有数据被更新或者删除时，记录事务的xid,当数据被vacuum后，重新变为0，如果在页面中数据多次delete掉，prune_id只保留最小的xid

```
tutorial=# truncate table test1;
TRUNCATE TABLE
tutorial=# insert into test1 values(1);
INSERT 0 1
tutorial=# insert into test1 values(3);
INSERT 0 1
tutorial=# insert into test1 values(2);
INSERT 0 1
tutorial=# begin;
BEGIN
tutorial=# select txid_current();
 txid_current 
--------------
         2239
(1 row)

tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/416A9B78 |        0 |     0 |    36 |  8096 |    8192 |     8192 |       4 |         0
(1 row)

tutorial=# delete from test1 where id = 1;
DELETE 1
tutorial=# select txid_current();
 txid_current 
--------------
         2239
(1 row)

tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/416A9BE8 |        0 |     0 |    36 |  8096 |    8192 |     8192 |       4 |      2239
(1 row)

tutorial=# delete from test1 where id = 2;
DELETE 1
tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/416A9C28 |        0 |     0 |    36 |  8096 |    8192 |     8192 |       4 |      2239
(1 row)

tutorial=# select txid_current();
 txid_current 
--------------
         2239
(1 row)

tutorial=# end;
COMMIT
tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/416A9C28 |        0 |     0 |    36 |  8096 |    8192 |     8192 |       4 |      2239
(1 row)


```



## heap_page_items

heap_page_items(page bytea) returns setof record

```
tutorial=# truncate table test1;
TRUNCATE TABLE
tutorial=# insert into test1 values(1);
INSERT 0 1
tutorial=# insert into test1 values(2);
INSERT 0 1
tutorial=# insert into test1 values(3);
INSERT 0 1
tutorial=# select * from page_header(get_raw_page('test1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 0/416B3A20 |        0 |     0 |    36 |  8096 |    8192 |     8192 |       4 |         0
(1 row)

tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2255 |      0 |        0 | (0,1)  |           1 |       2048 |     24 |        |      
  2 |   8128 |        1 |     28 |   2256 |      0 |        0 | (0,2)  |           1 |       2048 |     24 |        |      
  3 |   8096 |        1 |     28 |   2257 |      0 |        0 | (0,3)  |           1 |       2048 |     24 |        |      
(3 rows)


```

​	那么列lp,lp_off,lp_flags,lp_len,t_xmin,t_xmax,t_filelds,t_ctid,t_infomask2,t_infomask,t_hoff,t_bits,t_oid这些字段名信息又是什么





| 字段        | 类型            | 长度  | 描述                      |
| ----------- | --------------- | ----- | ------------------------- |
| t_xmin      | TransactionId   | 4字节 | 插入XID戳                 |
| t_xmax      | TransactionId   | 4字节 | 删除XID戳                 |
| t_ctid      | ItemPointerData | 6字节 | 这个当前的或新行版本的TID |
| t_infomask2 | uint16          | 2字节 | 字段个数，以及各种标志位  |
| t_infomask  | uint16          | 2字节 | 各种标志位                |
| t_hoff      | uint8           | 1字节 | 用户数据偏移量            |



### lp

### lp_off

### lp_flags

### t_xmin,t_max

### t_ctid

### t_filelds

### t_infomask2

### t_infomask 

### t_hoff

### t_bits



​	好多字段信息都不是很确定，需要自己好好学习。



### 测试数据DELETE,VACUUM或者vacuum full 对于数据页的影响



​	1、插入几条测试数据

```
tutorial=# truncate table test1;
TRUNCATE TABLE
tutorial=# insert into test1 select generate_series(1,5);
INSERT 0 5
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2261 |      0 |        0 | (0,1)  |           1 |       2048 |     24 |        |      
  2 |   8128 |        1 |     28 |   2261 |      0 |        0 | (0,2)  |           1 |       2048 |     24 |        |      
  3 |   8096 |        1 |     28 |   2261 |      0 |        0 | (0,3)  |           1 |       2048 |     24 |        |      
  4 |   8064 |        1 |     28 |   2261 |      0 |        0 | (0,4)  |           1 |       2048 |     24 |        |      
  5 |   8032 |        1 |     28 |   2261 |      0 |        0 | (0,5)  |           1 |       2048 |     24 |        |      
(5 rows)


```

​	

​	2、删除值为2的数据

```
tutorial=# delete from test1 where id = 2;
DELETE 1
tutorial=# select * from test1 where id = 2;
 id 
----
(0 rows)
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2261 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   2261 |   2262 |        0 | (0,2)  |        8193 |        256 |     24 |        |      
  3 |   8096 |        1 |     28 |   2261 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   2261 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8032 |        1 |     28 |   2261 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
(5 rows)

```

​	首先在表中已经查询不到ctid=(0,2),或者id=2的数据了，可是在items中还是找到了ctid(0,2)的信息；而且数据的空间并没有释放

​	3、插入新数据

```
tutorial=# insert into test1 select generate_series(6,10);
INSERT 0 5
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2261 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   2261 |   2262 |        0 | (0,2)  |        8193 |       1280 |     24 |        |      
  3 |   8096 |        1 |     28 |   2261 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   2261 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8032 |        1 |     28 |   2261 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8000 |        1 |     28 |   2263 |      0 |        0 | (0,6)  |           1 |       2048 |     24 |        |      
  7 |   7968 |        1 |     28 |   2263 |      0 |        0 | (0,7)  |           1 |       2048 |     24 |        |      
  8 |   7936 |        1 |     28 |   2263 |      0 |        0 | (0,8)  |           1 |       2048 |     24 |        |      
  9 |   7904 |        1 |     28 |   2263 |      0 |        0 | (0,9)  |           1 |       2048 |     24 |        |      
 10 |   7872 |        1 |     28 |   2263 |      0 |        0 | (0,10) |           1 |       2048 |     24 |        |      
(10 rows)

```

​	插入新数据，发现数据是从lp_off的8000出开始往下减少，对于数据空间来说，直接delete数据，并不会释放空间。

​	

​	4、vacuum table

​	执行vacuum table,可以将delete的空间释放出来(同时数据恢复将变得更加困难)

```
tutorial=# vacuum test1;
VACUUM
tutorial=# select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2261 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  3 |   8128 |        1 |     28 |   2261 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8096 |        1 |     28 |   2261 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8064 |        1 |     28 |   2261 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8032 |        1 |     28 |   2263 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   8000 |        1 |     28 |   2263 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7968 |        1 |     28 |   2263 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7936 |        1 |     28 |   2263 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7904 |        1 |     28 |   2263 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)

```

​	数据在ctid=(0,2)的值彻底不见了，而且仔细观察lp=3的数据位移发生变化，它的位置替换到了lp=2时之前的位置(vacuum前)

​	

​	其实vacuum来说，迁移行内数据变化



​       5、插入一条数据

```
tutorial=# insert into test1 select 100;
INSERT 0 1
tutorial=#  select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2292 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   7872 |        1 |     28 |   2295 |      0 |        0 | (0,2)  |           1 |       2048 |     24 |        |      
  3 |   8128 |        1 |     28 |   2292 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8096 |        1 |     28 |   2292 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8064 |        1 |     28 |   2292 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8032 |        1 |     28 |   2294 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   8000 |        1 |     28 |   2294 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7968 |        1 |     28 |   2294 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7936 |        1 |     28 |   2294 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7904 |        1 |     28 |   2294 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)

```

​	发现新的数据插入(0,2)的位置，但是它的位移地址确实最小的。



​	6、执行vacuum full table

```
tutorial=# vacuum full test1;
VACUUM
tutorial=#  select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2292 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   2295 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   2292 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   2292 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   2292 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   2294 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   2294 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   2294 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   2294 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
 10 |   7872 |        1 |     28 |   2294 |      0 |        0 | (0,10) |           1 |       2816 |     24 |        |      
(10 rows)

tutorial=# select ctid,* from test1;
  ctid  | id  
--------+-----
 (0,1)  |   1
 (0,2)  | 100
 (0,3)  |   3
 (0,4)  |   4
 (0,5)  |   5
 (0,6)  |   6
 (0,7)  |   7
 (0,8)  |   8
 (0,9)  |   9
 (0,10) |  10
(10 rows)

```



​	发现数据是按照ctid重新去整理数据，而ctid=(0,2)的数据的位移只是比（0,1）的小了一点，说明vacuum full table是对整个的page页进行整理，io相对vacuum要大。





​	

### 测试数据UPDATE,VACUUM或者vacuum full 对于数据页的影响

```
tutorial=# truncate table test1;
TRUNCATE TABLE
tutorial=# insert into test1 select generate_series(1,5);
INSERT 0 5
tutorial=#  select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask
 | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+-----------
-+--------+--------+-------
  1 |   8160 |        1 |     28 |   2298 |      0 |        0 | (0,1)  |           1 |       2048
 |     24 |        |      
  2 |   8128 |        1 |     28 |   2298 |      0 |        0 | (0,2)  |           1 |       2048
 |     24 |        |      
  3 |   8096 |        1 |     28 |   2298 |      0 |        0 | (0,3)  |           1 |       2048
 |     24 |        |      
  4 |   8064 |        1 |     28 |   2298 |      0 |        0 | (0,4)  |           1 |       2048
 |     24 |        |      
  5 |   8032 |        1 |     28 |   2298 |      0 |        0 | (0,5)  |           1 |       2048
 |     24 |        |      
(5 rows)

tutorial=# update test1 set id = 10 where id = 1;
UPDATE 1
tutorial=#  select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2298 |   2299 |        0 | (0,6)  |       16385 |        256 |     24 |        |      
  2 |   8128 |        1 |     28 |   2298 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   2298 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   2298 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8032 |        1 |     28 |   2298 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8000 |        1 |     28 |   2299 |      0 |        0 | (0,6)  |       32769 |      10240 |     24 |        |      
(6 rows)

tutorial=# vacuum test1;
VACUUM
tutorial=#  select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |      6 |        2 |      0 |        |        |          |        |             |            |        |        |      
  2 |   8160 |        1 |     28 |   2298 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8128 |        1 |     28 |   2298 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8096 |        1 |     28 |   2298 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8064 |        1 |     28 |   2298 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8032 |        1 |     28 |   2299 |      0 |        0 | (0,6)  |       32769 |      10496 |     24 |        |      
(6 rows)

tutorial=# vacuum full test1;
VACUUM
tutorial=#  select * from heap_page_items(get_raw_page('test1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   2298 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   2298 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   2298 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   2298 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   2299 |      0 |        0 | (0,5)  |           1 |      11008 |     24 |        |      
(5 rows)

tutorial=# 

```

​	首先delete数据不会插入一条数据，而update会插入一条新数据

​	其次update其实就是delete+insert数据的结合体，因此会出现两条数据的变化情况，删除数据的tmax就是插入数据的tmin

​	第三update是，items会在删除数据的上记录插入数据的ctid(hot)

​	第四当数据被vacuum时，数据会在第一行保留最新的数据的位置信息(lp_off)

​	第五数据被vacuum full这是就要对数据重新insert处理了







## 链接



https://www.postgresql.org/docs/current/static/pageinspect.html?spm=a2c4e.11153940.blogcont2291.5.36292b2bYnZkIy