[TOC]
# pg_rule

​	规则系统:讲真话真不懂什么意思，不过就是知道你查询这个语句，不是你真实查询的语句。

## SELECT 规则

- 视图语句创建

  ```
  create view vw_name as select column_name1,column_name2 ,... from table_name ...
  ```

  上面创建的语句等价于下面语句

- SELECT RULE 创建

  ```
  create table table_name_a as select *　from table_name where 1=2;
  CREATE RULE "_RETURN" AS ON SELECT TO table_name_a DO INSTEAD SELECT * FROM table_name;
  ```

- 备注

  1. SELECT 规则的后续操作只能是"INSTEAD SELECT"
  2. SELECT 规则名称只能是"_RETURN"

  	​	﻿

## 更新规则

- 更新语句

  ```
  osdba=# \h create rule
  Command:     CREATE RULE
  Description: define a new rewrite rule
  Syntax:
  CREATE [ OR REPLACE ] RULE name AS ON event
      TO table_name [ WHERE condition ]
      DO [ ALSO | INSTEAD ] { NOTHING | command | ( command ; command ... ) }
  
  where event can be one of:
  
      SELECT | INSERT | UPDATE | DELETE
  
  ```

  其中要除掉SELECT语句，发现有新的关键字ALSO, NOTHING

  ALSO:指的是不仅前者做，后面也要做。

  NOTHING:什么也不做

- INSERT ALSO|INSTEAD

  INSTEAD

  ```
  osdba=# CREATE TABLE my_180627 as select * from t_gh_180627 where 1=2;
  SELECT 0
  osdba=# CREATE RULE rule_t_gh_180627_insert AS ON INSERT TO t_gh_180627 DO INSTEAD insert into my_180627  values(new.id,new.name);
  CREATE RULE
  osdba=# truncate table t_gh_180627 ;
  TRUNCATE TABLE
  osdba=# truncate table my_180627 ;
  TRUNCATE TABLE
  osdba=# insert into t_gh_180627 select generate_series(1,100),'good';
  INSERT 0 100
  osdba=# select * from t_gh_180627 ;
   id | name 
  ----+------
  (0 rows)
  
  osdba=# select * from my_180627 ;
   id  | name 
  -----+------
     1 | good
     2 | good
     3 | good
     4 | good
  ```

  DROP RULE

  ```
  osdba=# drop rule rule_t_gh_180627_insert on t_gh_180627;
  DROP RULE
  osdba=# insert into t_gh_180627 values(1,'guohui');
  INSERT 0 1
  osdba=# select * from t_gh_180627 ;
   id |  name  
  ----+--------
    1 | guohui
  (1 row)
  
  osdba=# select count(1) from my_180627 ;
   count 
  -------
     100
  (1 row)
  
  osdba=# 
  
  ```

  ALSO

  ```
  osdba=# CREATE RULE rule_t_gh_180627_insert AS ON INSERT TO t_gh_180627 DO ALSO insert into my_180627  values(new.id,new.name);
  CREATE RULE
  osdba=# truncate table t_gh_180627 ;
  TRUNCATE TABLE
  osdba=# truncate table my_180627 ;
  TRUNCATE TABLE
  osdba=# insert into t_gh_180627 select generate_series(1,100),'good';
  INSERT 0 100
  osdba=# select count(1) from t_gh_180627 ;
   count 
  -------
     100
  (1 row)
  
  osdba=# select count(1) from my_180627 ;
   count 
  -------
     100
  (1 row)
  
  osdba=# 
  
  ```

  NOTHING

  ```
  osdba=# CREATE RULE rule_t_gh_180627_insert AS ON INSERT TO t_gh_180627 DO NOTHING;
  CREATE RULE
  osdba=# TRUNCATE TABLE t_gh_180627 ;
  TRUNCATE TABLE
  osdba=# truncate table my_180627 ;
  TRUNCATE TABLE
  osdba=# insert into t_gh_180627 select generate_series(1,100),'gggo';
  INSERT 0 100
  osdba=# select count(1) from t_gh_180627 ;
   count 
  -------
     100
  (1 row)
  
  osdba=# select count(1) from my_180627 ;
   count 
  -------
       0
  (1 row)
  
  osdba=# 
  
  ```

  以INSERT为例，分别查看ALSO,INSTEAD,NOTHING 三种状态下的数据情况。

  ALSO:前者执行，后者也执行

  INSTEAD:后者替代前者执行

  NOTHING:只做这件事

## 规则和权限

​	规则系统的权限定义

​	因规则而使用的关系对定义规则的属主进行权限检查，而不是检查执行规则的用户，这意味着一个用户只需要对查询里明确的表/视图拥有所需要的权限进行操作。

​       建立A表的规则去查询B表，首先建立这个规则的用户必须有查询B表的权利，否则无法查询(自己理解)。



## 规则和命令状态



​	postgresql对于每个命令会返回一个命令状态字符串，用来表示这条sql影响的行数。区别就在于有无where条件。



## 规则与触发器比较

​	从外部到数据的命令“copy from"会让触发器执行，不会调用规则

​	触发器比规则更容易理解

