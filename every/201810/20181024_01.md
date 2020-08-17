[TOC]

# gpload configure

**文档整理**

ysys

**日期**

20181024

**标签**

gpload,gpfdist,gpload configure



## 背景

​	喜欢将知识点能够细分的更新一点；gpload 部署已经完成，在正式使用之前需要做一些测试工作，测试gpload的安装是否准确，那么就要进行如下配置测试；这个其实更是了解gpload工作的一个途径，如果实在是不想测试这些流程，可以直接参考[kettle gpload](../20170504/kettle_greenplum_gpload.md)加载，两者是相通的。





## configure

### 启动gpfdist

```
# nohup /usr/local/greenplum-loaders-4.3.8.1-build-1/bin/gpfdist -d /home/admin/ -p 8888 > /tmp/gpfdist.log 2>&1 &
```

 	首先`/home`下必须要有`admin`目录才可以不报错，其次gpload的安装路径要在`/usr/local/greenplum-loaders-4.3.8.1-build-1`,查看启动是否报错`cat /tmp/gpfdist.log`

```
# cat /tmp/gpfdist.log 

nohup: ignoring input

2017-07-13 05:53:39 4729 INFO Before opening listening sockets - following listening sockets are available:

2017-07-13 05:53:39 4729 INFO IPV6 socket: [::]:8888

2017-07-13 05:53:39 4729 INFO IPV4 socket: 0.0.0.0:8888

2017-07-13 05:53:39 4729 INFO Trying to open listening socket:

2017-07-13 05:53:39 4729 INFO IPV6 socket: [::]:8888

2017-07-13 05:53:39 4729 INFO Opening listening socket succeeded

2017-07-13 05:53:39 4729 INFO Trying to open listening socket:

2017-07-13 05:53:39 4729 INFO IPV4 socket: 0.0.0.0:8888

Serving HTTP on port 8888, directory /home/admin
```



### 创建插入模板

```
# cd /usr/local/greenplum-loaders-4.3.8.1-build-1/bin/
# vim abc.yml

VERSION: 1.0.0.1

DATABASE: tutorial

USER: user1

HOST: 192.168.1.80

PORT: 5432

GPLOAD:

    INPUT:

    - SOURCE:

        LOCAL_HOSTNAME:

        - 192.168.1.18

        PORT: 8888

        FILE: ['/home/admin/test01.txt']

    - COLUMNS:

         - id:

         - name:

    - FORMAT: TEXT

    - DELIMITER: ','

    - QUOTE: ''

    - HEADER: FALSE

    - ENCODING: UTF8

    - ERROR_LIMIT: 50

    OUTPUT:

    - TABLE: "public.test001"

    - MODE: insert
```

​	在abc.yml中需要修改数据库名，ip地址，用户名；这个需要填写真是的信息

​	要在数据库中创建`test001`表，表的字段只有两个分别是`(id int,name varchar(100))`,而测试文件需要起的名为`test01.txt`,文件路径要在`/home/admin`下，文件里面的数据分布类似这样

```
1,gh
2,gk
3,g1
4,g2
```



### 执行测试脚本

```
# gpload -f abc.yml 

2017-07-13 07:12:01|INFO|gpload session started 2017-07-13 07:12:01

2017-07-13 07:12:01|INFO|started gpfdist -p 8888 -P 8889 -f "/home/admin/test01.txt" -t 30

2017-07-13 07:12:02|INFO|running time: 1.04 seconds

2017-07-13 07:12:02|INFO|rows Inserted = 7

2017-07-13 07:12:02|INFO|rows Updated = 0

2017-07-13 07:12:02|INFO|data formatting errors = 1

2017-07-13 07:12:02|INFO|gpload succeeded
```



## 错误

1、 nohup /usr/local/greenplum-loaders-4.3.8.1-build-1/bin/gpfdist -d /home/admin/ -p 8888 > /tmp/gpfdist.log 2>&1 &  执行报错

```
cat /tmp/gpfdist.log 

nohup: ignoring input

/usr/local/greenplum-loaders-4.3.8.1-build-1/bin/gpfdist: error while loading shared libraries: libyaml-0.so.1: cannot open shared object file: No such file or directory

[1]+ Exit 127 nohup /usr/local/greenplum-loaders-4.3.8.1-build-1/bin/gpfdist -d /home/admin/ -p 8888 > /tmp/gpfdist.log 2>&1
```

可能是环境变量设置有问题，重新执行`source greenplum_loaders_path.sh `



2、gpload -f abc.yml  no pg_hba.conf 执行报错

```
# gpload -f abc.yml 

2017-07-13 06:12:00|INFO|gpload session started 2017-07-13 06:12:00

2017-07-13 06:12:00|ERROR|could not connect to database: FATAL: no pg_hba.conf entry for host "192.168.1.18", user "user1", database "tutorial", SSL off

. Is the Greenplum Database running on port 5432?

2017-07-13 06:12:00|INFO|rows Inserted = 0

2017-07-13 06:12:00|INFO|rows Updated = 0

2017-07-13 06:12:00|INFO|data formatting errors = 0

2017-07-13 06:12:00|INFO|gpload failed
```

需要到数据库的主机上修改pg_hba.conf,允许这台服务器的ip地址可以访问

```
$ vim pg_hba.conf
$ gpstop -u
```



3、gpload -f abc.yml no privilege to create a readable gpfdist(s) external table  执行报错

```
# gpload -f abc.yml 
2017-07-13 06:23:34|INFO|gpload session started 2017-07-13 06:23:34
2017-07-13 06:23:34|INFO|started gpfdist -p 8888 -P 8889 -f "/home/admin/test01.txt" -t 30
2017-07-13 06:23:34|ERROR|could not run SQL "create external table ext_gpload_5356e268_67b5_11e7_9eb3_08002733502c(id integer,name character varying(128))location('gpfdist://etl:8889//home/admin/test01.txt') format'text' (delimiter ',' null '\\N' escape '\\' ) encoding'UTF8' segment reject limit 50 ": ERROR: permission denied: no privilege to create a readable gpfdist(s) external table

2017-07-13 06:23:34|INFO|rows Inserted = 0
2017-07-13 06:23:34|INFO|rows Updated = 0
2017-07-13 06:23:34|INFO|data formatting errors = 0
2017-07-13 06:23:34|INFO|gpload failed
```

需要到数据库的主机上修改postgresql.conf

```
gp_external_enable_exec = on   # enable external tables with EXECUTE.

gp_external_grant_privileges = on  #enable create http/gpfdist for non su's

```

4、安装完这些之后，在使用yum源可能就会报错，暂时无法解决

​	主要是因为使用了gpload的python依赖包，导致无法正常使用，因此建议先安装xterm包，可以使用xstart远程连接，或者先将`.bash_profile`的`source /usr/local/greenplum-loaders-4.3.8.1-build-1/ greenplum_loaders_path.sh `注销掉，就可以使用yum源了


