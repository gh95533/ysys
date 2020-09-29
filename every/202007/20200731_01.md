
## pg_postgres_外部表

pg和pg跨库查询，既可以通过dblink,也可以通过postgres_外部表，本章节先讲一下postgres_外部表

### 配置步骤

本身就是postgresql数据库，所以对于自己的支持还是比较大，建议使用源码安装，这样就可以想file_fdw执行一样简单

- 编译postgres_fdw

```

$ cd postgres_fdw

$ make

$ make install

```

- 创建extension

```
osdba=# create extension postgres_fdw;
CREATE EXTENSION
```

- 创建server

```

osdba=# CREATE SERVER postgres_fdw_52 FOREIGN DATA WRAPPER postgres_fdw OPTIONS(host '192.168.1.52',dbname 'tutorial',port '5432');
CREATE SERVER


```

- 创建user

```
osdba=# CREATE USER MAPPING FOR osdba SERVER postgres_fdw_52 OPTIONS(user 'osdba' ,password 'xxxxxxxx');
CREATE USER MAPPING

```

- 创建foreign table 

```
osdba=# CREATE FOREIGN TABLE pg_fdw_t_gh_cs(id int,info text,rksj timestamp(0)) SERVER postgres_fdw_52  OPTIONS( table_name 't_gh_cs');
CREATE FOREIGN TABLE

```

- 查询

```

osdba=# select * from pg_fdw_t_gh_cs ;
  id  | info |        rksj         
------+------+---------------------
    1 | good | 2018-06-27 06:24:24
    2 | good | 2018-06-27 06:24:24
    3 | good | 2018-06-27 06:24:24
    4 | good | 2018-06-27 06:24:24

```

pg_postgres_fdw和pg_file_fdw安装部署较为简单，且无需担心版本不兼容问题

