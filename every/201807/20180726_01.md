[TOC]

# PG 瞎搞 删除clog日志



​	最近一段时间在看postgresql数据库中wal,dbfile,clog以及walwrite,backgroudwrite,checkpoint之前的关系，突发奇想，如果上一个checkpoint结束后，下一个checkpoint没有到来之前，关闭数据库，删除clog日志，对数据库的影响，对于数据的影响。



## 试验开始

1. 重启数据库，并且插入10000条数据后，执行checkpoint

   ```
   tutorial=# insert into test2 select generate_series(1,10000);
   INSERT 0 10000
   tutorial=# checkpoint;
   CHECKPOINT
   tutorial=# update test2 set id= 1 where id = 10000;
   UPDATE 1
   tutorial=# select * from test2 where id = 10000;
    id 
   ----
   (0 rows)
   
   tutorial=# select * from test2 where id = 1;
    id 
   ----
     1
     1
   (2 rows)
   
   ```

2. 关闭数据库，将pg_clog/0000移动到其他地方

   ```
   [osdba@mysql45 data]$ cd pg_clog
   [osdba@mysql45 pg_clog]$ ls
   0000
   [osdba@mysql45 pg_clog]$ mv 0000 ../pg_log/
   [osdba@mysql45 pg_clog]$ ls -ls
   total 0
   [osdba@mysql45 pg_clog]$ 
   
   ```

3. 重新开启数据库,查看日志

   ```
   [osdba@mysql45 data]$ pg_ctl start
   server starting
   [osdba@mysql45 data]$ LOG:  redirecting log output to logging collector process
   HINT:  Future log output will appear in directory "pg_log".
   ^C
   [osdba@mysql45 data]$ psql
   psql: could not connect to server: No such file or directory
   	Is the server running locally and accepting
   	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
   [osdba@mysql45 pg_log]$ tail -200f postgresql-2018-07-26_082150.csv 
   2018-07-26 08:21:50.514 CST,,,2345,,5b59141e.929,1,,2018-07-26 08:21:50 CST,,0,LOG,00000,"ending log output to stderr",,"Future log output will go to log destination ""csvlog"".",,,,,,,""
   2018-07-26 08:21:50.516 CST,,,2347,,5b59141e.92b,1,,2018-07-26 08:21:50 CST,,0,LOG,00000,"database system was shut down at 2018-07-26 08:19:16 CST",,,,,,,,,""
   2018-07-26 08:21:50.519 CST,,,2347,,5b59141e.92b,2,,2018-07-26 08:21:50 CST,,0,FATAL,58P01,"could not access status of transaction 7237","Could not open file ""pg_clog/0000"": No such file or directory.",,,,,,,,""
   2018-07-26 08:21:50.519 CST,,,2345,,5b59141e.929,2,,2018-07-26 08:21:50 CST,,0,LOG,00000,"startup process (PID 2347) exited with exit code 1",,,,,,,,,""
   2018-07-26 08:21:50.519 CST,,,2345,,5b59141e.929,3,,2018-07-26 08:21:50 CST,,0,LOG,00000,"aborting startup due to startup process failure",,,,,,,,,""
   
   ```

   现在数据库启动不起来，那么迁移会之前的0000文件是否可行，答案是可行的

   

4.迁移后之前文件，查看是否可以启动

```

[osdba@mysql45 pg_log]$ mv 0000 ../pg_clog/
[osdba@mysql45 pg_log]$ cd ../pg_clog
[osdba@mysql45 pg_clog]$ ls
0000
[osdba@mysql45 pg_clog]$ pg_ctl start 
server starting
[osdba@mysql45 pg_clog]$ LOG:  redirecting log output to logging collector process
HINT:  Future log output will appear in directory "pg_log".
^C
[osdba@mysql45 pg_clog]$ psql
psql (9.4.1)
Type "help" for help.

osdba=# 

```

​	假设文件不存在，或者不知道在那里的话。



## 尝试方法pg_resetxlog

```
[osdba@mysql45 bin]$ pg_resetxlog -f $PGDATA
Transaction log reset
[osdba@mysql45 bin]$ pg_ctl start 
server starting
[osdba@mysql45 bin]$ LOG:  redirecting log output to logging collector process
HINT:  Future log output will appear in directory "pg_log".
^C
[osdba@mysql45 bin]$ psql
psql: could not connect to server: No such file or directory
	Is the server running locally and accepting
	connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
[osdba@mysql45 bin]$ 

```

这个方案不可行。



   







































## 链接地址

https://blog.csdn.net/Linzhongyilisha/article/details/79758920

http://b120702.blog.163.com/blog/static/208604206201661192752545/

https://blog.csdn.net/spche/article/details/5997754

https://blog.csdn.net/lk_db/article/details/78376500