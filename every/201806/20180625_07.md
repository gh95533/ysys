# pg_redis_外部表



​	redis是15年出差时在青海听到的数据，传说相当快，本次通过pg_redis_外部表来看看redis的数据。



- 版本

  由于当前公司产品使用的数据库都是postgresql9.4.12,本次测试都是依次为基础的。redis版本测试环境使用的是4.0.9，正式环境是3.2.100﻿

- 文件下载

  链接：https://pan.baidu.com/s/1Q01AUvLVaf-aOn6NZYiCRQ 密码：7gvr

  链接：https://pan.baidu.com/s/15eRH6PWWUI_5eJMfKIMELg 密码：w7md

- **安装**

  建议将redis_fdw-REL9_4_STABLE.zip 和hiredis-master.zip包放到/home/<user#/

  ```
  
  #unzip redis_fdw-REL9_4_STABLE.zip 
  #unzip hiredis-master.zip 
  
  #mv hiredis-master hiredis
  #cd hiredis
  #make
  #make install
  
  # cd redis_fdw-REL9_4_STABLE
  # make USE_PGXS=1
  # make USE_PGXS=1 install
  
  # su - osdba
  $ vim .bash_profile
  
  export PGHOME=/usr/local/pgsql
  export PATH=$PGHOME/bin/:$PATH
  export PGDATA=$PGHOME/data
  export LD_LIBRARY_PATH=$PGHOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/lib:/usr/lib:/usr/local/lib:$LD_LIBRARY_PATH
  
  $ source .bash_profile
  ```

- 创建extension,表

  ```
  osdba=# create extension redis_fdw;
  osdba=# create server redis_server_49 FOREIGN DATA WRAPPER redis_fdw OPTIONS (address '192.168.1.49',port'6379');
  osdba=# CREATE USER MAPPING FOR osdba SERVER redis_server_49  OPTIONS (password 'foobared');
  osdba=# CREATE FOREIGN TABLE redis_db0(key text,val text)SERVER redis_server_49 OPTIONS (database '0');
  osdba=# CREATE FOREIGN TABLE redis_hash_cs(key text ,val text[]) SERVER redis_server_49 OPTIONS( database '0' ,tabletype 'hash');
  
  osdba=# select * from redis_hash_cs ;
    key  |                 val                  
  -------+--------------------------------------
   db1   | {good2,dfdfd,good3,dfdfd,good4,fdfd}
   db11  | {dfdaa,dafda}
   db111 | {dfdf,dfadafd}
  (3 rows)
  
  osdba=# select * from redis_hash_cs where val @>'{dfdaa}';
   key  |      val      
  ------+---------------
   db11 | {dfdaa,dafda}
  ```

  redis数据映射到postgresql外部表时，val变成数据，所以”@>”就是数组的操作函数，表示包含的意思。

  

  感谢:石敦斌 多度 dba

 

### 参考链接

<http://blog.163.com/digoal@126/blog/static/16387704020119181188247/>

<http://blog.163.com/digoal@126/blog/static/163877040201622532547588>