# pg_物化视图





​	pg物化视图支持的版本是9.x系列，所以在9以下就不能使用物化视图来做一些﻿优化查询。

​	

- pg_物化视图

  物化视图在我认为等于数据不需要做一些抽取作业，只要更新一下语句，就可以数据变为最新，非常适合数据变动不是很频繁

- 创建语句

  ```
  \h create materialize view
  Command:     CREATE MATERIALIZED VIEW
  Description: define a new materialized view
  Syntax:
  CREATE MATERIALIZED VIEW [ IF NOT EXISTS ] table_name
      [ (column_name [, ...] ) ]
      [ WITH ( storage_parameter [= value] [, ... ] ) ]
      [ TABLESPACE tablespace_name ]
      AS query
      [ WITH [ NO ] DATA ]
  
  ```

  

  1、有存储数据并且可以选择表空间

  

  2、可以创建索引,唯一索引也是可以创建，但是主键索引不允许创建

  

  3、with no data表示数据结构建立，数据暂时没有，如果不同步数据，查询物化视图会报错

  

- 刷新语句

  ```
  \h refresh
  Command:     REFRESH MATERIALIZED VIEW
  Description: replace the contents of a materialized view
  Syntax:
  REFRESH MATERIALIZED VIEW [ CONCURRENTLY ] name
      [ WITH [ NO ] DATA ]
  ```

  

  全量刷新模板：REFRESH MATERIALIZED VIEW ma_view WITH DATA

  

  在线刷新模板：REFRESH MATERIALIZED VIEW CONCURRENTLY  ma_view WITH DATA

  

  pg_物化视图有两种更新方式

  

  1、全量更新

  

  全量更新较快，但是在更新时阻塞数据查询

  

  2、增量更新

  

  增量更新较慢，但是不阻塞数据查询,而且增量更新必须要求唯一键

  

  ***<u>在9.3中只有全量刷新，在9.4后增加了在线刷新功能</u>***

  

- 测试demo

  

  通过外部表我们可以访问不属于本地的数据，但是外部表不保存数据，如果对方突然删除或者挂掉我们就没有数据使用了，因此使用物化视图成为我们获取数据，更新数据一种比较好的方式(不用写一下sql来更新删除数据，不是很好吗)

  

  ```
  创建一个测试数据库，用户，表
  
  postgres=# create database tutorial;
  postgres=# create user study login password 'study';
  
  切换到新数据库，新用户创建表，插入测试数据
  
  tutorial=> create table test_tab(id int,info text,times timestamp(0));
  tutorial=> insert into test_tab select series_generate(1,100000),md5(random()::text),now();
  
  在数据库gh,用户postgres下创建外部数据包装器，外部数据库，外部用户映射，外部表
  
  gh=# create extension postgres_fdw;
  
  gh=# create server pg96_tutorial_server FOREIGN DATA WRAPPER postgres_fdw OPTIONS(host '192.168.1.12', dbname 'tutorial',port '5432');
  
  gh=# create user mapping for postgres SERVER pg96_tutorial_server OPTIONS (user 'study',password 'study');
  
  gh=#create foreign table for_pg96_tuto_tab(id int,info text,times timestamp(0)) server pg96_tutorial_server OPTIONS (schema_name 'public',table_name 'test_tab');
  
  
  gh=# select count(1) from for_pg96_tuto_tab ;
  
  
  gh=# create materialized view mv_pg_fdw_tab as select * from for_pg96_tuto_tab with no data;
  
  gh=# create unique index un_pg_fdw_tab_id on mv_pg_fdw_tab (id);
  
  
  gh=# refresh materialized view  mv_pg_fdw_tab with data;
  
  只有第一次完整的同步完物化视图，才能执行增量同步
  
  gh=# refresh materialized view CONCURRENTLY mv_pg_fdw_tab ;
  
  
  ```

  

  ​	









