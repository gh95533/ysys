[TOC]

# PG_TERMINATE_BACKEND('pid')与子进程关系

## 结论

​	pid就是子进程	



## 问题

​	在看德哥视频讲述pg的进程结构时，突然想到一个问题，pg_terminate_backend('pid')的pid是否就是父进程复制的子进程。

## 测试步骤

​	执行命令ps -ef|grep postgres查看当前活动的进程，通过pg_stat_activitity检查会话

```
[osdba@mysql45 ~]$ ps -ef|grep postgres
osdba     1964     1  0 08:03 pts/0    00:00:00 /usr/local/pgsql/bin/postgres
osdba     1965  1964  0 08:03 ?        00:00:00 postgres: logger process     
osdba     1967  1964  0 08:03 ?        00:00:09 postgres: checkpointer process   
osdba     1968  1964  0 08:03 ?        00:00:00 postgres: writer process     
osdba     1969  1964  0 08:03 ?        00:00:07 postgres: wal writer process   
osdba     1970  1964  0 08:03 ?        00:00:00 postgres: autovacuum launcher process   
osdba     1971  1964  0 08:03 ?        00:00:00 postgres: stats collector process   
osdba     2512  1938  0 08:37 pts/0    00:00:00 grep postgres
osdba=# select * from pg_stat_activity;
 datid | datname | pid  | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |  
       state_change          | waiting | state  | backend_xid | backend_xmin |              query              
-------+---------+------+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+--
-----------------------------+---------+--------+-------------+--------------+---------------------------------
 16384 | osdba   | 2516 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:37:31.502542+08 | 2018-07-20 08:37:44.735392+08 | 2018-07-20 08:37:44.735392+08 | 2
018-07-20 08:37:44.735395+08 | f       | active |             |         6933 | select * from pg_stat_activity;

[osdba@mysql45 ~]$ ps -ef|grep postgres
osdba     1964     1  0 08:03 pts/0    00:00:00 /usr/local/pgsql/bin/postgres
osdba     1965  1964  0 08:03 ?        00:00:00 postgres: logger process     
osdba     1967  1964  0 08:03 ?        00:00:09 postgres: checkpointer process   
osdba     1968  1964  0 08:03 ?        00:00:00 postgres: writer process     
osdba     1969  1964  0 08:03 ?        00:00:07 postgres: wal writer process   
osdba     1970  1964  0 08:03 ?        00:00:00 postgres: autovacuum launcher process   
osdba     1971  1964  0 08:03 ?        00:00:00 postgres: stats collector process   
osdba     2516  1964  0 08:37 ?        00:00:00 postgres: osdba osdba [local] idle
osdba     2523  1938  0 08:38 pts/0    00:00:00 grep postgres

```

​	通过上面示例可以得知，当启动一个会话或者连接时，postgres会folk子进程来与其对接。如果在2516进程上进行一个长sql:"insert into testtab_18_07_20 select generate_series(1,100000000),'info';",会有什么神奇情况。

```
osdba=# insert into testtab_18_07_20 select generate_series(1,100000000),'info';

[osdba@mysql45 ~]$ ps -ef|grep postgres
osdba     1964     1  0 08:03 pts/0    00:00:00 /usr/local/pgsql/bin/postgres
osdba     1965  1964  0 08:03 ?        00:00:00 postgres: logger process     
osdba     1967  1964  0 08:03 ?        00:00:09 postgres: checkpointer process   
osdba     1968  1964  0 08:03 ?        00:00:00 postgres: writer process     
osdba     1969  1964  0 08:03 ?        00:00:07 postgres: wal writer process   
osdba     1970  1964  0 08:03 ?        00:00:00 postgres: autovacuum launcher process   
osdba     1971  1964  0 08:03 ?        00:00:00 postgres: stats collector process   
osdba     2516  1964  1 08:37 ?        00:00:02 postgres: osdba osdba [local] INSERT
osdba     2534  1938  0 08:40 pts/0    00:00:00 grep postgres

重启一个会话查看活动的语句
osdba=# select * from pg_stat_activity;
 datid | datname | pid  | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |  
       state_change          | waiting | state  | backend_xid | backend_xmin |                                  query                                   
-------+---------+------+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+--
-----------------------------+---------+--------+-------------+--------------+--------------------------------------------------------------------------
 16384 | osdba   | 2516 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:37:31.502542+08 | 2018-07-20 08:40:48.5634+08   | 2018-07-20 08:40:48.5634+08   | 2
018-07-20 08:40:48.563402+08 | f       | active |        6933 |         6933 | insert into testtab_18_07_20 select generate_series(1,100000000),'info';
 16384 | osdba   | 2538 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:41:25.244387+08 | 2018-07-20 08:41:36.109527+08 | 2018-07-20 08:41:36.109527+08 | 2
018-07-20 08:41:36.10953+08  | f       | active |             |         6933 | select * from pg_stat_activity;
(2 rows)


```

在进程2516上不是idle而是insert

如果现在将执行终止命令:select pg_terminate_backend(2516);会发生什么情况？

```
osdba=# select pg_terminate_backend(2516);
 pg_terminate_backend 
----------------------
 t
(1 row)

在原本执行sql的会话

osdba=# insert into testtab_18_07_20 select generate_series(1,100000000),'info';
FATAL:  terminating connection due to administrator command
server closed the connection unexpectedly
	This probably means the server terminated abnormally
	before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.

查看postgres进程
osdba     1964     1  0 08:03 pts/0    00:00:00 /usr/local/pgsql/bin/postgres
osdba     1965  1964  0 08:03 ?        00:00:00 postgres: logger process     
osdba     1967  1964  0 08:03 ?        00:00:14 postgres: checkpointer process   
osdba     1968  1964  0 08:03 ?        00:00:01 postgres: writer process     
osdba     1969  1964  0 08:03 ?        00:00:12 postgres: wal writer process   
osdba     1970  1964  0 08:03 ?        00:00:00 postgres: autovacuum launcher process   
osdba     1971  1964  0 08:03 ?        00:00:00 postgres: stats collector process   
osdba     2538  1964  0 08:41 ?        00:00:00 postgres: osdba osdba [local] idle
osdba     2547  1964  0 08:43 ?        00:00:00 postgres: osdba osdba [local] idle
osdba     2549  1964  3 08:43 ?        00:00:03 postgres: autovacuum worker process   osdba
osdba     2555  1938  0 08:44 pts/0    00:00:00 grep postgres

```

发现2561进程已经消失了，而新出现的进程2547就是执行失败后产生的新进程，以及系统的vacuum产生的进程



```
osdba=# select * from pg_stat_activity;
 datid | datname | pid  | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |  
       state_change          | waiting | state  | backend_xid | backend_xmin |                   query                    
-------+---------+------+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+--
-----------------------------+---------+--------+-------------+--------------+--------------------------------------------
 16384 | osdba   | 2547 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:43:10.599066+08 |                               |                               | 2
018-07-20 08:43:11.055824+08 | f       | idle   |             |              | 
 16384 | osdba   | 2538 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:41:25.244387+08 | 2018-07-20 08:46:07.735192+08 | 2018-07-20 08:46:07.735192+08 | 2
018-07-20 08:46:07.735198+08 | f       | active |             |         6934 | select * from pg_stat_activity;
 16384 | osdba   | 2549 |       10 | osdba   |                  |             |                 |             | 2018-07-20 08:43:30.489722+08 | 2018-07-20 08:43:30.579553+08 | 2018-07-20 08:43:30.579553+08 | 2
018-07-20 08:43:30.57956+08  | f       | active |             |         6934 | autovacuum: VACUUM public.testtab_18_07_20
(3 rows)

osdba=# select * from pg_stat_activity;
 datid | datname | pid  | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |  
       state_change          | waiting | state  | backend_xid | backend_xmin |                   query                    
-------+---------+------+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+--
-----------------------------+---------+--------+-------------+--------------+--------------------------------------------
 16384 | osdba   | 2547 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:43:10.599066+08 | 2018-07-20 08:48:26.411478+08 | 2018-07-20 08:48:26.411478+08 | 2
018-07-20 08:48:26.411488+08 | f       | active |             |         6934 | select * from pg_stat_activity;
 16384 | osdba   | 2538 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:41:25.244387+08 |                               | 2018-07-20 08:46:07.735192+08 | 2
018-07-20 08:46:07.885597+08 | f       | idle   |             |              | select * from pg_stat_activity;
 16384 | osdba   | 2549 |       10 | osdba   |                  |             |                 |             | 2018-07-20 08:43:30.489722+08 | 2018-07-20 08:43:30.579553+08 | 2018-07-20 08:43:30.579553+08 | 2
018-07-20 08:43:30.57956+08  | f       | active |             |         6934 | autovacuum: VACUUM public.testtab_18_07_20
(3 rows)

```



测试一种新场景

就是手动kill 进程号，看看报错信息是否一致

还是在进程2547上执行一个长sql



```
osdba=# insert into testtab_18_07_20 select generate_series(1,100000000),'info';

osdba=# select * from pg_stat_activity;
 datid | datname | pid  | usesysid | usename | application_name | client_addr | client_hostname | client_port |         backend_start         |          xact_start           |          query_start          |  
       state_change          | waiting | state  | backend_xid | backend_xmin |                                  query                                   
-------+---------+------+----------+---------+------------------+-------------+-----------------+-------------+-------------------------------+-------------------------------+-------------------------------+--
-----------------------------+---------+--------+-------------+--------------+--------------------------------------------------------------------------
 16384 | osdba   | 2547 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:43:10.599066+08 | 2018-07-20 08:51:36.847466+08 | 2018-07-20 08:51:36.847466+08 | 2
018-07-20 08:51:36.847472+08 | f       | active |        6934 |         6934 | insert into testtab_18_07_20 select generate_series(1,100000000),'info';
 16384 | osdba   | 2538 |       10 | osdba   | psql             |             |                 |          -1 | 2018-07-20 08:41:25.244387+08 | 2018-07-20 08:51:40.696371+08 | 2018-07-20 08:51:40.696371+08 | 2
018-07-20 08:51:40.696376+08 | f       | active |             |         6934 | select * from pg_stat_activity;
 16384 | osdba   | 2549 |       10 | osdba   |                  |             |                 |             | 2018-07-20 08:43:30.489722+08 | 2018-07-20 08:43:30.579553+08 | 2018-07-20 08:43:30.579553+08 | 2
018-07-20 08:43:30.57956+08  | f       | active |             |         6934 | autovacuum: VACUUM public.testtab_18_07_20
(3 rows)


[osdba@mysql45 ~]$ ps -ef|grep postgres
osdba     1964     1  0 08:03 pts/0    00:00:00 /usr/local/pgsql/bin/postgres
osdba     1965  1964  0 08:03 ?        00:00:00 postgres: logger process     
osdba     1967  1964  0 08:03 ?        00:00:16 postgres: checkpointer process   
osdba     1968  1964  0 08:03 ?        00:00:01 postgres: writer process     
osdba     1969  1964  0 08:03 ?        00:00:13 postgres: wal writer process   
osdba     1970  1964  0 08:03 ?        00:00:00 postgres: autovacuum launcher process   
osdba     1971  1964  0 08:03 ?        00:00:00 postgres: stats collector process   
osdba     2538  1964  0 08:41 ?        00:00:00 postgres: osdba osdba [local] idle
osdba     2547  1964  2 08:43 ?        00:00:13 postgres: osdba osdba [local] INSERT
osdba     2549  1964  4 08:43 ?        00:00:22 postgres: autovacuum worker process   osdba
osdba     2588  1938  0 08:52 pts/0    00:00:00 grep postgres
[osdba@mysql45 ~]$ 

```

手动kill 

```
$ kill 2547
[osdba@mysql45 ~]$ ps -ef|grep postgres
osdba     1964     1  0 08:03 pts/0    00:00:00 /usr/local/pgsql/bin/postgres
osdba     1965  1964  0 08:03 ?        00:00:00 postgres: logger process     
osdba     1967  1964  0 08:03 ?        00:00:19 postgres: checkpointer process   
osdba     1968  1964  0 08:03 ?        00:00:01 postgres: writer process     
osdba     1969  1964  0 08:03 ?        00:00:14 postgres: wal writer process   
osdba     1970  1964  0 08:03 ?        00:00:00 postgres: autovacuum launcher process   
osdba     1971  1964  0 08:03 ?        00:00:00 postgres: stats collector process   
osdba     2538  1964  0 08:41 ?        00:00:00 postgres: osdba osdba [local] idle
osdba     2549  1964  4 08:43 ?        00:00:26 postgres: autovacuum worker process   osdba
osdba     2593  1964  0 08:53 ?        00:00:00 postgres: osdba osdba [local] idle
osdba     2595  1938  0 08:53 pts/0    00:00:00 grep postgres

osdba=# insert into testtab_18_07_20 select generate_series(1,100000000),'info';
FATAL:  terminating connection due to administrator command
server closed the connection unexpectedly
	This probably means the server terminated abnormally
	before or while processing the request.
The connection to the server was lost. Attempting reset: Succeeded.
osdba=# 
osdba=# select * from pg_stat_activity;
 datid | datname | pid  | usesysid | usename | application_name | client_addr | client_hostname | client_
port |         backend_start         |          xact_start           |          query_start          |   
      state_change          | waiting | state  | backend_xid | backend_xmin |                   query    
                
-------+---------+------+----------+---------+------------------+-------------+-----------------+--------
-----+-------------------------------+-------------------------------+-------------------------------+---
----------------------------+---------+--------+-------------+--------------+----------------------------
----------------
 16384 | osdba   | 2593 |       10 | osdba   | psql             |             |                 |        
  -1 | 2018-07-20 08:53:03.904405+08 |                               |                               | 20
18-07-20 08:53:04.662826+08 | f       | idle   |             |              | 
 16384 | osdba   | 2538 |       10 | osdba   | psql             |             |                 |        
  -1 | 2018-07-20 08:41:25.244387+08 | 2018-07-20 08:53:48.846555+08 | 2018-07-20 08:53:48.846555+08 | 20
18-07-20 08:53:48.846566+08 | f       | active |             |         6935 | select * from pg_stat_activ
ity;
 16384 | osdba   | 2549 |       10 | osdba   |                  |             |                 |        
     | 2018-07-20 08:43:30.489722+08 | 2018-07-20 08:43:30.579553+08 | 2018-07-20 08:43:30.579553+08 | 20
18-07-20 08:43:30.57956+08  | f       | active |             |         6934 | autovacuum: VACUUM public.t
esttab_18_07_20
(3 rows)


```



​	测试发现pg_terminate_backend(pid)和kill pid的效果是一样，那么pg_cancel_backend(pid)其实也是在进程上做文章，同时可以测试出pg_terminate_backend的pid就是子进程。