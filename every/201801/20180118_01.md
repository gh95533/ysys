[TOC]

# pg_一次性授权

## 背景

​	应用系统经常会访问一些表，可是将select any table 授权后，创建新表，依然需要授权，而目标表经常是几张几张的创建，每次创建完后，还要再次授权，非常麻烦，不适合一次性操作。

## 操作

### 要求

​	如果是按照操作具体来做，需要先确定如下几点：

​	1、查询与写入权限分离

​	该用户只能作为查询权限，不可以对表进行DML语句

​	2、数据写入用户必须具有创建schema的权限

### 步骤

​	说明A用户创建SCHEMA模式，B用户访问A用户的schema.table表

​	1、dba授权A用户创建DATABASE权限

​	本次是osdba用户具有dba权限

```
osdba=# create role learn1 login password 'learn1';
CREATE ROLE
osdba=# create role learn2 login password 'learn2';
CREATE ROLE
osdba=# \l
                               List of databases
    Name    | Owner | Encoding |   Collate   |    Ctype    | Access privileges 
------------+-------+----------+-------------+-------------+-------------------
 etl_0511   | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 etl_kettle | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 etl_pg     | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 gh_etl     | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 osdba      | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres   | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0  | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/osdba         +
            |       |          |             |             | osdba=CTc/osdba
 template1  | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/osdba         +
            |       |          |             |             | osdba=CTc/osdba
 tutorial   | osdba | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(9 rows)

osdba=# grant create on database tutorial to learn1;
GRANT

```



2、切换数据库用户创建schema并授权给learn2

```
osdba=# \c tutorial learn1;
You are now connected to database "tutorial" as user "learn1".
tutorial=> create schema sa;
CREATE SCHEMA
tutorial=> alter default privileges for role learn1 in schema sa grant select on tables to learn2;
ALTER DEFAULT PRIVILEGES
tutorial=> grant usage on schema sa to learn2;
GRANT
tutorial=> create table sa.gh_1(id int4,info text);
CREATE TABLE
tutorial=> insert into sa.gh_1 select generate_series(1,10),'guohui';
INSERT 0 10
tutorial=> select * from sa.gh_1;
 id |  info  
----+--------
  1 | guohui
  2 | guohui
  3 | guohui
  4 | guohui
  5 | guohui
  6 | guohui
  7 | guohui
  8 | guohui
  9 | guohui
 10 | guohui
(10 rows)
tutorial=> \c tutorial learn2;
You are now connected to database "tutorial" as user "learn2".
tutorial=> 
tutorial=> 
tutorial=> \dt
No relations found.
tutorial=> select * from sa.gh_1 ;
 id |  info  
----+--------
  1 | guohui
  2 | guohui
  3 | guohui
  4 | guohui
  5 | guohui
  6 | guohui
  7 | guohui
  8 | guohui
  9 | guohui
 10 | guohui
(10 rows)

tutorial=> 

```

注意：一定要先授权给用户，再开始创建表，否则第一次测试的表可能无法查询。

