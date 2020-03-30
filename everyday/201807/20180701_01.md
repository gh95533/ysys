[TOC]

# pg_dblink_fdw_比较

​	每个数据库默认都会有自己的dblink，就是不知道dblink除了访问和自己相同的数据库外，是否还可以访问其他类型的数据库，而这就是dblink和fdw在postgresql的区别。

​	

## dblink

### 创建extension

```
$ cd ${pg安装包}/contrib/dblink/
$ make
$ make install
[osdba@mysql45 dblink]$ psql
psql (9.4.1)
Type "help" for help.
osdba=# create extension dblink;
CREATE EXTENSION

```

### 使用dblink

```
osdba=# select * from dblink('host=localhost dbname=tutorial user=osdba password=osdba','select * from test01')as t(id int4,note text);
 id |   note   
----+----------
  1 | abcabefg
  2 | abxyz
  3 | 123abe
  4 | ab_abefg
  5 | ab%abefg
  6 | ababefg
(6 rows)

```

数据库链接很难跨越不同类型的数据库的链接，而fdw就不是如此了



## fdw



[pg_fdw整理](../20180625/readme.md)