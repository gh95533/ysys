[TOC]

# redis cluster install

**文档整理**

ysys

**日期**

2018-09-17，2018-10-01

**标签**

redis,redis cluster,single install



## 背景

​	在TJ形成一套体系后，现场这边需要部署一套redis集群(tj的redis集群是由华三部署)，从支持部CQL这边要到信息后，准备进行部署安装，顺带要了安装的链接地址(https://www.cnblogs.com/yingchen/p/6763524.html)，后期参考官网文档(https://redis.io/topics/cluster-tutorial,http://www.redis.cn/topics/cluster-tutorial.html),通过一台服务器模拟集群

## REDIS 集群部署

### 环境

| 环境     | 版本                |
| -------- | ------------------- |
| 操作系统 | centos6.5_x64       |
| 软件     | redis-3.2.12.tar.gz |
| 软件     | redis-3.2.2.gem     |





### 操作步骤

#### 上传软件包并解压文件

```
# cd /software
# tar xzvf redis-3.2.12.tar.gz 
```

#### 编译软件

软件编译过程中可能会报错，遇到报错，可以查看报错信息

```
# cd /software/redis-3.2.12/
# make
# make install PREFIX=/usr/local/redis-cluster
```



#### 拷贝文件夹并修改文件

```
# cd /usr/local/redis-cluster
# mv bin redis01
# cd redis01
# vim redis.conf

daemonize yes
port 7001
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
appendonly yes


```

```
# cd /usr/local/redis-cluster
# cp redis01 redis02
# cp redis01 redis03
# cp redis01 redis04
# cp redis01 redis05
# cp redis01 redis06

--修改redis02 redis03 redis04 redis05 redis06的redis.conf
# cd /usr/local/redis-cluster/redis02
# vim redis.conf

port 7002

#  vim  /usr/local/redis-cluster/redis03/redis.conf

port 7003

#  vim  /usr/local/redis-cluster/redis04/redis.conf

port 7004

#  vim  /usr/local/redis-cluster/redis05/redis.conf

port 7005

#  vim  /usr/local/redis-cluster/redis06/redis.conf

port 7006
```

#### 复制解压文件src的redis-trib.rb文件到redis-cluster下

```
# cp /software/redis-3.2.12/src/redis-trib.rb ./
```

#### 安装ruby环境

```
# yum install ruby
...
Installing:
 ruby                    x86_64        1.8.7.374-5.el6        base        538 k
Installing for dependencies:
 compat-readline5        x86_64        5.2-17.1.el6           base        130 k
 ruby-libs               x86_64        1.8.7.374-5.el6        base        1.7 M

Transaction Summary
...
Running Transaction
  Installing : compat-readline5-5.2-17.1.el6.x86_64                         1/3 
  Installing : ruby-libs-1.8.7.374-5.el6.x86_64                             2/3 
  Installing : ruby-1.8.7.374-5.el6.x86_64                                  3/3 
  Verifying  : compat-readline5-5.2-17.1.el6.x86_64                         1/3 
  Verifying  : ruby-1.8.7.374-5.el6.x86_64                                  2/3 
  Verifying  : ruby-libs-1.8.7.374-5.el6.x86_64                             3/3 
```

```
# yum install rubygems

...

Installing:
 rubygems          noarch         1.3.7-5.el6                base         207 k
Installing for dependencies:
 ruby-irb          x86_64         1.8.7.374-5.el6            base         318 k
 ruby-rdoc         x86_64         1.8.7.374-5.el6            base         381 k
 
...

Running Transaction
  Installing : ruby-irb-1.8.7.374-5.el6.x86_64                              1/3 
  Installing : ruby-rdoc-1.8.7.374-5.el6.x86_64                             2/3 
  Installing : rubygems-1.3.7-5.el6.noarch                                  3/3 
  Verifying  : ruby-irb-1.8.7.374-5.el6.x86_64                              1/3 
  Verifying  : rubygems-1.3.7-5.el6.noarch                                  2/3 
  Verifying  : ruby-rdoc-1.8.7.374-5.el6.x86_64                             3/3 

```

```
# cd /software
# gem install redis-3.2.2.gem  
```



#### 创建脚本启动redis节点

```
# cd /usr/local/redis-cluster
# vim start-all.sh

cd redis01
./redis-server redis.conf
cd ..
cd redis02
./redis-server redis.conf
cd ..
cd redis03
./redis-server redis.conf
cd ..
cd redis04
./redis-server redis.conf
cd ..
cd redis05
./redis-server redis.conf
cd ..
cd redis06
./redis-server redis.conf
cd ..

# cd /usr/local/redis-cluster
# chmod 777 start-all.sh
# ./start-all.sh
# ps -ef|grep redis
```



#### 使用redis-trib.rb创建集群



```
# cd /usr/local/redis-cluster
# ./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7001
127.0.0.1:7002
127.0.0.1:7003
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
Adding replica 127.0.0.1:7006 to 127.0.0.1:7003
M: cb341bd6d3adcbf919e306ebd39d5780649cd5ec 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
M: abe95347f19f882acf67a728ddf3ee5793673f43 127.0.0.1:7002
   slots:5461-10922 (5462 slots) master
M: 14c49ea4fcdece34fd501007242c87aae33c9dda 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
S: 7d1863deac04f225c794855238e67ea91fd253fe 127.0.0.1:7004
   replicates cb341bd6d3adcbf919e306ebd39d5780649cd5ec
S: 8b568abb7ccf27020b2563d45398ebb6f50f1e33 127.0.0.1:7005
   replicates abe95347f19f882acf67a728ddf3ee5793673f43
S: 75615852222132e923ade94b4b1ba6d78e639758 127.0.0.1:7006
   replicates 14c49ea4fcdece34fd501007242c87aae33c9dda
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join......
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: cb341bd6d3adcbf919e306ebd39d5780649cd5ec 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 7d1863deac04f225c794855238e67ea91fd253fe 127.0.0.1:7004
   slots: (0 slots) slave
   replicates cb341bd6d3adcbf919e306ebd39d5780649cd5ec
M: 14c49ea4fcdece34fd501007242c87aae33c9dda 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: abe95347f19f882acf67a728ddf3ee5793673f43 127.0.0.1:7002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 8b568abb7ccf27020b2563d45398ebb6f50f1e33 127.0.0.1:7005
   slots: (0 slots) slave
   replicates abe95347f19f882acf67a728ddf3ee5793673f43
S: 75615852222132e923ade94b4b1ba6d78e639758 127.0.0.1:7006
   slots: (0 slots) slave
   replicates 14c49ea4fcdece34fd501007242c87aae33c9dda
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

上面显示创建成功，有3个主节点，3个从节点，每个节点都是成功连接状态。

 

三个主节点[M]以及分配的哈希卡槽如下：

```
M: cb341bd6d3adcbf919e306ebd39d5780649cd5ec 127.0.0.1:7001
   slots:0-5460 (5461 slots) master
M: abe95347f19f882acf67a728ddf3ee5793673f43 127.0.0.1:7002
   slots:5461-10922 (5462 slots) master
M: 14c49ea4fcdece34fd501007242c87aae33c9dda 127.0.0.1:7003
   slots:10923-16383 (5461 slots) master
```

三个从节点[S]以及附属的主节点如下:

```
S: 7d1863deac04f225c794855238e67ea91fd253fe 127.0.0.1:7004
   replicates cb341bd6d3adcbf919e306ebd39d5780649cd5ec
S: 8b568abb7ccf27020b2563d45398ebb6f50f1e33 127.0.0.1:7005
   replicates abe95347f19f882acf67a728ddf3ee5793673f43
S: 75615852222132e923ade94b4b1ba6d78e639758 127.0.0.1:7006
```



### 集群测试

```
# cd /usr/local/redis-cluster/redis01
# ./redis-cli -c -p 7001  
127.0.0.1:7001> set name ysys
-> Redirected to slot [5798] located at 127.0.0.1:7002
OK
127.0.0.1:7002> get name
"ysys"
127.0.0.1:7002> exit
# cd /usr/local/redis-cluster/redis06
# ./redis-cli -c -p 7006
127.0.0.1:7006> get name
-> Redirected to slot [5798] located at 127.0.0.1:7002
"ysys"
127.0.0.1:7002> exit

# ps -ef|grep redis | grep cluster
root      2771     1  0 22:17 ?        00:00:09 ./redis-server *:7001 [cluster]
root      2773     1  0 22:17 ?        00:00:09 ./redis-server *:7002 [cluster]
root      2775     1  0 22:17 ?        00:00:09 ./redis-server *:7003 [cluster]
root      2783     1  0 22:17 ?        00:00:08 ./redis-server *:7004 [cluster]
root      2785     1  0 22:17 ?        00:00:08 ./redis-server *:7005 [cluster]
root      2789     1  0 22:17 ?        00:00:09 ./redis-server *:7006 [cluster]
```



 	当前通过一台服务器来测试了redis集群。



## 错误

1、	make 编译报错

```
# make
...
cd hiredis && make static
make[3]: Entering directory `/software/redis-3.2.12/deps/hiredis'
gcc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  net.c
make[3]: gcc: Command not found
make[3]: *** [net.o] Error 127
make[3]: Leaving directory `/software/redis-3.2.12/deps/hiredis'
make[2]: *** [hiredis] Error 2
make[2]: Leaving directory `/software/redis-3.2.12/deps'
make[1]: [persist-settings] Error 2 (ignored)
    CC adlist.o
/bin/sh: cc: command not found
make[1]: *** [adlist.o] Error 127
make[1]: Leaving directory `/software/redis-3.2.12/src'
make: *** [all] Error 2
```

```
# yum -y install gcc*
```

```
# make
...
cd src && make all
make[1]: Entering directory `/software/redis-3.2.12/src'
    CC adlist.o
In file included from adlist.c:34:
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/software/redis-3.2.12/src'
make: *** [all] Error 2
```

​	执行命令

```
# make MALLOC=libc
```



2、redis-trib.rb

```
# ./redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006
>>> Creating cluster
[ERR] Sorry, can't connect to node 127.0.0.1:7001
```

 	关掉了进程 redis进程导致的

```
# ./start-all.sh 
```





## 链接地址

https://www.cnblogs.com/yingchen/p/6763524.html

https://blog.csdn.net/fengshizty/article/details/51368004

https://blog.csdn.net/yejingtao703/article/details/78484151

https://redis.io/topics/cluster-tutorial

https://blog.csdn.net/wzygis/article/details/51705559

https://blog.csdn.net/dream_an/article/details/77199462

http://www.redis.cn/topics/cluster-tutorial.html

http://mdba.cn/2015/08/19/redis-cluster%E5%88%9B%E5%BB%BA/

https://blog.csdn.net/babylovewei/article/details/82851451

http://www.redis.cn/topics/cluster-spec.html

https://www.cnblogs.com/kangoroo/p/7657616.html

https://www.cnblogs.com/yuanermen/p/5717885.html



