[TOC]

# redis cluster install multiple not internet

**文档整理**

ysys

**日期**

2018-10-01

**标签**

redis cluster install,multiple install, not internet



## 背景

​	安装redis cluster时是通过互联网上的yum来进行安装，而现实情况是所处的网络不是互联网，一些安装的脚本需要调整，此文档就是因此需要整理的



## 环境

| software  | 版本                |
| --------- | ------------------- |
| operation | centos6.5_64        |
| redis     | redis-3.2.12.tar.gz |
| gem       | redis-3.2.2.gem     |

| hostname | 7000 | 7001 | 7002 | 7003 | 7004 | 7005 |
| -------- | ---- | ---- | ---- | ---- | ---- | ---- |
| gh18     | √    | √    |      |      |      |      |
| gh19     |      |      | √    | √    |      |      |
| gh20     |      |      |      |      | √    | √    |



## 安装



### 每个服务器都要执行一遍



#### 创建文件夹并上传文件



ruby_rpm

链接：https://pan.baidu.com/s/1eY8SBKl-Vmeu1XNQPH1qbg 
提取码：br0n

createrepo_rpm

链接：https://pan.baidu.com/s/1ItAFu_t8Qjv9qTPf7MovZA 
提取码：afur



```
# mkdir /software
# cd /softwate

--上传redis-3.2.12.tar.gz
--上传redis-3.2.2.gem
--上传createrepo-rpm
--上传ruby-rpm
```

#### 解压文件并安装redis-3.2.12

```
# cd /software/
# tar xvzf redis-3.2.12.tar.gz 
# cd /software/redis-3.2.12
# make
# make install=/usr/local/redis-cluster
```

#### 在每台服务器上文件夹下创建文件并修改redis.conf

​	每个服务器上创建两个文件夹，依次创建不同的文件夹，分别为7000,7001;7002,7003;7004,7005

```
# cd /usr/local/redis-cluster
# mkdir 7000
# mkdir 7001
```

```
# cd /usr/local/redis-cluster/7000
# vim redis.conf

bind 192.168.1.18
daemonize    yes                          
pidfile  /var/run/redis_7000.pid          
port  7000                               
cluster-enabled  yes                      
cluster-config-file  nodes_7000.conf      
cluster-node-timeout  5000                
appendonly  yes

# cd /usr/local/redis-cluster/7001
# vim redis.conf

bind 192.168.1.18
daemonize    yes                          
pidfile  /var/run/redis_7001.pid          
port  7001                               
cluster-enabled  yes                      
cluster-config-file  nodes_7001.conf      
cluster-node-timeout  5000                
appendonly  yes
```

​	服务器一创建7000,7001两个文件夹，服务器二创建7002,7003文件夹，服务器三创建7004,7005文件夹

#### 三台服务器启动redis服务

```
[gh18]
# cd /usr/local/redis-cluster/bin
# ./redis-server ../7000/redis.conf
# ./redis-server ../7001/redis.conf

[gh19]
# cd /usr/local/redis-cluster/bin
# ./redis-server ../7002/redis.conf
# ./redis-server ../7003/redis.conf

[gh20]
# cd /usr/local/redis-cluster/bin
# ./redis-server ../7004/redis.conf
# ./redis-server ../7005/redis.conf
```



#### 检查节点redis服务启动

```
# ps -ef|grep redis
```



#### 安装ruby等依赖包

​	因为是内网，没有办法直接连接互联网；而且本次的依赖包需要更新之前的依赖包就想到了使用createrepo的方式自制yum形式

##### createrepo 

​	createrepo是yum创建的基础，它的依赖包较少，总共3个rpm包，已经上传到了/software/linux_yum_6.5/上

```
# cd /software/linux_yum_6.5/
# rpm -ivh python-deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm  createrepo-0.9.9-18.el6.noarch.rpm deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm 
```

##### ruby依赖包

​	ruby依赖包已经上传到/software/redis-rpm-3.2.22-cluster/

```
# cd /software/redis-rpm-3.2.22-cluster/
# createrepo -v .
# ls -ls repodata*
```

##### 修改yum源

```
# vim /etc/yum.repos.d/{你的yum}.repo

[ysys]
name=ysys install
baseurl=file:///software/redis-rpm-3.2.22-cluster
enabled=1
gpgcheck=0
```

##### 安装ruby依赖包

```
# yum -y install  ruby ruby-devel rubygems rpm-build 
```

#### 安装 gem

```
# cd /software/
# gem install redis-3.2.2.gem
```



### 集群安装

只要在一台服务器上创建就可以了

```
# cd /usr/local/redis-cluster/bin
# cp /software/redis-3.2.12/src/redis-trib.rb ./
# ./redis-trib.rb 
# ./redis-trib.rb create --replicas 1 192.168.1.18:7000 192.168.1.18:7001 192.168.1.19:7002 192.168.1.19:7003 192.168.1.20:7004 192.168.1.20:7005


>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.1.19:7002
192.168.1.18:7000
192.168.1.20:7004
Adding replica 192.168.1.18:7001 to 192.168.1.19:7002
Adding replica 192.168.1.19:7003 to 192.168.1.18:7000
Adding replica 192.168.1.20:7005 to 192.168.1.20:7004
M: 7fec74ccd4ac03ffb2a6ddb789bc71e2cf88e1f4 192.168.1.18:7000
   slots:5461-10922 (5462 slots) master
S: 2fec7ba73c69f09acc67e3bf20c980ab319f08ef 192.168.1.18:7001
   replicates c0213d85374423a2e624a8506d75f128c503264a
M: c0213d85374423a2e624a8506d75f128c503264a 192.168.1.19:7002
   slots:0-5460 (5461 slots) master
S: b3d3187fb89cd7d27def83ddf80558aa257d3daf 192.168.1.19:7003
   replicates 7fec74ccd4ac03ffb2a6ddb789bc71e2cf88e1f4
M: f1acbf85af5908899896542ddca3c1ec95173e94 192.168.1.20:7004
   slots:10923-16383 (5461 slots) master
S: 6627d0d722c2f651dfacabce30b9a55e759290c2 192.168.1.20:7005
   replicates f1acbf85af5908899896542ddca3c1ec95173e94
Can I set the above configuration? (type 'yes' to accept): *yes*
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
>>> Performing Cluster Check (using node 192.168.1.18:7000)
M: 7fec74ccd4ac03ffb2a6ddb789bc71e2cf88e1f4 192.168.1.18:7000
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 2fec7ba73c69f09acc67e3bf20c980ab319f08ef 192.168.1.18:7001
   slots: (0 slots) slave
   replicates c0213d85374423a2e624a8506d75f128c503264a
M: f1acbf85af5908899896542ddca3c1ec95173e94 192.168.1.20:7004
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: c0213d85374423a2e624a8506d75f128c503264a 192.168.1.19:7002
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 6627d0d722c2f651dfacabce30b9a55e759290c2 192.168.1.20:7005
   slots: (0 slots) slave
   replicates f1acbf85af5908899896542ddca3c1ec95173e94
S: b3d3187fb89cd7d27def83ddf80558aa257d3daf 192.168.1.19:7003
   slots: (0 slots) slave
   replicates 7fec74ccd4ac03ffb2a6ddb789bc71e2cf88e1f4
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



