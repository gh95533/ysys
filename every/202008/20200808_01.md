[TOC]
# pg_列式存储



##  列式存储行式存储

行式存储：简单来说就是一条数据写到数据页里面

列式存储：简单来说就是一列数据写到数据页里面

具体参考文档:[列式存储行式存储](../20180629/列式存储行式存储.md)

##  搭建过程

###  环境介绍

- 操作系统版本

  centos_6.5_x64

- postgresql版本

  pg_9.4.12

- cstore_fdw

  cstore_fdw-1.6.0.zip

###  安装部署

####  解压cstore_fdw-1.6.0.zip并放到{PG}/contrib/

```
# mv cstore_fdw-1.6.0.zip  postgresql-9.4.12/contrib/
# cd postgresql-9.4.12/contrib/
# unzip cstore_fdw-1.6.0.zip 
# chown -R osdba:osdba cstore_fdw-1.6.0
# chown -R osdba:osdba cstore_fdw-1.6.0/
```

####  安装centos依赖包

#####  EPEL(extra packages for Enterprise Linux)

文档:[EPEL_使用](EPEL_使用.md)

#####  yum install protobuf-c-devel

```
# yum install protobuf-c-devel
```

在默认安装包镜像包中并没有protobuf-c相关的依赖包，需要通过EPEL方式来获取，在yum时，发现安装的依赖包有如下一下，或者可以通过其他方式将这些安装包下载并使用

```
(1/4): protobuf-2.3.0-9.el6.x86_64.rpm                                            | 289 kB      
(2/4): protobuf-c-0.15-2.el6.x86_64.rpm                                           |  93 kB     
(3/4): protobuf-c-devel-0.15-2.el6.x86_64.rpm                                     |  12 kB     
(4/4): protobuf-compiler-2.3.0-9.el6.x86_64.rpm                                   | 190 kB       

```

####  编译安装(make make install)

```
$ cd {PG}/contrib/cstore_fdw-1.6.0
$ make
$ make install
```

####  修改postgresql.conf文件

```
$ vim postgresql.conf

shared_preload_libraries = 'cstore_fdw'         # (change requires restart)

$ pg_ctl stop -m fast
$ pg_ctl start &
```

####  下载数据

```
wget http://examples.citusdata.com/customer_reviews_1998.csv.gz
wget http://examples.citusdata.com/customer_reviews_1999.csv.gz

gzip -d customer_reviews_1998.csv.gz
gzip -d customer_reviews_1999.csv.gz
```

####  创建 extension

```
osdba=# create extension cstore_fdw;
CREATE EXTENSION
```

####  创建SERVER

```
osdba=# CREATE SERVER cstore_server FOREIGN DATA WRAPPER cstore_fdw;
CREATE SERVER
```

####  创建外部表

```
CREATE FOREIGN TABLE customer_reviews
(
    customer_id TEXT,
    review_date DATE,
    review_rating INTEGER,
    review_votes INTEGER,
    review_helpful_votes INTEGER,
    product_id CHAR(10),
    product_title TEXT,
    product_sales_rank BIGINT,
    product_group TEXT,
    product_category TEXT,
    product_subcategory TEXT,
    similar_product_ids CHAR(10)[]
)
SERVER cstore_server
OPTIONS(compression 'pglz');
```

####  插入数据并查看空间大小

```
osdba=# COPY customer_reviews FROM '/home/osdba/customer_reviews_1998.csv' WITH CSV;
COPY 589859
osdba=# COPY customer_reviews FROM'/home/osdba/customer_reviews_1999.csv' WITH CSV;
COPY 1172645
osdba=# select count(1) from customer_reviews ;
  count  
---------
 1762504
(1 row)
osdba=# \x
Expanded display is on.
osdba=# select * from customer_reviews limit 10;
-[ RECORD 1 ]--------+---------------------------------------------------------
customer_id          | AE22YDHSBFYIP
review_date          | 1970-12-30
review_rating        | 5
review_votes         | 10
review_helpful_votes | 0
product_id           | 1551803542
product_title        | Start and Run a Coffee Bar (Start & Run a)
product_sales_rank   | 11611
product_group        | Book
product_category     | Business & Investing
product_subcategory  | General
similar_product_ids  | {0471136174,0910627312,047112138X,0786883561,0201570483}

osdba=# create table t_gh_0628 as select * from customer_reviews ;
SELECT 1762504
                                               ^
osdba=# select pg_size_pretty(pg_relation_size('t_gh_0628'));
 pg_size_pretty 
----------------
 413 MB
(1 row)

osdba=# select pg_size_pretty(pg_relation_size('customer_reviews'));
 pg_size_pretty 
----------------
 0 bytes
(1 row)
直接使用命令无法查看customer_reviews的大小，查看$PGDATA/cstore_fdw的文件信息大小为
110M,差距是差不过的4倍左右。
```



##  参考链接

https://github.com/citusdata/cstore_fdw

https://pgxn.org/dist/cstore_fdw/

http://lib.csdn.net/article/datastructure/8951

https://www.cnblogs.com/qiaoyihang/p/6262806.html

<http://code.google.com/p/protobuf/>

https://yq.aliyun.com/articles/18042

http://blogs.ugidotnet.org/sqlog/archive/2015/08/24/postgresql-cstore_fdw.aspx







