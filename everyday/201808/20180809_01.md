[TOC]

# pg pageinspect 测试









**页整体布局**

| 项             | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| PageHeaderData | 24字节长整型。包含关于页的一般信息，包括空闲空间指针。       |
| ItemIdData     | 指向实际项的（偏移量，长度）数组对。每项4字节。              |
| Free space     | 未分配空间。从这个区域开始分配新项指针，从结尾开始分配新项。 |
| Items          | 实际项本身                                                   |
| Special space  | 索引访问方法专用数据。不同方法存储不同的数据。对于普通表该区域为空。 |





函数：`page_header(page bytea) returns record` 

`page_header` shows fields that are common to all PostgreSQL heap and index pages.

page_header：可以返回一些堆页面或者索引页的共有的字段

```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/500C4ED0 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |         0
(1 row)


```

The returned columns correspond to the fields in the `PageHeaderData` struct. See `src/include/storage/bufpage.h` for details. 

查看一下源码

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







pd_pagesize_version， 存储页大小和版本指示符。 从PostgreSQL 8.3开始版本编号是4；  PostgreSQL 8.1和8.2使用版本编号3； PostgreSQL 8.0使用版本编号2； PostgreSQL 7.3和7.4使用版本编号1； 先前发布版本使用版本编号0 



pd_flags

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

当pd_flags=2时，表示当前没有足够的空间来插入新的行

```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,100000);
INSERT 0 100000
osdba=# select max(ctid) from test_1;
    max    
-----------
 (442,108)
(1 row)

osdba=# select max(ctid) from test_1 where id < 200;
   max   
---------
 (0,199)
(1 row)
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51054950 |        0 |     0 |   928 |   960 |    8192 |     8192 |       4 |         0
(1 row)
osdba=# update test_1 set id = id + 1 where id < 199;
UPDATE 198
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51675C80 |        0 |     2 |   928 |   960 |    8192 |     8192 |       4 |      7027
(1 row)


```

当pd_flags等于4，表示当前数据都能被其他事务访问

```
osdba=# begin;
BEGIN
osdba=# update test_1 set id = id + 1 where id = 1;
UPDATE 2
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/5104AAB8 |        0 |     0 |   436 |  4896 |    8192 |     8192 |       4 |      7023
(1 row)

osdba=# commit;
COMMIT
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/5104AAB8 |        0 |     0 |   436 |  4896 |    8192 |     8192 |       4 |      7023
(1 row)

osdba=# vacuum test_1;
VACUUM
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/5104AB28 |        0 |     4 |   436 |  4960 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# 

```



通过pd_lsn来查看当前lsn在那个xlog日志下

```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/5016F6C8 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |         0
(1 row)

查看当前的lsn是在那个xlog文件中

osdba=# select pg_xlogfile_name('3/5016F6C8');
     pg_xlogfile_name     
--------------------------
 000000010000000300000050
(1 row)

osdba=# insert into test_1 select generate_series(1,100);
INSERT 0 100
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/50171010 |        0 |     0 |   464 |  4672 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select pg_xlog_location_diff('3/50171010','3/5016F6C8');
 pg_xlog_location_diff 
-----------------------
                  6472


手动检查点，切换新的xlog日志

osdba=# checkpoint;
CHECKPOINT

osdba=# select pg_switch_xlog();
 pg_switch_xlog 
----------------
 3/50173190
(1 row)

osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51001270 |        0 |     0 |   504 |  4352 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select pg_xlogfile_name('3/51001270');
     pg_xlogfile_name     
--------------------------
 000000010000000300000051
(1 row)


```

prune_xid:当页面有数据被更新或者删除时，记录事务的xid

问题是当事务回滚时，prune_id还是保持之前的事务id，而非变为0

```
osdba=# select ctid,* from test_1 where id = 100;
  ctid   | id  
---------+-----
 (0,110) | 100
(1 row)

osdba=# begin;
BEGIN
osdba=# delete from test_1 where id = 100;
DELETE 1
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/510012E0 |        0 |     0 |   504 |  4352 |    8192 |     8192 |       4 |      7005
(1 row)

osdba=# select txid_current();
 txid_current 
--------------
         7005
(1 row)

osdba=# rollback;

ROLLBACK
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/510012E0 |        0 |     0 |   504 |  4352 |    8192 |     8192 |       4 |      7005
(1 row)

```





heap_page_items(page bytea):items的相关信息

`src/include/storage/itemid.h` ，`src/include/access/htup_details.h` 



| 字段        | 类型            | 长度  | 描述                                |
| ----------- | --------------- | ----- | ----------------------------------- |
| t_xmin      | TransactionId   | 4字节 | 插入XID戳                           |
| t_xmax      | TransactionId   | 4字节 | 删除XID戳                           |
| t_cid       | CommandId       | 4字节 | 插入和/或删除 CID戳(使用t_xvac覆盖) |
| t_xvac      | TransactionId   | 4字节 | VACUUM操作移动一行版本的XID         |
| t_ctid      | ItemPointerData | 6字节 | 这个当前的或新行版本的TID           |
| t_infomask2 | uint16          | 2字节 | 字段个数，以及各种标志位            |
| t_infomask  | uint16          | 2字节 | 各种标志位                          |
| t_hoff      | uint8           | 1字节 | 用户数据偏移量                      |



插入数据检查lp_off,lp_flags,lp_len,t_xmin,t_xmax,t_ctid的信息

```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,10);
INSERT 0 10


osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51009210 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2048 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2048 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2048 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2048 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2048 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2048 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2048 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2048 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2048 |     24 |        |      
 10 |   7872 |        1 |     28 |   7010 |      0 |        0 | (0,10) |           1 |       2048 |     24 |        |      
(10 rows)

```

删除数据后的lp_off,lp_flags,lp_len,t_xmin,t_xmax,t_ctid的信息

```
osdba=# delete from test_1 where id = 4;
DELETE 1
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51009480 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |      7011
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |   7011 |        0 | (0,4)  |        8193 |        256 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7872 |        1 |     28 |   7010 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)

```

​	常规情况下，直接使用语句`select * from test_1 where ctid='(0,4)'`或者使用语句`select * from test_1 where id = 4`都不会有返回值。而在`page_header()`还是可以发现`(0,4)`的存在，并且知道是`xid=7011`删除了这条数据。

​	如果不被执行vacuum,或者vacuum full ，这条被删除的数据依然会保留在数据页中。

 	vacuum table后，lp_off,lp_flags,lp_len,t_xmin,t_xmax,t_ctid的信息变化情况

```
osdba=# vacuum test_1;
VACUUM
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  5 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)




```

​	

发现lp_off，lp_flags,lp_len都变为了0，其他剩余字段除了lp外，都置为"NULL"，而且紧随其后的lp_off都往上迁移了，说明数据确实被删除了，空间得到了释放。



​	那么vacuum full table又是怎样，对于lp_off,lp_flags,lp_len,t_xmin,t_xmax,t_ctid，又有什么变化





```
osdba=# vacuum full test_1;
VACUUM
osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7010 |      0 |        0 | (0,1)  |           1 |       2816 |     24 |        |      
  2 |   8128 |        1 |     28 |   7010 |      0 |        0 | (0,2)  |           1 |       2816 |     24 |        |      
  3 |   8096 |        1 |     28 |   7010 |      0 |        0 | (0,3)  |           1 |       2816 |     24 |        |      
  4 |   8064 |        1 |     28 |   7010 |      0 |        0 | (0,4)  |           1 |       2816 |     24 |        |      
  5 |   8032 |        1 |     28 |   7010 |      0 |        0 | (0,5)  |           1 |       2816 |     24 |        |      
  6 |   8000 |        1 |     28 |   7010 |      0 |        0 | (0,6)  |           1 |       2816 |     24 |        |      
  7 |   7968 |        1 |     28 |   7010 |      0 |        0 | (0,7)  |           1 |       2816 |     24 |        |      
  8 |   7936 |        1 |     28 |   7010 |      0 |        0 | (0,8)  |           1 |       2816 |     24 |        |      
  9 |   7904 |        1 |     28 |   7010 |      0 |        0 | (0,9)  |           1 |       2816 |     24 |        |      
(9 rows)

osdba=# select * from test_1 where ctid='(0,7)';
 id 
----
  8
(1 row)

```

​	vacuum full table:数据页会整理自己的数据，这样的操作不建议经常使用，会导致io变高。



​	测试一种场景：delete + vacuum + insert 的数据存放情况



​	步骤：首先插入5条数据，删除一条数据后，在插入5条数据，进行vacuum操作，重新插入一条数据，查看分布情况

```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,5);
INSERT 0 5
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51685880 |        0 |     0 |    44 |  8032 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7037 |      0 |        0 | (0,1)  |           1 |       2048 |     24 |        |      
  2 |   8128 |        1 |     28 |   7037 |      0 |        0 | (0,2)  |           1 |       2048 |     24 |        |      
  3 |   8096 |        1 |     28 |   7037 |      0 |        0 | (0,3)  |           1 |       2048 |     24 |        |      
  4 |   8064 |        1 |     28 |   7037 |      0 |        0 | (0,4)  |           1 |       2048 |     24 |        |      
  5 |   8032 |        1 |     28 |   7037 |      0 |        0 | (0,5)  |           1 |       2048 |     24 |        |      
(5 rows)

osdba=# delete from test_1 where id = 5;
DELETE 1
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51685A58 |        0 |     0 |    44 |  8032 |    8192 |     8192 |       4 |      7039
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7037 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7037 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7037 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   7037 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8032 |        1 |     28 |   7037 |   7039 |        0 | (0,5)  |        8193 |        256 |     24 |        |      
(5 rows)
osdba=# insert into test_1 select generate_series(1,5);
INSERT 0 5
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51685BC8 |        0 |     0 |    64 |  7872 |    8192 |     8192 |       4 |      7039
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7037 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7037 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7037 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   7037 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   8032 |        1 |     28 |   7037 |   7039 |        0 | (0,5)  |        8193 |        256 |     24 |        |      
  6 |   8000 |        1 |     28 |   7040 |      0 |        0 | (0,6)  |           1 |       2048 |     24 |        |      
  7 |   7968 |        1 |     28 |   7040 |      0 |        0 | (0,7)  |           1 |       2048 |     24 |        |      
  8 |   7936 |        1 |     28 |   7040 |      0 |        0 | (0,8)  |           1 |       2048 |     24 |        |      
  9 |   7904 |        1 |     28 |   7040 |      0 |        0 | (0,9)  |           1 |       2048 |     24 |        |      
 10 |   7872 |        1 |     28 |   7040 |      0 |        0 | (0,10) |           1 |       2048 |     24 |        |      
(10 rows)
osdba=# vacuum test_1;
VACUUM
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51685C78 |        0 |     5 |    64 |  7904 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7037 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7037 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7037 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   7037 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |      0 |        0 |      0 |        |        |          |        |             |            |        |        |      
  6 |   8032 |        1 |     28 |   7040 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   8000 |        1 |     28 |   7040 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7968 |        1 |     28 |   7040 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7936 |        1 |     28 |   7040 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7904 |        1 |     28 |   7040 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)

osdba=# insert into test_1 values(1);
INSERT 0 1
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51687DD8 |        0 |     1 |    64 |  7872 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7037 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7037 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7037 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   7037 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   7872 |        1 |     28 |   7041 |      0 |        0 | (0,5)  |           1 |       2048 |     24 |        |      
  6 |   8032 |        1 |     28 |   7040 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   8000 |        1 |     28 |   7040 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7968 |        1 |     28 |   7040 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7936 |        1 |     28 |   7040 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7904 |        1 |     28 |   7040 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
(10 rows)

osdba=# select * from test_1 where ctid = '(0,5)';
 id 
----
  1
(1 row)

osdba=# select * from test_1 where ctid = '(0,6)';
 id 
----
  1
(1 row)

osdba=# select * from test_1 where ctid = '(0,4)';
 id 
----
  4
(1 row)

osdba=# insert into test_1 values(2);
INSERT 0 1
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/51688080 |        0 |     0 |    68 |  7840 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
----+--------+----------+--------+--------+--------+----------+--------+-------------+------------+--------+--------+-------
  1 |   8160 |        1 |     28 |   7037 |      0 |        0 | (0,1)  |           1 |       2304 |     24 |        |      
  2 |   8128 |        1 |     28 |   7037 |      0 |        0 | (0,2)  |           1 |       2304 |     24 |        |      
  3 |   8096 |        1 |     28 |   7037 |      0 |        0 | (0,3)  |           1 |       2304 |     24 |        |      
  4 |   8064 |        1 |     28 |   7037 |      0 |        0 | (0,4)  |           1 |       2304 |     24 |        |      
  5 |   7872 |        1 |     28 |   7041 |      0 |        0 | (0,5)  |           1 |       2304 |     24 |        |      
  6 |   8032 |        1 |     28 |   7040 |      0 |        0 | (0,6)  |           1 |       2304 |     24 |        |      
  7 |   8000 |        1 |     28 |   7040 |      0 |        0 | (0,7)  |           1 |       2304 |     24 |        |      
  8 |   7968 |        1 |     28 |   7040 |      0 |        0 | (0,8)  |           1 |       2304 |     24 |        |      
  9 |   7936 |        1 |     28 |   7040 |      0 |        0 | (0,9)  |           1 |       2304 |     24 |        |      
 10 |   7904 |        1 |     28 |   7040 |      0 |        0 | (0,10) |           1 |       2304 |     24 |        |      
 11 |   7840 |        1 |     28 |   7042 |      0 |        0 | (0,11) |           1 |       2048 |     24 |        |      
(11 rows)



```

​	vacuum table表后，之后插入的数据都在的lp_off都往上提了，新插入的数据除了得到一个之前删除的ctid外，其他并没有什么区别。



​	delete then vacuum

​	1、数据的lp_off会往上移动，而新的数据只能在最小的lp_off分配

​	2、新的数据可能ctid较小，但是数据位置可能在数据页的中部，而非底部







​	测试场景 update + vacuum 

​	这个场景其实就是类似delete  + vacuum + insert

```
osdba=# truncate table test_1;
TRUNCATE TABLE
osdba=# insert into test_1 select generate_series(1,100);
INSERT 0 100
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/516F2D08 |        0 |     0 |   424 |  4992 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp  | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid  | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
-----+--------+----------+--------+--------+--------+----------+---------+-------------+------------+--------+--------+-------
   1 |   8160 |        1 |     28 |   7061 |      0 |        0 | (0,1)   |           1 |       2048 |     24 |        |      
   2 |   8128 |        1 |     28 |   7061 |      0 |        0 | (0,2)   |           1 |       2048 |     24 |        |      
   3 |   8096 |        1 |     28 |   7061 |      0 |        0 | (0,3)   |           1 |       2048 |     24 |        |      
   4 |   8064 |        1 |     28 |   7061 |      0 |        0 | (0,4)   |           1 |       2048 |     24 |        |      
   5 |   8032 |        1 |     28 |   7061 |      0 |        0 | (0,5)   |           1 |       2048 |     24 |        |      
   6 |   8000 |        1 |     28 |   7061 |      0 |        0 | (0,6)   |           1 |       2048 |     24 |        |      
   7 |   7968 |        1 |     28 |   7061 |      0 |        0 | (0,7)   |           1 |       2048 |     24 |        |      
   8 |   7936 |        1 |     28 |   7061 |      0 |        0 | (0,8)   |           1 |       2048 |     24 |        |      
   9 |   7904 |        1 |     28 |   7061 |      0 |        0 | (0,9)   |           1 |       2048 |     24 |        |      
  10 |   7872 |        1 |     28 |   7061 |      0 |        0 | (0,10)  |           1 |       2048 |     24 |        |      
  11 |   7840 |        1 |     28 |   7061 |      0 |        0 | (0,11)  |           1 |       2048 |     24 |        |      
  12 |   7808 |        1 |     28 |   7061 |      0 |        0 | (0,12)  |           1 |       2048 |     24 |        |      
  13 |   7776 |        1 |     28 |   7061 |      0 |        0 | (0,13)  |           1 |       2048 |     24 |        |      
  14 |   7744 |        1 |     28 |   7061 |      0 |        0 | (0,14)  |           1 |       2048 |     24 |        |      
  15 |   7712 |        1 |     28 |   7061 |      0 |        0 | (0,15)  |           1 |       2048 |     24 |        |      
  16 |   7680 |        1 |     28 |   7061 |      0 |        0 | (0,16)  |           1 |       2048 |     24 |        |      
  17 |   7648 |        1 |     28 |   7061 |      0 |        0 | (0,17)  |           1 |       2048 |     24 |        |      
  18 |   7616 |        1 |     28 |   7061 |      0 |        0 | (0,18)  |           1 |       2048 |     24 |        |      
  19 |   7584 |        1 |     28 |   7061 |      0 |        0 | (0,19)  |           1 |       2048 |     24 |        |      
  20 |   7552 |        1 |     28 |   7061 |      0 |        0 | (0,20)  |           1 |       2048 |     24 |        |      
  21 |   7520 |        1 |     28 |   7061 |      0 |        0 | (0,21)  |           1 |       2048 |     24 |        |      
  22 |   7488 |        1 |     28 |   7061 |      0 |        0 | (0,22)  |           1 |       2048 |     24 |        |      
  23 |   7456 |        1 |     28 |   7061 |      0 |        0 | (0,23)  |           1 |       2048 |     24 |        |      
  24 |   7424 |        1 |     28 |   7061 |      0 |        0 | (0,24)  |           1 |       2048 |     24 |        |      
  25 |   7392 |        1 |     28 |   7061 |      0 |        0 | (0,25)  |           1 |       2048 |     24 |        |      
  26 |   7360 |        1 |     28 |   7061 |      0 |        0 | (0,26)  |           1 |       2048 |     24 |        |      
  27 |   7328 |        1 |     28 |   7061 |      0 |        0 | (0,27)  |           1 |       2048 |     24 |        |      
  28 |   7296 |        1 |     28 |   7061 |      0 |        0 | (0,28)  |           1 |       2048 |     24 |        |      
  29 |   7264 |        1 |     28 |   7061 |      0 |        0 | (0,29)  |           1 |       2048 |     24 |        |      
  30 |   7232 |        1 |     28 |   7061 |      0 |        0 | (0,30)  |           1 |       2048 |     24 |        |      
  31 |   7200 |        1 |     28 |   7061 |      0 |        0 | (0,31)  |           1 |       2048 |     24 |        |      
  32 |   7168 |        1 |     28 |   7061 |      0 |        0 | (0,32)  |           1 |       2048 |     24 |        |      
  33 |   7136 |        1 |     28 |   7061 |      0 |        0 | (0,33)  |           1 |       2048 |     24 |        |      
  34 |   7104 |        1 |     28 |   7061 |      0 |        0 | (0,34)  |           1 |       2048 |     24 |        |      
  35 |   7072 |        1 |     28 |   7061 |      0 |        0 | (0,35)  |           1 |       2048 |     24 |        |      
  36 |   7040 |        1 |     28 |   7061 |      0 |        0 | (0,36)  |           1 |       2048 |     24 |        |      
  37 |   7008 |        1 |     28 |   7061 |      0 |        0 | (0,37)  |           1 |       2048 |     24 |        |      
  38 |   6976 |        1 |     28 |   7061 |      0 |        0 | (0,38)  |           1 |       2048 |     24 |        |      
  39 |   6944 |        1 |     28 |   7061 |      0 |        0 | (0,39)  |           1 |       2048 |     24 |        |      
  40 |   6912 |        1 |     28 |   7061 |      0 |        0 | (0,40)  |           1 |       2048 |     24 |        |      
  41 |   6880 |        1 |     28 |   7061 |      0 |        0 | (0,41)  |           1 |       2048 |     24 |        |      
  42 |   6848 |        1 |     28 |   7061 |      0 |        0 | (0,42)  |           1 |       2048 |     24 |        |      
  43 |   6816 |        1 |     28 |   7061 |      0 |        0 | (0,43)  |           1 |       2048 |     24 |        |      
  44 |   6784 |        1 |     28 |   7061 |      0 |        0 | (0,44)  |           1 |       2048 |     24 |        |      
  45 |   6752 |        1 |     28 |   7061 |      0 |        0 | (0,45)  |           1 |       2048 |     24 |        |      
  46 |   6720 |        1 |     28 |   7061 |      0 |        0 | (0,46)  |           1 |       2048 |     24 |        |      
  47 |   6688 |        1 |     28 |   7061 |      0 |        0 | (0,47)  |           1 |       2048 |     24 |        |      
  48 |   6656 |        1 |     28 |   7061 |      0 |        0 | (0,48)  |           1 |       2048 |     24 |        |      
  49 |   6624 |        1 |     28 |   7061 |      0 |        0 | (0,49)  |           1 |       2048 |     24 |        |      
  50 |   6592 |        1 |     28 |   7061 |      0 |        0 | (0,50)  |           1 |       2048 |     24 |        |      
  51 |   6560 |        1 |     28 |   7061 |      0 |        0 | (0,51)  |           1 |       2048 |     24 |        |      
  52 |   6528 |        1 |     28 |   7061 |      0 |        0 | (0,52)  |           1 |       2048 |     24 |        |      
  53 |   6496 |        1 |     28 |   7061 |      0 |        0 | (0,53)  |           1 |       2048 |     24 |        |      
  54 |   6464 |        1 |     28 |   7061 |      0 |        0 | (0,54)  |           1 |       2048 |     24 |        |      
  55 |   6432 |        1 |     28 |   7061 |      0 |        0 | (0,55)  |           1 |       2048 |     24 |        |      
  56 |   6400 |        1 |     28 |   7061 |      0 |        0 | (0,56)  |           1 |       2048 |     24 |        |      
  57 |   6368 |        1 |     28 |   7061 |      0 |        0 | (0,57)  |           1 |       2048 |     24 |        |      
  58 |   6336 |        1 |     28 |   7061 |      0 |        0 | (0,58)  |           1 |       2048 |     24 |        |      
  59 |   6304 |        1 |     28 |   7061 |      0 |        0 | (0,59)  |           1 |       2048 |     24 |        |      
  60 |   6272 |        1 |     28 |   7061 |      0 |        0 | (0,60)  |           1 |       2048 |     24 |        |      
  61 |   6240 |        1 |     28 |   7061 |      0 |        0 | (0,61)  |           1 |       2048 |     24 |        |      
  62 |   6208 |        1 |     28 |   7061 |      0 |        0 | (0,62)  |           1 |       2048 |     24 |        |      
  63 |   6176 |        1 |     28 |   7061 |      0 |        0 | (0,63)  |           1 |       2048 |     24 |        |      
  64 |   6144 |        1 |     28 |   7061 |      0 |        0 | (0,64)  |           1 |       2048 |     24 |        |      
  65 |   6112 |        1 |     28 |   7061 |      0 |        0 | (0,65)  |           1 |       2048 |     24 |        |      
  66 |   6080 |        1 |     28 |   7061 |      0 |        0 | (0,66)  |           1 |       2048 |     24 |        |      
  67 |   6048 |        1 |     28 |   7061 |      0 |        0 | (0,67)  |           1 |       2048 |     24 |        |      
  68 |   6016 |        1 |     28 |   7061 |      0 |        0 | (0,68)  |           1 |       2048 |     24 |        |      
  69 |   5984 |        1 |     28 |   7061 |      0 |        0 | (0,69)  |           1 |       2048 |     24 |        |      
  70 |   5952 |        1 |     28 |   7061 |      0 |        0 | (0,70)  |           1 |       2048 |     24 |        |      
  71 |   5920 |        1 |     28 |   7061 |      0 |        0 | (0,71)  |           1 |       2048 |     24 |        |      
  72 |   5888 |        1 |     28 |   7061 |      0 |        0 | (0,72)  |           1 |       2048 |     24 |        |      
  73 |   5856 |        1 |     28 |   7061 |      0 |        0 | (0,73)  |           1 |       2048 |     24 |        |      
  74 |   5824 |        1 |     28 |   7061 |      0 |        0 | (0,74)  |           1 |       2048 |     24 |        |      
  75 |   5792 |        1 |     28 |   7061 |      0 |        0 | (0,75)  |           1 |       2048 |     24 |        |      
  76 |   5760 |        1 |     28 |   7061 |      0 |        0 | (0,76)  |           1 |       2048 |     24 |        |      
  77 |   5728 |        1 |     28 |   7061 |      0 |        0 | (0,77)  |           1 |       2048 |     24 |        |      
  78 |   5696 |        1 |     28 |   7061 |      0 |        0 | (0,78)  |           1 |       2048 |     24 |        |      
  79 |   5664 |        1 |     28 |   7061 |      0 |        0 | (0,79)  |           1 |       2048 |     24 |        |      
  80 |   5632 |        1 |     28 |   7061 |      0 |        0 | (0,80)  |           1 |       2048 |     24 |        |      
  81 |   5600 |        1 |     28 |   7061 |      0 |        0 | (0,81)  |           1 |       2048 |     24 |        |      
  82 |   5568 |        1 |     28 |   7061 |      0 |        0 | (0,82)  |           1 |       2048 |     24 |        |      
  83 |   5536 |        1 |     28 |   7061 |      0 |        0 | (0,83)  |           1 |       2048 |     24 |        |      
  84 |   5504 |        1 |     28 |   7061 |      0 |        0 | (0,84)  |           1 |       2048 |     24 |        |      
  85 |   5472 |        1 |     28 |   7061 |      0 |        0 | (0,85)  |           1 |       2048 |     24 |        |      
  86 |   5440 |        1 |     28 |   7061 |      0 |        0 | (0,86)  |           1 |       2048 |     24 |        |      
  87 |   5408 |        1 |     28 |   7061 |      0 |        0 | (0,87)  |           1 |       2048 |     24 |        |      
  88 |   5376 |        1 |     28 |   7061 |      0 |        0 | (0,88)  |           1 |       2048 |     24 |        |      
  89 |   5344 |        1 |     28 |   7061 |      0 |        0 | (0,89)  |           1 |       2048 |     24 |        |      
  90 |   5312 |        1 |     28 |   7061 |      0 |        0 | (0,90)  |           1 |       2048 |     24 |        |      
  91 |   5280 |        1 |     28 |   7061 |      0 |        0 | (0,91)  |           1 |       2048 |     24 |        |      
  92 |   5248 |        1 |     28 |   7061 |      0 |        0 | (0,92)  |           1 |       2048 |     24 |        |      
  93 |   5216 |        1 |     28 |   7061 |      0 |        0 | (0,93)  |           1 |       2048 |     24 |        |      
  94 |   5184 |        1 |     28 |   7061 |      0 |        0 | (0,94)  |           1 |       2048 |     24 |        |      
  95 |   5152 |        1 |     28 |   7061 |      0 |        0 | (0,95)  |           1 |       2048 |     24 |        |      
  96 |   5120 |        1 |     28 |   7061 |      0 |        0 | (0,96)  |           1 |       2048 |     24 |        |      
  97 |   5088 |        1 |     28 |   7061 |      0 |        0 | (0,97)  |           1 |       2048 |     24 |        |      
  98 |   5056 |        1 |     28 |   7061 |      0 |        0 | (0,98)  |           1 |       2048 |     24 |        |      
  99 |   5024 |        1 |     28 |   7061 |      0 |        0 | (0,99)  |           1 |       2048 |     24 |        |      
 100 |   4992 |        1 |     28 |   7061 |      0 |        0 | (0,100) |           1 |       2048 |     24 |        |      
(100 rows)

osdba=# update test_1 set id = 111 where id < 10;
UPDATE 9
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/516F3008 |        0 |     0 |   460 |  4704 |    8192 |     8192 |       4 |      7062
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp  | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid  | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
-----+--------+----------+--------+--------+--------+----------+---------+-------------+------------+--------+--------+-------
   1 |   8160 |        1 |     28 |   7061 |   7062 |        0 | (0,101) |       16385 |       1280 |     24 |        |      
   2 |   8128 |        1 |     28 |   7061 |   7062 |        0 | (0,102) |       16385 |       1280 |     24 |        |      
   3 |   8096 |        1 |     28 |   7061 |   7062 |        0 | (0,103) |       16385 |       1280 |     24 |        |      
   4 |   8064 |        1 |     28 |   7061 |   7062 |        0 | (0,104) |       16385 |       1280 |     24 |        |      
   5 |   8032 |        1 |     28 |   7061 |   7062 |        0 | (0,105) |       16385 |       1280 |     24 |        |      
   6 |   8000 |        1 |     28 |   7061 |   7062 |        0 | (0,106) |       16385 |       1280 |     24 |        |      
   7 |   7968 |        1 |     28 |   7061 |   7062 |        0 | (0,107) |       16385 |       1280 |     24 |        |      
   8 |   7936 |        1 |     28 |   7061 |   7062 |        0 | (0,108) |       16385 |       1280 |     24 |        |      
   9 |   7904 |        1 |     28 |   7061 |   7062 |        0 | (0,109) |       16385 |       1280 |     24 |        |      
  10 |   7872 |        1 |     28 |   7061 |      0 |        0 | (0,10)  |           1 |       2304 |     24 |        |      
  11 |   7840 |        1 |     28 |   7061 |      0 |        0 | (0,11)  |           1 |       2304 |     24 |        |      
  12 |   7808 |        1 |     28 |   7061 |      0 |        0 | (0,12)  |           1 |       2304 |     24 |        |      
  13 |   7776 |        1 |     28 |   7061 |      0 |        0 | (0,13)  |           1 |       2304 |     24 |        |      
  14 |   7744 |        1 |     28 |   7061 |      0 |        0 | (0,14)  |           1 |       2304 |     24 |        |      
  15 |   7712 |        1 |     28 |   7061 |      0 |        0 | (0,15)  |           1 |       2304 |     24 |        |      
  16 |   7680 |        1 |     28 |   7061 |      0 |        0 | (0,16)  |           1 |       2304 |     24 |        |      
  17 |   7648 |        1 |     28 |   7061 |      0 |        0 | (0,17)  |           1 |       2304 |     24 |        |      
  18 |   7616 |        1 |     28 |   7061 |      0 |        0 | (0,18)  |           1 |       2304 |     24 |        |      
  19 |   7584 |        1 |     28 |   7061 |      0 |        0 | (0,19)  |           1 |       2304 |     24 |        |      
  20 |   7552 |        1 |     28 |   7061 |      0 |        0 | (0,20)  |           1 |       2304 |     24 |        |      
  21 |   7520 |        1 |     28 |   7061 |      0 |        0 | (0,21)  |           1 |       2304 |     24 |        |      
  22 |   7488 |        1 |     28 |   7061 |      0 |        0 | (0,22)  |           1 |       2304 |     24 |        |      
  23 |   7456 |        1 |     28 |   7061 |      0 |        0 | (0,23)  |           1 |       2304 |     24 |        |      
  24 |   7424 |        1 |     28 |   7061 |      0 |        0 | (0,24)  |           1 |       2304 |     24 |        |      
  25 |   7392 |        1 |     28 |   7061 |      0 |        0 | (0,25)  |           1 |       2304 |     24 |        |      
  26 |   7360 |        1 |     28 |   7061 |      0 |        0 | (0,26)  |           1 |       2304 |     24 |        |      
  27 |   7328 |        1 |     28 |   7061 |      0 |        0 | (0,27)  |           1 |       2304 |     24 |        |      
  28 |   7296 |        1 |     28 |   7061 |      0 |        0 | (0,28)  |           1 |       2304 |     24 |        |      
  29 |   7264 |        1 |     28 |   7061 |      0 |        0 | (0,29)  |           1 |       2304 |     24 |        |      
  30 |   7232 |        1 |     28 |   7061 |      0 |        0 | (0,30)  |           1 |       2304 |     24 |        |      
  31 |   7200 |        1 |     28 |   7061 |      0 |        0 | (0,31)  |           1 |       2304 |     24 |        |      
  32 |   7168 |        1 |     28 |   7061 |      0 |        0 | (0,32)  |           1 |       2304 |     24 |        |      
  33 |   7136 |        1 |     28 |   7061 |      0 |        0 | (0,33)  |           1 |       2304 |     24 |        |      
  34 |   7104 |        1 |     28 |   7061 |      0 |        0 | (0,34)  |           1 |       2304 |     24 |        |      
  35 |   7072 |        1 |     28 |   7061 |      0 |        0 | (0,35)  |           1 |       2304 |     24 |        |      
  36 |   7040 |        1 |     28 |   7061 |      0 |        0 | (0,36)  |           1 |       2304 |     24 |        |      
  37 |   7008 |        1 |     28 |   7061 |      0 |        0 | (0,37)  |           1 |       2304 |     24 |        |      
  38 |   6976 |        1 |     28 |   7061 |      0 |        0 | (0,38)  |           1 |       2304 |     24 |        |      
  39 |   6944 |        1 |     28 |   7061 |      0 |        0 | (0,39)  |           1 |       2304 |     24 |        |      
  40 |   6912 |        1 |     28 |   7061 |      0 |        0 | (0,40)  |           1 |       2304 |     24 |        |      
  41 |   6880 |        1 |     28 |   7061 |      0 |        0 | (0,41)  |           1 |       2304 |     24 |        |      
  42 |   6848 |        1 |     28 |   7061 |      0 |        0 | (0,42)  |           1 |       2304 |     24 |        |      
  43 |   6816 |        1 |     28 |   7061 |      0 |        0 | (0,43)  |           1 |       2304 |     24 |        |      
  44 |   6784 |        1 |     28 |   7061 |      0 |        0 | (0,44)  |           1 |       2304 |     24 |        |      
  45 |   6752 |        1 |     28 |   7061 |      0 |        0 | (0,45)  |           1 |       2304 |     24 |        |      
  46 |   6720 |        1 |     28 |   7061 |      0 |        0 | (0,46)  |           1 |       2304 |     24 |        |      
  47 |   6688 |        1 |     28 |   7061 |      0 |        0 | (0,47)  |           1 |       2304 |     24 |        |      
  48 |   6656 |        1 |     28 |   7061 |      0 |        0 | (0,48)  |           1 |       2304 |     24 |        |      
  49 |   6624 |        1 |     28 |   7061 |      0 |        0 | (0,49)  |           1 |       2304 |     24 |        |      
  50 |   6592 |        1 |     28 |   7061 |      0 |        0 | (0,50)  |           1 |       2304 |     24 |        |      
  51 |   6560 |        1 |     28 |   7061 |      0 |        0 | (0,51)  |           1 |       2304 |     24 |        |      
  52 |   6528 |        1 |     28 |   7061 |      0 |        0 | (0,52)  |           1 |       2304 |     24 |        |      
  53 |   6496 |        1 |     28 |   7061 |      0 |        0 | (0,53)  |           1 |       2304 |     24 |        |      
  54 |   6464 |        1 |     28 |   7061 |      0 |        0 | (0,54)  |           1 |       2304 |     24 |        |      
  55 |   6432 |        1 |     28 |   7061 |      0 |        0 | (0,55)  |           1 |       2304 |     24 |        |      
  56 |   6400 |        1 |     28 |   7061 |      0 |        0 | (0,56)  |           1 |       2304 |     24 |        |      
  57 |   6368 |        1 |     28 |   7061 |      0 |        0 | (0,57)  |           1 |       2304 |     24 |        |      
  58 |   6336 |        1 |     28 |   7061 |      0 |        0 | (0,58)  |           1 |       2304 |     24 |        |      
  59 |   6304 |        1 |     28 |   7061 |      0 |        0 | (0,59)  |           1 |       2304 |     24 |        |      
  60 |   6272 |        1 |     28 |   7061 |      0 |        0 | (0,60)  |           1 |       2304 |     24 |        |      
  61 |   6240 |        1 |     28 |   7061 |      0 |        0 | (0,61)  |           1 |       2304 |     24 |        |      
  62 |   6208 |        1 |     28 |   7061 |      0 |        0 | (0,62)  |           1 |       2304 |     24 |        |      
  63 |   6176 |        1 |     28 |   7061 |      0 |        0 | (0,63)  |           1 |       2304 |     24 |        |      
  64 |   6144 |        1 |     28 |   7061 |      0 |        0 | (0,64)  |           1 |       2304 |     24 |        |      
  65 |   6112 |        1 |     28 |   7061 |      0 |        0 | (0,65)  |           1 |       2304 |     24 |        |      
  66 |   6080 |        1 |     28 |   7061 |      0 |        0 | (0,66)  |           1 |       2304 |     24 |        |      
  67 |   6048 |        1 |     28 |   7061 |      0 |        0 | (0,67)  |           1 |       2304 |     24 |        |      
  68 |   6016 |        1 |     28 |   7061 |      0 |        0 | (0,68)  |           1 |       2304 |     24 |        |      
  69 |   5984 |        1 |     28 |   7061 |      0 |        0 | (0,69)  |           1 |       2304 |     24 |        |      
  70 |   5952 |        1 |     28 |   7061 |      0 |        0 | (0,70)  |           1 |       2304 |     24 |        |      
  71 |   5920 |        1 |     28 |   7061 |      0 |        0 | (0,71)  |           1 |       2304 |     24 |        |      
  72 |   5888 |        1 |     28 |   7061 |      0 |        0 | (0,72)  |           1 |       2304 |     24 |        |      
  73 |   5856 |        1 |     28 |   7061 |      0 |        0 | (0,73)  |           1 |       2304 |     24 |        |      
  74 |   5824 |        1 |     28 |   7061 |      0 |        0 | (0,74)  |           1 |       2304 |     24 |        |      
  75 |   5792 |        1 |     28 |   7061 |      0 |        0 | (0,75)  |           1 |       2304 |     24 |        |      
  76 |   5760 |        1 |     28 |   7061 |      0 |        0 | (0,76)  |           1 |       2304 |     24 |        |      
  77 |   5728 |        1 |     28 |   7061 |      0 |        0 | (0,77)  |           1 |       2304 |     24 |        |      
  78 |   5696 |        1 |     28 |   7061 |      0 |        0 | (0,78)  |           1 |       2304 |     24 |        |      
  79 |   5664 |        1 |     28 |   7061 |      0 |        0 | (0,79)  |           1 |       2304 |     24 |        |      
  80 |   5632 |        1 |     28 |   7061 |      0 |        0 | (0,80)  |           1 |       2304 |     24 |        |      
  81 |   5600 |        1 |     28 |   7061 |      0 |        0 | (0,81)  |           1 |       2304 |     24 |        |      
  82 |   5568 |        1 |     28 |   7061 |      0 |        0 | (0,82)  |           1 |       2304 |     24 |        |      
  83 |   5536 |        1 |     28 |   7061 |      0 |        0 | (0,83)  |           1 |       2304 |     24 |        |      
  84 |   5504 |        1 |     28 |   7061 |      0 |        0 | (0,84)  |           1 |       2304 |     24 |        |      
  85 |   5472 |        1 |     28 |   7061 |      0 |        0 | (0,85)  |           1 |       2304 |     24 |        |      
  86 |   5440 |        1 |     28 |   7061 |      0 |        0 | (0,86)  |           1 |       2304 |     24 |        |      
  87 |   5408 |        1 |     28 |   7061 |      0 |        0 | (0,87)  |           1 |       2304 |     24 |        |      
  88 |   5376 |        1 |     28 |   7061 |      0 |        0 | (0,88)  |           1 |       2304 |     24 |        |      
  89 |   5344 |        1 |     28 |   7061 |      0 |        0 | (0,89)  |           1 |       2304 |     24 |        |      
  90 |   5312 |        1 |     28 |   7061 |      0 |        0 | (0,90)  |           1 |       2304 |     24 |        |      
  91 |   5280 |        1 |     28 |   7061 |      0 |        0 | (0,91)  |           1 |       2304 |     24 |        |      
  92 |   5248 |        1 |     28 |   7061 |      0 |        0 | (0,92)  |           1 |       2304 |     24 |        |      
  93 |   5216 |        1 |     28 |   7061 |      0 |        0 | (0,93)  |           1 |       2304 |     24 |        |      
  94 |   5184 |        1 |     28 |   7061 |      0 |        0 | (0,94)  |           1 |       2304 |     24 |        |      
  95 |   5152 |        1 |     28 |   7061 |      0 |        0 | (0,95)  |           1 |       2304 |     24 |        |      
  96 |   5120 |        1 |     28 |   7061 |      0 |        0 | (0,96)  |           1 |       2304 |     24 |        |      
  97 |   5088 |        1 |     28 |   7061 |      0 |        0 | (0,97)  |           1 |       2304 |     24 |        |      
  98 |   5056 |        1 |     28 |   7061 |      0 |        0 | (0,98)  |           1 |       2304 |     24 |        |      
  99 |   5024 |        1 |     28 |   7061 |      0 |        0 | (0,99)  |           1 |       2304 |     24 |        |      
 100 |   4992 |        1 |     28 |   7061 |      0 |        0 | (0,100) |           1 |       2304 |     24 |        |      
 101 |   4960 |        1 |     28 |   7062 |      0 |        0 | (0,101) |       32769 |      10496 |     24 |        |      
 102 |   4928 |        1 |     28 |   7062 |      0 |        0 | (0,102) |       32769 |      10496 |     24 |        |      
 103 |   4896 |        1 |     28 |   7062 |      0 |        0 | (0,103) |       32769 |      10496 |     24 |        |      
 104 |   4864 |        1 |     28 |   7062 |      0 |        0 | (0,104) |       32769 |      10496 |     24 |        |      
 105 |   4832 |        1 |     28 |   7062 |      0 |        0 | (0,105) |       32769 |      10496 |     24 |        |      
 106 |   4800 |        1 |     28 |   7062 |      0 |        0 | (0,106) |       32769 |      10496 |     24 |        |      
 107 |   4768 |        1 |     28 |   7062 |      0 |        0 | (0,107) |       32769 |      10496 |     24 |        |      
 108 |   4736 |        1 |     28 |   7062 |      0 |        0 | (0,108) |       32769 |      10496 |     24 |        |      
 109 |   4704 |        1 |     28 |   7062 |      0 |        0 | (0,109) |       32769 |      10496 |     24 |        |      
(109 rows)


osdba=# vacuum test_1;
VACUUM
osdba=# select * from page_header(get_raw_page('test_1',0));
    lsn     | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
------------+----------+-------+-------+-------+---------+----------+---------+-----------
 3/516F4580 |        0 |     4 |   460 |  4992 |    8192 |     8192 |       4 |         0
(1 row)

osdba=# select * from heap_page_items(get_raw_page('test_1',0));
 lp  | lp_off | lp_flags | lp_len | t_xmin | t_xmax | t_field3 | t_ctid  | t_infomask2 | t_infomask | t_hoff | t_bits | t_oid 
-----+--------+----------+--------+--------+--------+----------+---------+-------------+------------+--------+--------+-------
   1 |    101 |        2 |      0 |        |        |          |         |             |            |        |        |      
   2 |    102 |        2 |      0 |        |        |          |         |             |            |        |        |      
   3 |    103 |        2 |      0 |        |        |          |         |             |            |        |        |      
   4 |    104 |        2 |      0 |        |        |          |         |             |            |        |        |      
   5 |    105 |        2 |      0 |        |        |          |         |             |            |        |        |      
   6 |    106 |        2 |      0 |        |        |          |         |             |            |        |        |      
   7 |    107 |        2 |      0 |        |        |          |         |             |            |        |        |      
   8 |    108 |        2 |      0 |        |        |          |         |             |            |        |        |      
   9 |    109 |        2 |      0 |        |        |          |         |             |            |        |        |      
  10 |   8160 |        1 |     28 |   7061 |      0 |        0 | (0,10)  |           1 |       2304 |     24 |        |      
  11 |   8128 |        1 |     28 |   7061 |      0 |        0 | (0,11)  |           1 |       2304 |     24 |        |      
  12 |   8096 |        1 |     28 |   7061 |      0 |        0 | (0,12)  |           1 |       2304 |     24 |        |      
  13 |   8064 |        1 |     28 |   7061 |      0 |        0 | (0,13)  |           1 |       2304 |     24 |        |      
  14 |   8032 |        1 |     28 |   7061 |      0 |        0 | (0,14)  |           1 |       2304 |     24 |        |      
  15 |   8000 |        1 |     28 |   7061 |      0 |        0 | (0,15)  |           1 |       2304 |     24 |        |      
  16 |   7968 |        1 |     28 |   7061 |      0 |        0 | (0,16)  |           1 |       2304 |     24 |        |      
  17 |   7936 |        1 |     28 |   7061 |      0 |        0 | (0,17)  |           1 |       2304 |     24 |        |      
  18 |   7904 |        1 |     28 |   7061 |      0 |        0 | (0,18)  |           1 |       2304 |     24 |        |      
  19 |   7872 |        1 |     28 |   7061 |      0 |        0 | (0,19)  |           1 |       2304 |     24 |        |      
  20 |   7840 |        1 |     28 |   7061 |      0 |        0 | (0,20)  |           1 |       2304 |     24 |        |      
  21 |   7808 |        1 |     28 |   7061 |      0 |        0 | (0,21)  |           1 |       2304 |     24 |        |      
  22 |   7776 |        1 |     28 |   7061 |      0 |        0 | (0,22)  |           1 |       2304 |     24 |        |      
  23 |   7744 |        1 |     28 |   7061 |      0 |        0 | (0,23)  |           1 |       2304 |     24 |        |      
  24 |   7712 |        1 |     28 |   7061 |      0 |        0 | (0,24)  |           1 |       2304 |     24 |        |      
  25 |   7680 |        1 |     28 |   7061 |      0 |        0 | (0,25)  |           1 |       2304 |     24 |        |      
  26 |   7648 |        1 |     28 |   7061 |      0 |        0 | (0,26)  |           1 |       2304 |     24 |        |      
  27 |   7616 |        1 |     28 |   7061 |      0 |        0 | (0,27)  |           1 |       2304 |     24 |        |      
  28 |   7584 |        1 |     28 |   7061 |      0 |        0 | (0,28)  |           1 |       2304 |     24 |        |      
  29 |   7552 |        1 |     28 |   7061 |      0 |        0 | (0,29)  |           1 |       2304 |     24 |        |      
  30 |   7520 |        1 |     28 |   7061 |      0 |        0 | (0,30)  |           1 |       2304 |     24 |        |      
  31 |   7488 |        1 |     28 |   7061 |      0 |        0 | (0,31)  |           1 |       2304 |     24 |        |      
  32 |   7456 |        1 |     28 |   7061 |      0 |        0 | (0,32)  |           1 |       2304 |     24 |        |      
  33 |   7424 |        1 |     28 |   7061 |      0 |        0 | (0,33)  |           1 |       2304 |     24 |        |      
  34 |   7392 |        1 |     28 |   7061 |      0 |        0 | (0,34)  |           1 |       2304 |     24 |        |      
  35 |   7360 |        1 |     28 |   7061 |      0 |        0 | (0,35)  |           1 |       2304 |     24 |        |      
  36 |   7328 |        1 |     28 |   7061 |      0 |        0 | (0,36)  |           1 |       2304 |     24 |        |      
  37 |   7296 |        1 |     28 |   7061 |      0 |        0 | (0,37)  |           1 |       2304 |     24 |        |      
  38 |   7264 |        1 |     28 |   7061 |      0 |        0 | (0,38)  |           1 |       2304 |     24 |        |      
  39 |   7232 |        1 |     28 |   7061 |      0 |        0 | (0,39)  |           1 |       2304 |     24 |        |      
  40 |   7200 |        1 |     28 |   7061 |      0 |        0 | (0,40)  |           1 |       2304 |     24 |        |      
  41 |   7168 |        1 |     28 |   7061 |      0 |        0 | (0,41)  |           1 |       2304 |     24 |        |      
  42 |   7136 |        1 |     28 |   7061 |      0 |        0 | (0,42)  |           1 |       2304 |     24 |        |      
  43 |   7104 |        1 |     28 |   7061 |      0 |        0 | (0,43)  |           1 |       2304 |     24 |        |      
  44 |   7072 |        1 |     28 |   7061 |      0 |        0 | (0,44)  |           1 |       2304 |     24 |        |      
  45 |   7040 |        1 |     28 |   7061 |      0 |        0 | (0,45)  |           1 |       2304 |     24 |        |      
  46 |   7008 |        1 |     28 |   7061 |      0 |        0 | (0,46)  |           1 |       2304 |     24 |        |      
  47 |   6976 |        1 |     28 |   7061 |      0 |        0 | (0,47)  |           1 |       2304 |     24 |        |      
  48 |   6944 |        1 |     28 |   7061 |      0 |        0 | (0,48)  |           1 |       2304 |     24 |        |      
  49 |   6912 |        1 |     28 |   7061 |      0 |        0 | (0,49)  |           1 |       2304 |     24 |        |      
  50 |   6880 |        1 |     28 |   7061 |      0 |        0 | (0,50)  |           1 |       2304 |     24 |        |      
  51 |   6848 |        1 |     28 |   7061 |      0 |        0 | (0,51)  |           1 |       2304 |     24 |        |      
  52 |   6816 |        1 |     28 |   7061 |      0 |        0 | (0,52)  |           1 |       2304 |     24 |        |      
  53 |   6784 |        1 |     28 |   7061 |      0 |        0 | (0,53)  |           1 |       2304 |     24 |        |      
  54 |   6752 |        1 |     28 |   7061 |      0 |        0 | (0,54)  |           1 |       2304 |     24 |        |      
  55 |   6720 |        1 |     28 |   7061 |      0 |        0 | (0,55)  |           1 |       2304 |     24 |        |      
  56 |   6688 |        1 |     28 |   7061 |      0 |        0 | (0,56)  |           1 |       2304 |     24 |        |      
  57 |   6656 |        1 |     28 |   7061 |      0 |        0 | (0,57)  |           1 |       2304 |     24 |        |      
  58 |   6624 |        1 |     28 |   7061 |      0 |        0 | (0,58)  |           1 |       2304 |     24 |        |      
  59 |   6592 |        1 |     28 |   7061 |      0 |        0 | (0,59)  |           1 |       2304 |     24 |        |      
  60 |   6560 |        1 |     28 |   7061 |      0 |        0 | (0,60)  |           1 |       2304 |     24 |        |      
  61 |   6528 |        1 |     28 |   7061 |      0 |        0 | (0,61)  |           1 |       2304 |     24 |        |      
  62 |   6496 |        1 |     28 |   7061 |      0 |        0 | (0,62)  |           1 |       2304 |     24 |        |      
  63 |   6464 |        1 |     28 |   7061 |      0 |        0 | (0,63)  |           1 |       2304 |     24 |        |      
  64 |   6432 |        1 |     28 |   7061 |      0 |        0 | (0,64)  |           1 |       2304 |     24 |        |      
  65 |   6400 |        1 |     28 |   7061 |      0 |        0 | (0,65)  |           1 |       2304 |     24 |        |      
  66 |   6368 |        1 |     28 |   7061 |      0 |        0 | (0,66)  |           1 |       2304 |     24 |        |      
  67 |   6336 |        1 |     28 |   7061 |      0 |        0 | (0,67)  |           1 |       2304 |     24 |        |      
  68 |   6304 |        1 |     28 |   7061 |      0 |        0 | (0,68)  |           1 |       2304 |     24 |        |      
  69 |   6272 |        1 |     28 |   7061 |      0 |        0 | (0,69)  |           1 |       2304 |     24 |        |      
  70 |   6240 |        1 |     28 |   7061 |      0 |        0 | (0,70)  |           1 |       2304 |     24 |        |      
  71 |   6208 |        1 |     28 |   7061 |      0 |        0 | (0,71)  |           1 |       2304 |     24 |        |      
  72 |   6176 |        1 |     28 |   7061 |      0 |        0 | (0,72)  |           1 |       2304 |     24 |        |      
  73 |   6144 |        1 |     28 |   7061 |      0 |        0 | (0,73)  |           1 |       2304 |     24 |        |      
  74 |   6112 |        1 |     28 |   7061 |      0 |        0 | (0,74)  |           1 |       2304 |     24 |        |      
  75 |   6080 |        1 |     28 |   7061 |      0 |        0 | (0,75)  |           1 |       2304 |     24 |        |      
  76 |   6048 |        1 |     28 |   7061 |      0 |        0 | (0,76)  |           1 |       2304 |     24 |        |      
  77 |   6016 |        1 |     28 |   7061 |      0 |        0 | (0,77)  |           1 |       2304 |     24 |        |      
  78 |   5984 |        1 |     28 |   7061 |      0 |        0 | (0,78)  |           1 |       2304 |     24 |        |      
  79 |   5952 |        1 |     28 |   7061 |      0 |        0 | (0,79)  |           1 |       2304 |     24 |        |      
  80 |   5920 |        1 |     28 |   7061 |      0 |        0 | (0,80)  |           1 |       2304 |     24 |        |      
  81 |   5888 |        1 |     28 |   7061 |      0 |        0 | (0,81)  |           1 |       2304 |     24 |        |      
  82 |   5856 |        1 |     28 |   7061 |      0 |        0 | (0,82)  |           1 |       2304 |     24 |        |      
  83 |   5824 |        1 |     28 |   7061 |      0 |        0 | (0,83)  |           1 |       2304 |     24 |        |      
  84 |   5792 |        1 |     28 |   7061 |      0 |        0 | (0,84)  |           1 |       2304 |     24 |        |      
  85 |   5760 |        1 |     28 |   7061 |      0 |        0 | (0,85)  |           1 |       2304 |     24 |        |      
  86 |   5728 |        1 |     28 |   7061 |      0 |        0 | (0,86)  |           1 |       2304 |     24 |        |      
  87 |   5696 |        1 |     28 |   7061 |      0 |        0 | (0,87)  |           1 |       2304 |     24 |        |      
  88 |   5664 |        1 |     28 |   7061 |      0 |        0 | (0,88)  |           1 |       2304 |     24 |        |      
  89 |   5632 |        1 |     28 |   7061 |      0 |        0 | (0,89)  |           1 |       2304 |     24 |        |      
  90 |   5600 |        1 |     28 |   7061 |      0 |        0 | (0,90)  |           1 |       2304 |     24 |        |      
  91 |   5568 |        1 |     28 |   7061 |      0 |        0 | (0,91)  |           1 |       2304 |     24 |        |      
  92 |   5536 |        1 |     28 |   7061 |      0 |        0 | (0,92)  |           1 |       2304 |     24 |        |      
  93 |   5504 |        1 |     28 |   7061 |      0 |        0 | (0,93)  |           1 |       2304 |     24 |        |      
  94 |   5472 |        1 |     28 |   7061 |      0 |        0 | (0,94)  |           1 |       2304 |     24 |        |      
  95 |   5440 |        1 |     28 |   7061 |      0 |        0 | (0,95)  |           1 |       2304 |     24 |        |      
  96 |   5408 |        1 |     28 |   7061 |      0 |        0 | (0,96)  |           1 |       2304 |     24 |        |      
  97 |   5376 |        1 |     28 |   7061 |      0 |        0 | (0,97)  |           1 |       2304 |     24 |        |      
  98 |   5344 |        1 |     28 |   7061 |      0 |        0 | (0,98)  |           1 |       2304 |     24 |        |      
  99 |   5312 |        1 |     28 |   7061 |      0 |        0 | (0,99)  |           1 |       2304 |     24 |        |      
 100 |   5280 |        1 |     28 |   7061 |      0 |        0 | (0,100) |           1 |       2304 |     24 |        |      
 101 |   5248 |        1 |     28 |   7062 |      0 |        0 | (0,101) |       32769 |      10496 |     24 |        |      
 102 |   5216 |        1 |     28 |   7062 |      0 |        0 | (0,102) |       32769 |      10496 |     24 |        |      
 103 |   5184 |        1 |     28 |   7062 |      0 |        0 | (0,103) |       32769 |      10496 |     24 |        |      
 104 |   5152 |        1 |     28 |   7062 |      0 |        0 | (0,104) |       32769 |      10496 |     24 |        |      
 105 |   5120 |        1 |     28 |   7062 |      0 |        0 | (0,105) |       32769 |      10496 |     24 |        |      
 106 |   5088 |        1 |     28 |   7062 |      0 |        0 | (0,106) |       32769 |      10496 |     24 |        |      
 107 |   5056 |        1 |     28 |   7062 |      0 |        0 | (0,107) |       32769 |      10496 |     24 |        |      
 108 |   5024 |        1 |     28 |   7062 |      0 |        0 | (0,108) |       32769 |      10496 |     24 |        |      
 109 |   4992 |        1 |     28 |   7062 |      0 |        0 | (0,109) |       32769 |      10496 |     24 |        |      
(109 rows)


```







lp：这是插件自己定义的列，在源码中其实没有，这个是项指针的顺序

lp_off：tuple在page中的位置。

lp_flags：tuple的flags，具体为

```
#define LP_UNUSED       0       /* unused (should always have lp_len=0) */
#define LP_NORMAL       1       /* used (should always have lp_len>0) */
#define LP_REDIRECT     2       /* HOT redirect (should have lp_len=0) */
#define LP_DEAD         3       /* dead, may or may not have storage */

```

