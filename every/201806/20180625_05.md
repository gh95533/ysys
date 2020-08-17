
## pg_oracle_外部表

部署完file_fdw,postgres_fdw，mysql_fdw,现在部署oracle_fdw，其实oracle_fdw部署类似mysql_fdw，主要是需要依赖oracle的jar包，考虑到一些包可能就导致安装失败，就将oracle的安装后整体打包。

- 当前版本

	pg:9.4.12 oracle_fdw:oracle_fdw-2.0.0.zip

- 下载oracle压缩包

链接：https://pan.baidu.com/s/16A0OMot4-FSsV6MJRWxfaQ 密码：2ecr

- 解压并且将路径定义到oracle原始安装目录下(本次地址为/u01下)


```
# tar -xvf oracle_postgres_lib.tar 
# mv u01 /
# chown -R osdba:osdba /u01
# chown -R osdba:osdba /u01/
```

- 修改环境变量

```

# su - osdba
$ vim .bash_profile

export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/11.2.0/db_1
export MYSQLHOME=/usr/local/mysql
export PGHOME=/usr/local/pgsql
export PGDATA=$PGHOME/data
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$PGHOME/lib:$MYSQLHOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/lib:/usr/lib:/usr/local/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PGHOME/bin:$MYSQLHOME/bin:$PATH:

$ source .bash_profile

```
注意LD_LIBRARY_PATH的最后$LD_LIBRARY_PATH，一般环境变量都没有带上，但是当前环境需要(我也不知道为什么)

- 将oracle_fdw上传{PG}/contrib/下编译oracle_fdw

```
# unzip oracle_fdw-2.0.0.zip 
# chown -R osdba:osdba oracle_fdw-2.0.0
# chown -R osdba:osdba oracle_fdw-2.0.0/

# su - osdba
$ cd {PG}/contrib/oracle_fdw-2.0.0
$ make
$ make install
```



- 创建extension

```
create extension oracle_fdw

```

  

- 创建server

```
CREATE SERVER ora11 FOREIGN DATA WRAPPER oracle_fdw OPTIONS (dbserver 			'//192.168.1.11:1521/orcl');

```

  

- 创建user

```
CREATE USER MAPPING FOR postgres SERVER ora11 OPTIONS ( user 'etl' , password 'etl');

```

- 创建外部表

```
CREATE FOREIGN TABLE ft_ora_userinfo(id int,info text,crt_time timestamp(0)) SERVER ora11 OPTIONS(schema 'ETL', table 'USERINFO');
  
select * from ft_ora_userinfo;

```

  

## error

- 无法创建extension

```
osdba=# create extension oracle_fdw;
ERROR:  could not load library "/usr/local/pgsql/lib/oracle_fdw.so": libclntsh.so.11.1: cannot open shared object file: No such file or directory
    
```

解决方案：重新启动数据库，再次运行即刻。

​    






