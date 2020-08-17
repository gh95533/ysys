## pg_mongo_外部表

​	之前是在天津出差时，研发组讨论动态创建字段的数据库选型时选择mongo，而postgresql和mongo关于动态选型都是基于jsonb类型来完成的。

​	到底是pg好，还是mongo好，请期待后续章节

-  版本

		pg:9.4.12 mongo:﻿3.6

-  下载并且安装依赖包

  链接：https://pan.baidu.com/s/1UbzhQMymJ7w3qoYeNSj4Zg 密码：xx7c

- 解压master,并将解压包移动到源码的解压包contrib

```
[root@gh51 postgresql-9.4.12]# cd /home/osdba/
[root@gh51 osdba]# chown -R osdba:osdba master 
[root@gh51 osdba]# chmod 777 master 
[root@gh51 osdba]# su - osdba
[osdba@gh51 ~]$ ls
master
[osdba@gh51 ~]$ unzip master 
Archive:  master
[osdba@gh51 ~]$ mv mongo_fdw-master /software/postgresql-9.4.12/contrib/
```

- make && install mongo

```
[root@gh51 mongo_fdw-master]$ cd /software/postgresql-9.4.12/contrib/
[root@gh51 contrib]# chown -R osdba:osdba mongo_fdw-master
[root@gh51 contrib]# chown -R osdba:osdba mongo_fdw-master/
[root@gh51 contrib]# su - osdba
[osdba@gh51 mongo_fdw-master]$ cd /software/postgresql-9.4.12/contrib/mongo_fdw-master/
[osdba@gh51 mongo_fdw-master]$ ls -ls
total 104
 4 -rw-rw-r-- 1 osdba osdba  2895 Apr  1  2016 CONTRIBUTING.md
 8 -rw-rw-r-- 1 osdba osdba  7632 Apr  1  2016 LICENSE
 4 -rw-rw-r-- 1 osdba osdba   961 Apr  1  2016 Makefile
 4 drwxrwxr-x 5 osdba osdba  4096 Apr  1  2016 mongo-c-driver-v0.6
 4 -rw-rw-r-- 1 osdba osdba   520 Apr  1  2016 mongo_fdw--1.0.sql
44 -rw-rw-r-- 1 osdba osdba 44679 Apr  1  2016 mongo_fdw.c
 4 -rw-rw-r-- 1 osdba osdba   202 Apr  1  2016 mongo_fdw.control
 4 -rw-rw-r-- 1 osdba osdba  3635 Apr  1  2016 mongo_fdw.h
16 -rw-rw-r-- 1 osdba osdba 15834 Apr  1  2016 mongo_query.c
 4 -rw-rw-r-- 1 osdba osdba   386 Apr  1  2016 mongo_query.h
[osdba@gh51 mongo_fdw-master]$ make
[osdba@gh51 mongo_fdw-master]$ make install
/bin/mkdir -p '/usr/local/pgsql/lib'
/bin/mkdir -p '/usr/local/pgsql/share/extension'
/bin/mkdir -p '/usr/local/pgsql/share/extension'
/usr/bin/install -c -m 755  mongo_fdw.so '/usr/local/pgsql/lib/mongo_fdw.so'
/usr/bin/install -c -m 644 mongo_fdw.control '/usr/local/pgsql/share/extension/'
/usr/bin/install -c -m 644 mongo_fdw--1.0.sql '/usr/local/pgsql/share/extension/'

```

- mongo_fdw

```
[osdba@gh51 mongo_fdw-master]$ psql
psql (9.4.12)
Type "help" for help.

osdba=# create extension mongo_fdw;
CREATE EXTENSION

osdba=# create server mongo_server_49 FOREIGN DATA WRAPPER mongo_fdw 
OPTIONS(address '192.168.1.49',port '11111');
CREATE SERVER


osdba=# create foreign table foreign_col(title text,description text) server mongo_server_49 OPTIONS( database 'test',collection 'col');
CREATE FOREIGN TABLE
osdba=# select * from foreign_col ;
    title     |         description         
--------------+-----------------------------
 MongoDB 教程 | MongoDB 是一个 Nosql 数据库
 MongoDB 教程 | MongoDB 是一个 Nosql 数据库
 guohui       | 
 guosex       | 
 fddd         | 
 fdddddd      | 
```



#### 遗留问题：

- 正式环境中MONGODB的字段项大多是是大写字母，这样的话，创建表就需要下面语句

 ```
Create table sss (“ABC” text);
 ```



### 参考链接:

<https://pgxn.org/dist/mongo_fdw/>

<http://blog.163.com/digoal@126/blog/static/163877040201321984940903/>