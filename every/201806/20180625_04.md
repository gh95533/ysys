
## pg_mysql_外部表

- 环境

本次测试环境是在同一台服务器，它的信息分别是（centos6.5_x64,1.5g内存，1核cpu,30g磁盘空间）,软件（mysql5.7.17，postgres 9.4.1)

- 测试发现

mysql_fdw-2.1.2.zip不匹配postgresql_9.6.5,测试中使用的是postgresql_9.4.12

### mysql_fdw 安装步骤

mysql_fdw文件依赖mysql的lib包，为了更好的安装部署mysql_fdw,建议在本地安装一套mysql数据库，或者拷贝测试通过压缩包(就是将mysql数据库安装完成后重新打的包)

本次将以压缩包为例讲解

- 下载mysql压缩包

链接：https://pan.baidu.com/s/18Ld-tIdj4gdGy0EjuUxHpQ 密码：v8l1

- 将压缩包上传到/software后，解压文件，移动到/usr/local/mysql下

```

# tar -xvf mysql_postgres_lib.tar 

# mv mysql /usr/local/

# chown -R osdba:osdba mysql


```

- 修改root和postgres环境变量

```
# cd ~

# vim .bash_profile

export MYSQLHOME=/usr/local/mysql
export PGHOME=/usr/local/pgsql
export LD_LIBRARY_PATH=$PGHOME/lib:$MYSQLHOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/lib:/usr/lib:/usr/local/lib
export PATH=$PGHOME/bin:$MYSQLHOME/bin:$PATH:.


# source .bash_profile

# su - osdba(postgres安装用户)

$ cd ~

$ vim .bash_profile

export MYSQLHOME=/usr/local/mysql
export PGHOME=/usr/local/pgsql
export PGDATA=$PGHOME/data
export LD_LIBRARY_PATH=$PGHOME/lib:$MYSQLHOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/lib:/usr/lib:/usr/local/lib
export PATH=$PGHOME/bin:$MYSQLHOME/bin:$PATH:.

$ source .bash_profile

$ exit

# source /home/osdba/.bash_profile


```

- 上传mysql_fdw-2.1.2.zip并在postgresql安装包/contrib/

```

# unzip mysql_fdw-2.1.2.zip 

# chown -R osdba:osdba mysql_fdw-2.1.2

# su - osdba

$ cd {postgres}/contrib/mysql_fdw-2.1.2

$ make 

$ make install


```

- 创建extension

```
create extension mysql_fdw;

```
- 创建server

```

CREATE SERVER mysql_server FOREIGN DATA WRAPPER mysql_fdw OPTIONS (host '192.168.1.45', port '3306'); 

```

- 创建user

```
CREATE USER MAPPING FOR osdba SERVER mysql_server OPTIONS (username 'root', password '123456'); 

```

- 创建外部表

```
CREATE FOREIGN TABLE for_mysql_t_etl_trans_exp( trans_exp_id varchar(32),trans_id varchar(32),
exp_node_id varchar(32)) SERVER mysql_server OPTIONS (dbname 'etl', table_name 't_etl_trans_exp');

```

一般在创建过程中都不会报错(数据库链接问题)，只有真正查询才有可能就报错，查看具体报错来具体处理问题。



### error

- 无法创建extension 

```
 [osdba@gh52 ~]$ psql
psql (9.4.12)
Type "help" for help.

osdba=# create extension mysql_fdw;
ERROR:  failed to load the mysql query: 
libmysqlclient.so: cannot open shared object file: No such file or directory
HINT:  export LD_LIBRARY_PATH to locate the library

```

建议重新启动postgresql数据库

