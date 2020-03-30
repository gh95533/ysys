[TOC]

# GZ mongo cluster install

## 背景

​	由于GZ申请的服务器较少，之前MONGO申请了五台服务器，而现在MONGO只有三台，这样就导致之前的文档要重新书写(其实就是自己对于mongo分片副本不是很了解)

## 规划

![_](../img_src/000/2018-09-13_143925.png)

​	其中MMS不需要关注(MMS是什么啊，不知道)

​	后期正常安装后，发现有问题，每个shard{X}节点都需要仲裁节点，安装手册请参考[步骤](../20180919/gz_mongo_cluster_install_er.md)

​	当前的部署安装手册有问题，问题主要是因为Arbiter

​	Arbiter，仲裁者，是复制集中的一个MongbDB实例，他并不保存数据。仲裁节点使用最小的资源并且要求硬件设备，不能将Arbiter部署在同一个数据集节点中，可以部署在其他应用服务器或者监视服务器中，也可部署在单独的虚拟中。为了确保复制集中有奇数的投票成员(包括primary)，需要添加仲裁节点作为投票，否则primary不能运行时不会自动切换primary。

​	



## 安装步骤



### 每台服务器

​	在每台服务器上都要执行下面操作，要注意提醒哪个节点执行哪个节点不执行

#### 解压文件并放到规定的目录

```
# mkdir /software
# cd /software
--通过软件将mongodb-linux-x86_64-3.4.6.tgz上传到/software
# tar -xzvf mongodb-linux-x86_64-3.6.3.tgz  -C /usr/local/
# cd /usr/local
# mv mongodb-linux-x86_64-3.6.3 mongodb
```

#### 创建目录及用户，授权，修改变量

```
# mkdir /data
# groupadd ysys
# useradd -g ysys ysys
# passwd ysys
--密码确认
# chown -R ysys:ysys /usr/local/
# chown -R ysys:ysys /data
```

```
# vim /etc/profile
--在export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL 上面添加下面两行
export MONGODB_HOME=/usr/local/mongodb
export PATH=$MONGODB_HOME/bin:$PATH

# source /etc/profile
```

```
# su - ysys
$ mongod -v 
--查看mongo版本是否存在
```

在每台服务器上执行下面语句

```
mkdir -p /usr/local/mongodb/conf
mkdir -p /data/mongos/log
mkdir -p /data/config/data
mkdir -p /data/config/log
mkdir -p /data/shard1/data
mkdir -p /data/shard1/log
mkdir -p /data/shard2/data
mkdir -p /data/shard2/log
mkdir -p /data/shard3/data
mkdir -p /data/shard3/log
```

```
# chown -R ysys:ysys /usr/local/
# chown -R ysys:ysys /data
```

#### 创建config

```
# su - ysys
$ cd /usr/local/mongodb/conf
$ vi config.conf
```

```
## content
systemLog:
  destination: file
  logAppend: true
  path: /data/config/log/config.log
 
# Where and how to store data.
storage:
  dbPath: /data/config/data
  journal:
    enabled: true
# how the process runs
processManagement:
  fork: true
  pidFilePath: /data/config/log/configsrv.pid
 
# network interfaces
net:
  port: 21000
  bindIp: 192.168.1.31
 
#operationProfiling:
replication:
    replSetName: config        

sharding:
    clusterRole: configsvr
```

​	注意修改每台的ip地址 服务器为192.168.1.31的bindIp: 192.168.1.31，服务器为192.168.1.32的bindIp: 192.168.1.32，服务器为192.168.1.33的bindIp: 192.168.1.33;上述配置完成后，才能执行启动下面的服务

**每台服务器都要执行**

```
$ numactl --interleave=all mongod --config /usr/local/mongodb/conf/config.conf
```

**只要登陆到其中一台就可以了，以ip:192.168.1.31为例**

```
$ mongo 192.168.1.31:21000
```

```
> config = { _id : "config",members : [{_id : 0, host : "192.168.1.31:21000" },{_id : 1, host : "192.168.1.32:21000" }, {_id : 2, host : "192.168.1.33:21000" }]}

> rs.initiate(config)

> rs.status()
```



### 节点一节点二 创建 shard1

**在本次安装中以192.168.1.31为节点一，192.168.1.32为节点二，192.168.1.33为节点三**

```
# su - ysys
$ cd /usr/local/mongodb/conf/
$ vim shard1.conf
```

```
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/shard1/log/shard1.log
 
# Where and how to store data.
storage:
  dbPath: /data/shard1/data
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 20

# how the process runs
processManagement:
  fork: true 
  pidFilePath: /data/shard1/log/shard1.pid
 
# network interfaces
net:
  port: 27001
  bindIp: 192.168.1.31

#operationProfiling:
replication:
    replSetName: shard1
sharding:
    clusterRole: shardsvr
```

​	节点二为192.168.1.32，需要将bindIp:192.168.1.31改成自己的ip，bindIp:192.168.1.32，只有在节点二也创建完成 ` vim shard1.conf `,才能启动下面的服务，服务必须节点一节点二启动完成

**启动两台服务器的shard1 server **

```
$ numactl --interleave=all mongod  --config  /usr/local/mongodb/conf/shard1.conf
```

**登陆任意一台服务器，初始化副本集 **

```
$ mongo 192.168.1.31:27001
> use admin
```

```
> config = { _id : "shard1", members : [ {_id : 0, host : "192.168.1.31:27001" }, {_id : 1, host : "192.168.1.32:27001" }  ]}

> rs.initiate(config)
```



### 节点二 节点三 创建 shard2

**在本次安装中以192.168.1.31为节点一，192.168.1.32为节点二，192.168.1.33为节点三**

```
# su - ysys
$ cd /usr/local/mongodb/conf/
$ vim shard2.conf
```

```
systemLog:
  destination: file
  logAppend: true
  path: /data/shard2/log/shard2.log
 
# Where and how to store data.
storage:
  dbPath: /data/shard2/data
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 20
 
# how the process runs
processManagement:
  fork: true 
  pidFilePath: /data/shard2/log/shard2.pid
 
# network interfaces
net:
  port: 27002
  bindIp: 192.168.1.32

 
#operationProfiling:
replication:
    replSetName: shard2
sharding:
    clusterRole: shardsvr
```

节点三为192.168.1.33，需要将bindIp:192.168.1.32改成自己的ip，bindIp:192.168.1.33，只有在节点三也创建完成 ` vim shard1.conf `,才能启动下面的服务，服务必须节点二节点三启动完成

**启动两台服务器的shard2 server **

```
$ numactl --interleave=all mongod  --config  /usr/local/mongodb/conf/shard2.conf
```

**登陆任意一台服务器，初始化副本集 **

```
$ mongo 192.168.1.32:27002
> use admin
```

```
> config = {_id : "shard2", members : [{_id : 0, host : "192.168.1.32:27002" },{_id : 1, host : "192.168.1.33:27002" } ]}


> rs.initiate(config)
```



### 节点三 节点一 创建 shard3

**在本次安装中以192.168.1.31为节点一，192.168.1.32为节点二，192.168.1.33为节点三**

```
# su - ysys
$ cd /usr/local/mongodb/conf/
$ vim shard3.conf
```

```

#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /data/shard3/log/shard3.log
 
# Where and how to store data.
storage:
  dbPath: /data/shard3/data
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 20
# how the process runs
processManagement:
  fork: true 
  pidFilePath: /data/shard3/log/shard3.pid
 
# network interfaces
net:
  port: 27003
  bindIp: 192.168.1.31

 
#operationProfiling:
replication:
    replSetName: shard3
sharding:
    clusterRole: shardsvr
```

节点三为192.168.1.33，需要将bindIp:192.168.1.31改成自己的ip，bindIp:192.168.1.33，只有在节点三也创建完成 ` vim shard1.conf `,才能启动下面的服务，服务必须节点一节点三启动完成

**启动两台服务器的shard3 server **

```
$ numactl --interleave=all mongod  --config  /usr/local/mongodb/conf/shard3.conf
```

**登陆任意一台服务器，初始化副本集 **

```
$ mongo 192.168.1.31:27003
> use admin
```

```
> config = { _id : "shard3",members : [ {_id : 0, host : "192.168.1.31:27003" },{_id : 1, host : "192.168.1.33:27003" }] }

> rs.initiate(config)
```





### 创建mongos

**在本次安装中以192.168.1.31为节点一，192.168.1.32为节点二，192.168.1.33为节点三**

​	本次需要的是从节点一(192.168.1.31)，节点二(192.168.1.32)开启mongos

#### 节点一 配置mongos.conf

```
# su - ysys
$ cd /usr/local/mongodb/conf/
$ vim mongos.conf
```

```
systemLog:
  destination: file
  logAppend: true
  path: /data/mongos/log/mongos.log
processManagement:
  fork: true
#  pidFilePath: /usr/local/mongodb/mongos.pid
 
# network interfaces
net:
  port: 20000
  bindIp: 192.168.1.31
#监听的配置服务器,只能有1个或者3个 configs为配置服务器的副本集名字
sharding:
   configDB: config/192.168.1.31:21000,192.168.1.32:21000,192.168.1.33:21000
```



#### 节点二配置mongos.conf

```
# su - ysys
$ cd /usr/local/mongodb/conf/
$ vim mongos.conf
```

```
systemLog:
  destination: file
  logAppend: true
  path: /data/mongos/log/mongos.log
processManagement:
  fork: true
#  pidFilePath: /usr/local/mongodb/mongos.pid
 
# network interfaces
net:
  port: 20000
  bindIp: 192.168.1.32
#监听的配置服务器,只能有1个或者3个 configs为配置服务器的副本集名字
sharding:
   configDB: config/192.168.1.31:21000,192.168.1.32:21000,192.168.1.33:21000
```



#### 两台服务器启动服务

```
$ mongos  --config  /usr/local/mongodb/conf/mongos.conf
```



### 启用分片

​	目前搭建了mongodb配置服务器、路由服务器，各个分片服务器，不过应用程序连接到mongos路由服务器并不能使用分片机制，还需要在程序里设置分片配置，让分片生效。 

​	在这里安装服务mongos的是节点一和节点二

​	登陆任意一台mongos 

```
$ mongo 192.168.1.31:20000
> use admin
> sh.addShard("shard1/192.168.1.31:27001,192.168.1.32:27001")
> sh.addShard("shard2/192.168.1.32:27002,192.168.1.33:27002")
> sh.addShard("shard3/192.168.1.33:27003,192.168.1.31:27003")
> sh.status()
```



##  启动步骤

​	启动步骤请注意安装的顺序就是启动的过程

### 每一台服务器启动

```
# su - ysys
$ numactl --interleave=all mongod --config /usr/local/mongodb/conf/config.conf
```

### 节点一节点二 启动 shard1

```
$ numactl --interleave=all mongod  --config  /usr/local/mongodb/conf/shard1.conf
```

### 节点二节点三 启动 shard2

```
$ numactl --interleave=all mongod  --config  /usr/local/mongodb/conf/shard2.conf
```

### 节点三节点一 启动 shard3

```
$ numactl --interleave=all mongod  --config  /usr/local/mongodb/conf/shard3.conf
```

### 节点一节点二启动 mongos

```
$ mongos  --config  /usr/local/mongodb/conf/mongos.conf
```



### 关闭步骤

​	从mongos>shard3>shard2>shard1>config 删除进程



## 链接地址

https://www.cnblogs.com/ityouknow/p/7566682.html





