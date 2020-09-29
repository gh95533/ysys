[TOC]

# pg pgbench



## 安装

$ make 

$ make install

## 帮助

```
[osdba@mysql45 ~]$ pgbench --help
pgbench is a benchmarking tool for PostgreSQL.

Usage:
  pgbench [OPTION]... [DBNAME]

Initialization options:
  -i, --initialize         invokes initialization mode
  -F, --fillfactor=NUM     set fill factor
  -n, --no-vacuum          do not run VACUUM after initialization
  -q, --quiet              quiet logging (one message each 5 seconds)
  -s, --scale=NUM          scaling factor
  --foreign-keys           create foreign key constraints between tables
  --index-tablespace=TABLESPACE
                           create indexes in the specified tablespace
  --tablespace=TABLESPACE  create tables in the specified tablespace
  --unlogged-tables        create tables as unlogged tables

Benchmarking options:
  -c, --client=NUM         number of concurrent database clients (default: 1)
  -C, --connect            establish new connection for each transaction
  -D, --define=VARNAME=VALUE
                           define variable for use by custom script
  -f, --file=FILENAME      read transaction script from FILENAME
  -j, --jobs=NUM           number of threads (default: 1)
  -l, --log                write transaction times to log file
  -M, --protocol=simple|extended|prepared
                           protocol for submitting queries (default: simple)
  -n, --no-vacuum          do not run VACUUM before tests
  -N, --skip-some-updates  skip updates of pgbench_tellers and pgbench_branches
  -P, --progress=NUM       show thread progress report every NUM seconds
  -r, --report-latencies   report average latency per command
  -R, --rate=NUM           target rate in transactions per second
  -s, --scale=NUM          report this scale factor in output
  -S, --select-only        perform SELECT-only transactions
  -t, --transactions=NUM   number of transactions each client runs (default: 10)
  -T, --time=NUM           duration of benchmark test in seconds
  -v, --vacuum-all         vacuum all four standard tables before tests
  --aggregate-interval=NUM aggregate data over NUM seconds
  --sampling-rate=NUM      fraction of transactions to log (e.g. 0.01 for 1%)

Common options:
  -d, --debug              print debugging output
  -h, --host=HOSTNAME      database server host or socket directory
  -p, --port=PORT          database server port number
  -U, --username=USERNAME  connect as specified database user
  -V, --version            output version information, then exit
  -?, --help               show this help, then exit

Report bugs to <pgsql-bugs@postgresql.org>.

```



```
pgbench -M prepared -n -r -f ./test.sql -c 16 -j 4 -C -T 30
```

-M:{simple|extended|prepared} 给服务器提交查询的协议

-n:在测试之前不运行vacuum

-r:报告每条命令的平均延迟 

-f FILENAME 从文件FILENAME读取事务脚本 

-c:数据库客户端并发数（默认：1） 

-j:线程数 

-C:为每个事务建立新的连接 

-T:benchmark测试时间 

```
transaction type: Custom query
scaling factor: 1
query mode: extended
number of clients: 16
number of threads: 4
duration: 30 s
number of transactions actually processed: 15660
latency average: 30.651 ms
tps = 521.868802 (including connections establishing)
tps = 190594.420914 (excluding connections establishing)
statement latencies in milliseconds:
	22.989001	select 1;

```







## 链接地址

http://www.postgres.cn/docs/9.4/pgbench.html

https://www.postgresql.org/docs/9.4/static/pgbench.html

https://www.cnblogs.com/songyuejie/p/5007605.html