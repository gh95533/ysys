[TOC]

# pg dba 6

​	

​	本章节主要讲述的是连接池及数据库高速缓存



## 连接池



### 短连接长连接tps

#### 短连接tps

```
[osdba@mysql45 ~]$ cat test.sql
select 1;
[osdba@mysql45 ~]$ pgbench -M extended -n -r -f ./test.sql -c 16 -j 4 -C -T 30
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 11662
latency average: 41.159 ms
tps = 388.651729 (including connections establishing)
tps = 135108.207053 (excluding connections establishing)
statement latencies in milliseconds:
	30.871288	select 1;
[osdba@mysql45 ~]$ 



# sar -w 1 100000
06:22:25 AM      1.01    104.04
06:22:26 AM      0.00     71.00
06:22:27 AM      0.00     94.95
06:22:28 AM    389.90   7618.18
06:22:29 AM    460.61   9070.71
06:22:30 AM    394.95   7926.26
06:22:31 AM    407.07   8241.41
06:22:32 AM    311.46   6239.58
06:22:33 AM    386.46   7741.67
06:22:34 AM    384.85   7783.84
06:22:35 AM    394.95   7871.72
06:22:36 AM    383.33   7734.44
06:22:37 AM    412.00   8223.00
06:22:38 AM    378.12   7631.25

06:22:38 AM    proc/s   cswch/s
06:22:39 AM    373.47   7502.04
06:22:40 AM    414.14   8301.01
06:22:41 AM    389.00   7790.00
06:22:42 AM    356.12   7200.00
06:22:43 AM    433.66   8646.53
06:22:44 AM    380.00   7656.00
06:22:45 AM    426.26   8601.01
06:22:46 AM    398.99   8042.42
06:22:47 AM    328.28   6621.21
06:22:48 AM    414.14   8260.61
06:22:49 AM    423.23   8408.08
06:22:50 AM    423.76   8435.64
06:22:51 AM    370.71   7438.38
06:22:52 AM    382.65   7711.22
06:22:53 AM    425.25   8517.17
06:22:54 AM    321.21   6538.38
06:22:55 AM    408.00   8155.00
06:22:56 AM    407.07   7994.95
06:22:57 AM    400.00   8091.30
06:22:58 AM     81.00   1808.00


```



#### 长连接tps

```
[osdba@mysql45 ~]$ pgbench -M extended -n -r -f ./test.sql -c 16 -j 4  -T 30
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 551789
latency average: 0.870 ms
tps = 18386.280802 (including connections establishing)
tps = 18419.581888 (excluding connections establishing)
statement latencies in milliseconds:
	0.866824	select 1;



# sar -w 1 1000000
06:24:40 AM      0.00     80.00
06:24:41 AM     21.00  31223.00
06:24:42 AM      0.00  36553.61
06:24:43 AM      0.00  40828.00
06:24:44 AM      0.00  37095.92
06:24:45 AM      0.00  42553.00

06:24:45 AM    proc/s   cswch/s
06:24:46 AM      0.00  36169.00
06:24:47 AM      0.00  36923.76
06:24:48 AM      0.00  36021.88
06:24:49 AM      0.00  45886.46
06:24:50 AM      0.00  37419.59
06:24:51 AM      0.00  47719.19
06:24:52 AM      0.00  42354.08
06:24:53 AM      0.00  38317.17
06:24:54 AM      0.00  37618.18

```

#### prepared

```
[osdba@mysql45 ~]$ pgbench -M prepared -n -r -f ./test.sql -c 16 -j 4  -T 30
transaction type: Custom query
scaling factor: 1
query mode: prepared
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 835698
latency average: 0.574 ms
tps = 27842.177752 (including connections establishing)
tps = 27889.353489 (excluding connections establishing)
statement latencies in milliseconds:
	0.572068	select 1;
[osdba@mysql45 ~]$ 


# sar -w 1 100000
06:30:18 AM      0.00     81.00
06:30:19 AM      0.00     85.86
06:30:20 AM     20.79  40064.36
06:30:21 AM      0.00 108161.62
06:30:22 AM      0.00  83697.94
06:30:23 AM      0.00  80395.96
06:30:24 AM      0.00  83589.00
06:30:25 AM      0.00  95197.98

06:30:25 AM    proc/s   cswch/s
06:30:26 AM      0.00 116096.94
06:30:27 AM      0.00 111639.80
06:30:28 AM      0.00 119719.00
06:30:29 AM      0.00 118795.96
06:30:30 AM      0.00 111051.52
06:30:31 AM      1.03 105829.90
06:30:32 AM      0.00 114250.51
06:30:33 AM      0.00 109773.74
06:30:34 AM      0.00 114905.10
06:30:35 AM      0.00 118304.95
06:30:36 AM      0.00 123064.29
06:30:37 AM      0.00 120571.00
06:30:38 AM      0.00 122222.22
06:30:39 AM      0.00 121755.00

```

​	比较三者，其中短连接的tps值为388.651729，长连接的tps值为18386.280802,prepared的tps值为27842.177752，明显短连接的tps太低，消耗性能



​	

### pgbouncer 安装

[readme](../20180810/pg_pgbouncer_install.md)



### 连接池tps

```
$ cd ~
$ vim .pgpass
*:6543:db45:ysys:ysys
```

*:任何ip

6543：端口6543

db45:pgbouncer下的数据库链接

ysys：用户

ysys:密码



#### 短链接tps测试

```
$ pgbench -M extended -n -r -f ./test.sql -h /home/ysys/pgbouncer/etc/ -p 6543 -U ysys -c 16 -j 4 -C -T 30 db45
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 120121
latency average: 3.996 ms
tps = 4003.607216 (including connections establishing)
tps = 321134.923934 (excluding connections establishing)
statement latencies in milliseconds:
	3.002545	select 1;


$ sar -w 1 10000
Linux 2.6.32-431.el6.x86_64 (gh56) 	08/17/2018 	_x86_64_	(1 CPU)

04:14:09 AM    proc/s   cswch/s
04:14:10 AM      0.00     80.81
04:14:11 AM      0.00     87.00
04:14:12 AM      0.00     79.80
04:14:13 AM      0.00     76.00
04:14:14 AM      0.00     73.74
04:14:15 AM      0.00    104.00
04:14:17 AM      4.00  55962.00
04:14:18 AM      0.00  58652.53
04:14:19 AM      0.00  58956.00
04:14:20 AM      0.00  58667.33
04:14:21 AM      0.00  58683.00
04:14:22 AM      0.00  56754.00
04:14:23 AM      0.00  58658.59
04:14:24 AM      0.00  58929.00
04:14:25 AM      0.00  56918.81
04:14:26 AM      0.00  59385.86
04:14:27 AM      0.00  58589.90
04:14:28 AM      0.00  58552.00
04:14:29 AM      1.00  58203.00
04:14:30 AM      0.00  59311.11
04:14:31 AM      1.98  58125.74
04:14:32 AM      0.00  57757.00
04:14:33 AM      0.00  54143.43
04:14:34 AM      0.00  56225.74
04:14:35 AM      0.00  55780.00
04:14:36 AM      0.00  54749.00
04:14:37 AM      0.00  54690.00
04:14:38 AM      0.00  53962.00
04:14:39 AM      0.00  46022.45
04:14:40 AM      0.00  52496.00
04:14:41 AM      0.00  51546.39
04:14:42 AM      1.01  54052.53
04:14:43 AM      0.00  53856.44
04:14:44 AM      0.00  56566.00
04:14:45 AM      0.00  57261.62
04:14:46 AM      0.00  53304.95
04:14:47 AM      0.00    203.03
04:14:48 AM      0.00     75.00

04:14:48 AM    proc/s   cswch/s
04:14:49 AM      0.00     74.00
04:14:50 AM      0.00     76.00
04:14:51 AM      0.00     77.78
04:14:52 AM      0.00    127.84
04:14:53 AM      0.00     89.00
04:14:54 AM      0.00     75.76
04:14:55 AM      0.00     78.00
04:14:56 AM      0.00     88.00
04:14:57 AM      0.00     98.99
04:14:58 AM      0.00     77.00
04:14:59 AM      0.00     77.00
04:15:00 AM      0.00     99.00
04:15:01 AM      1.01    124.24
04:15:02 AM      0.00     86.00
04:15:03 AM      0.00     80.00
04:15:04 AM      0.00     77.00
04:15:05 AM      0.00     73.74
04:15:06 AM      0.00     79.00
04:15:07 AM      0.00    100.00
04:15:08 AM      0.00     78.00
04:15:09 AM      0.00     73.74
04:15:10 AM      0.00     77.78

```



​	将discared all注销掉

```

$ pgbench -M extended -n -r -f ./test.sql -h /home/ysys/pgbouncer/etc/ -p 6543 -U ysys -c 16 -j 4 -C -T 30 db45
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 175983
latency average: 2.728 ms
tps = 5865.988937 (including connections establishing)
tps = 374830.072156 (excluding connections establishing)
statement latencies in milliseconds:
	2.050816	select 1;

$ sar -w 1 10000
Linux 2.6.32-431.el6.x86_64 (gh56) 	08/17/2018 	_x86_64_	(1 CPU)

04:49:02 AM    proc/s   cswch/s
04:49:03 AM      0.00     82.00
04:49:04 AM      0.00     78.22
04:49:05 AM      0.00     74.00
04:49:06 AM      0.00     90.91
04:49:07 AM      0.00     82.83
04:49:08 AM      0.00     83.00
04:49:09 AM      0.00     79.80
04:49:10 AM      0.00     75.00
04:49:11 AM      0.00     77.23
04:49:12 AM      0.00     80.81
04:49:13 AM      0.00     83.84
04:49:14 AM      4.00  36658.00
04:49:15 AM      0.00  60889.90
04:49:16 AM      0.00  58731.68
04:49:17 AM      0.00  55376.29
04:49:18 AM      0.00  47075.00
04:49:19 AM      0.00  64096.00
04:49:20 AM      0.00  51772.73
04:49:21 AM      0.00  52788.89
04:49:22 AM      0.00  62365.00
04:49:23 AM      0.00  43864.95
04:49:24 AM      0.00  46041.24
04:49:25 AM      0.00  56407.07
04:49:26 AM      0.00  54647.42
04:49:27 AM      0.00  57461.39
04:49:28 AM      0.00  60503.03
04:49:29 AM      1.00  61458.00
04:49:30 AM      0.00  58113.13
04:49:31 AM      0.00  62063.00
04:49:32 AM      1.00  63612.00
04:49:33 AM      0.00  63389.00
04:49:34 AM      0.00  63738.38
04:49:35 AM      0.00  63772.00
04:49:36 AM      0.00  62217.17
04:49:37 AM      0.00  62137.62
04:49:38 AM      0.00  61965.00
04:49:39 AM      0.00  63630.30
04:49:40 AM      0.00  62242.00

04:49:40 AM    proc/s   cswch/s
04:49:41 AM      0.00  62708.00
04:49:42 AM      1.01  63515.15
04:49:43 AM      0.00  61820.59
04:49:44 AM      0.00  25917.00
04:49:45 AM      0.00     88.78
04:49:46 AM      0.00     77.23
04:49:47 AM      0.00     74.00

```

将transaction变为session

```
$ pgbench -M prepared -n -r -f ./test.sql -h /home/ysys/pgbouncer/etc/ -p 6543 -U ysys -c 16 -j 4 -C  -T 30 db45
transaction type: Custom query
scaling factor: 1
query mode: prepared
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 98481
latency average: 4.874 ms
tps = 3282.465523 (including connections establishing)
tps = 7761.501444 (excluding connections establishing)
statement latencies in milliseconds:
	4.164251	select 1;

```





#### 长连接tps

```
$ pgbench -M extended -n -r -f ./test.sql -h /home/ysys/pgbouncer/etc/ -p 6543 -U ysys -c 16 -j 4  -T 30 db45
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 241173
latency average: 1.990 ms
tps = 8038.380565 (including connections establishing)
tps = 8042.285952 (excluding connections establishing)
statement latencies in milliseconds:
	1.988176	select 1;
	
	
	
$ sar -w 1 10000
Linux 2.6.32-431.el6.x86_64 (gh56) 	08/17/2018 	_x86_64_	(1 CPU)

04:59:41 AM    proc/s   cswch/s
04:59:42 AM      0.00     86.00
04:59:43 AM      0.00     79.80
04:59:44 AM      0.00     94.06
04:59:45 AM      4.04  48206.06
04:59:46 AM      0.00  73229.70
04:59:47 AM      0.00  82595.00
04:59:48 AM      0.00  81371.00
04:59:49 AM      0.00  81583.00
04:59:50 AM      0.00  82756.00
04:59:51 AM      0.00  81583.84
04:59:52 AM      0.00  79465.00
04:59:53 AM      0.00  78181.37
04:59:54 AM      0.00  82522.22
04:59:55 AM      0.00  82811.00
04:59:56 AM      0.00  81027.00
04:59:57 AM      0.00  77633.66
04:59:58 AM      0.00  74783.84
04:59:59 AM      0.00  80546.46
05:00:00 AM      0.00  81225.74
05:00:01 AM      2.02  81028.28
05:00:02 AM      0.99  78330.69
05:00:03 AM      0.00  78420.20
05:00:04 AM      0.00  81553.00
05:00:05 AM      0.00  81860.00
05:00:06 AM      0.00  82276.77
05:00:07 AM      0.00  80551.49
05:00:08 AM      0.00  80744.00
05:00:09 AM      0.00  81453.00
05:00:10 AM      0.00  80850.00
05:00:11 AM      0.00  80877.00
05:00:12 AM      0.00  81785.86
05:00:13 AM      0.00  80982.00
05:00:14 AM      0.00  82489.00
05:00:15 AM      0.00  33036.73
05:00:16 AM      0.00     80.00
05:00:17 AM      0.00     78.00
05:00:18 AM      0.00     79.00
05:00:19 AM      0.00     83.17

```



将discared all注销掉

```
$ pgbench -M extended -n -r -f ./test.sql -h /home/ysys/pgbouncer/etc/ -p 6543 -U ysys -c 16 -j 4  -T 30 db45
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 533375
latency average: 0.900 ms
tps = 17776.453980 (including connections establishing)
tps = 17784.341614 (excluding connections establishing)
statement latencies in milliseconds:
	0.898878	select 1;

$ sar -w 1 10000
Linux 2.6.32-431.el6.x86_64 (gh56) 	08/17/2018 	_x86_64_	(1 CPU)

04:50:55 AM    proc/s   cswch/s
04:50:56 AM      0.00     81.00
04:50:57 AM      1.00     91.00
04:50:58 AM      0.00     82.83
04:50:59 AM      0.00     80.61
04:51:00 AM      9.18  12008.16
04:51:01 AM      0.00 110103.00
04:51:02 AM      0.98 105980.39
04:51:03 AM      1.02 111591.84
04:51:04 AM      0.00 112428.00
04:51:05 AM      0.00 111201.00
04:51:06 AM      0.00 110348.00
04:51:07 AM      0.00 112001.00
04:51:08 AM      0.00 108139.60
04:51:09 AM      0.00 116930.30
04:51:10 AM      0.00 117369.70
04:51:11 AM      0.00 113807.92
04:51:12 AM      0.00 116273.74
04:51:13 AM      0.00 115552.00
04:51:14 AM      0.00 114886.00
04:51:15 AM      0.00 115300.00
04:51:16 AM      0.00 112712.87
04:51:17 AM      0.00 116902.04
04:51:18 AM      0.00 113026.00
04:51:19 AM      0.00 106337.00
04:51:20 AM      0.00 109114.85
04:51:21 AM      0.00 113291.00
04:51:22 AM      0.00  72005.15
04:51:23 AM      0.00 106188.89
04:51:24 AM      1.02 106426.53
04:51:25 AM      0.00 107293.00
04:51:26 AM      0.00  91272.92
04:51:27 AM      0.00  95781.63
04:51:28 AM      0.00 115796.00
04:51:29 AM      0.00  80526.04
04:51:30 AM      0.00  68419.59
04:51:31 AM      0.00     79.21
04:51:32 AM      1.02    132.65
04:51:33 AM      0.00     85.86

04:51:33 AM    proc/s   cswch/s
04:51:34 AM      0.00     75.00
04:51:35 AM      0.00     78.79
04:51:36 AM      0.00     89.11
04:51:37 AM      0.00     80.00
04:51:38 AM      0.00     80.81


```

将transaction变为session

```
 pgbench -M prepared -n -r -f ./test.sql -h /home/ysys/pgbouncer/etc/ -p 6543 -U ysys -c 16 -j 4  -T 30 db45
transaction type: Custom query
scaling factor: 1
query mode: prepared
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 797843
latency average: 0.602 ms
tps = 26594.140818 (including connections establishing)
tps = 26599.079264 (excluding connections establishing)
statement latencies in milliseconds:
	0.600630	select 1;

```





#### 小结

​	使用pgbouncer后明显在transaction或者sesession的情况下提高效率，不过相对而言，对于短链接更加具有对比性。





## 数据库高速缓存



### 本地高速缓存



#### 安装

```
# tar -zxvf pgfincore-41f2b9f.tar.gz 
# cd pgfincore-41f2b9f
# export PATH=/usr/local/pgsql/bin:$PATH
# which pg_config
# make clean
# make 
# make install
$ su - ysys
$ psql -d tutorial
tutorial=# create extension pgfincore;
CREATE EXTENSION

```

#### 测试

```
tutorial=# create table user_info(id int primary key,info text);
CREATE TABLE
tutorial=# insert into user_info select generate_series(1,5000000),md5(random()::text);
INSERT 0 5000000
tutorial=# analyze user_info;
ANALYZE
tutorial=# select pg_size_pretty(pg_total_relation_size('user_info'::regclass));
 pg_size_pretty 
----------------
 433 MB
(1 row)
```

修改postgresql.conf参数shared_buffers

```
$ vim $PGDATA/postgresql.conf
shared_buffers = 32MB
$ pg_ctl stop -m fast
$ pg_ctl start
```

清空缓存

```
# echo 3 > /proc/sys/vm/drop_caches
```

使用pgbench测试

```
$ vi test.sql
$ cat test.sql
\setrandom id 1 5000000
select * from user_info where id =:id;

$  pgbench -M prepared -n -r -f ./test.sql -c 16 -j 4 -T 10 tutorial
transaction type: Custom query
scaling factor: 1
query mode: prepared
number of clients: 16
number of threads: 4
duration: 10 s
number of transactions actually processed: 948
latency average: 172.889 ms
tps = 92.544749 (including connections establishing)
tps = 92.713889 (excluding connections establishing)
statement latencies in milliseconds:
	0.005976	\setrandom id 1 5000000
	170.067445	select * from user_info where id =:id;
```



加载本地缓存

```
tutorial=# \d user_info;
   Table "public.user_info"
 Column |  Type   | Modifiers 
--------+---------+-----------
 id     | integer | not null
 info   | text    | 
Indexes:
    "user_info_pkey" PRIMARY KEY, btree (id)
    
tutorial=#  select pgfadvise_willneed('user_info');
          pgfadvise_willneed          
--------------------------------------
 (base/16384/16385,4096,83334,131753)
(1 row)
tutorial=# select pgfadvise_willneed('user_info_pkey');
          pgfadvise_willneed          
--------------------------------------
 (base/16384/16391,4096,27424,109805)
(1 row)


$  pgbench -M prepared -n -r -f ./test.sql -c 16 -j 4 -T 10 tutorial
transaction type: Custom query
scaling factor: 1
query mode: prepared
number of clients: 16
number of threads: 4
duration: 10 s
number of transactions actually processed: 223084
latency average: 0.718 ms
tps = 22280.892011 (including connections establishing)
tps = 22346.974789 (excluding connections establishing)
statement latencies in milliseconds:
	0.001186	\setrandom id 1 5000000
	0.713107	select * from user_info where id =:id;
	
其实她还有toast表需要加载到本地缓存中


tutorial=# \x
Expanded display is on.
tutorial=# select * from pg_class where relname = 'user_info';
-[ RECORD 1 ]--+------------
relname        | user_info
relnamespace   | 2200
reltype        | 16387
reloftype      | 0
relowner       | 10
relam          | 0
relfilenode    | 16385
reltablespace  | 0
relpages       | 41667
reltuples      | 4.99998e+06
relallvisible  | 0
reltoastrelid  | 16388
relhasindex    | t
relisshared    | f
relpersistence | p
relkind        | r
relnatts       | 2
relchecks      | 0
relhasoids     | f
relhaspkey     | t
relhasrules    | f
relhastriggers | f
relhassubclass | f
relispopulated | t
relreplident   | d
relfrozenxid   | 1809
relminmxid     | 1
relacl         | 
reloptions     | 

tutorial=# select * from pg_class where oid ='16388';
-[ RECORD 1 ]--+---------------
relname        | pg_toast_16385
relnamespace   | 99
reltype        | 16389
reloftype      | 0
relowner       | 10
relam          | 0
relfilenode    | 16388
reltablespace  | 0
relpages       | 0
reltuples      | 0
relallvisible  | 0
reltoastrelid  | 0
relhasindex    | t
relisshared    | f
relpersistence | p
relkind        | t
relnatts       | 3
relchecks      | 0
relhasoids     | f
relhaspkey     | t
relhasrules    | f
relhastriggers | f
relhassubclass | f
relispopulated | t
relreplident   | n
relfrozenxid   | 1809
relminmxid     | 1
relacl         | 
reloptions     | 

tutorial=#  select pgfadvise_willneed('pg_toast.pg_toast_16385');
-[ RECORD 1 ]------+---------------------------------
pgfadvise_willneed | (base/16384/16388,4096,0,109154)

$  pgbench -M prepared -n -r -f ./test.sql -c 16 -j 4 -T 10 tutorial
transaction type: Custom query
scaling factor: 1
query mode: prepared
number of clients: 16
number of threads: 4
duration: 10 s
number of transactions actually processed: 220928
latency average: 0.725 ms
tps = 22072.369814 (including connections establishing)
tps = 22113.591198 (excluding connections establishing)
statement latencies in milliseconds:
	0.001281	\setrandom id 1 5000000
	0.720442	select * from user_info where id =:id;

发现两者没有太大区别(可能是因为测试环境较差)，不过建议将toast加载到本地缓存中
```







### 异地高速缓存



略过



 

## 链接地址



https://git.postgresql.org/gitweb/