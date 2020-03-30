[TOC]

# mongodb cluster install for linux



## 环境准备

操作系统：centos6.5

服务器：五台

软件：mongodb-linux-x86_64-3.4.6.tgz

服务器规划

| 服务器1        | 服务器2        | 服务器3        | 服务器4        | 服务器5        |
| -------------- | -------------- | -------------- | -------------- | -------------- |
| mongos server  | mongos server  | config server  | config server  | config server  |
| shared1 server | shared2 server | shared3 server | shared4 server | shared5 server |
| shared5 server | shared1 server | shared2 server | shared3 server | shared4 server |
| shared4 server | shared5 server | shared1 server | shared2 server | shared3 server |



## 使用虚拟机操作

​	使用一台服务器将部分可以直接拷贝的文件先安装

​	上传文件解压文件

```
# mkdir /software --文件上传到这个路径下
# cd /software
# tar -xzvf mongodb-linux-x86_64-3.4.6.tgz -C /usr/local/
# cd /usr/local/
# mv mongodb-linux-x86_64-3.4.6 mongodb
```

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
mkdir -p /data/shard4/data
mkdir -p /data/shard4/log
mkdir -p /data/shard5/data
mkdir -p /data/shard5/log
```

```
# cd ~
# vim .bahs_profile
export MONGODB_HOME=/usr/local/mongodb
export PATH=$MONGODB_HOME/bin:$PATH
# source .bash_profile
```

```
# groupadd ysys
# useradd -g ysys ysys
# chown -R ysys:ysys /usr/local
# chown -R ysys:ysys /data
```



校验mongo环境是否配置正常

```
# mongod -v
...
2018-09-11T07:23:53.759+0800 I CONTROL  [initandlisten] db version v3.4.6
...
```



关机将修改其他四台的ip和主机名

```
mongo 192.168.1.33:27000
```



```
config = {_id : "config", members : [ {_id : 0, host : "192.168.1.33:21000" }, {_id : 1, host : "192.168.1.34:21000" },{_id : 2, host : "192.168.1.35:21000" } ]}
```

```
rs.initiate(config)
```

```
rs.status();
```

```
> config = {_id : "config", members : [ {_id : 0, host : "192.168.1.33:21000" }, {_id : 1, host : "192.168.1.34:21000" },{_id : 2, host : "192.168.1.35:21000" } ]}
{
	"_id" : "config",
	"members" : [
		{
			"_id" : 0,
			"host" : "192.168.1.33:21000"
		},
		{
			"_id" : 1,
			"host" : "192.168.1.34:21000"
		},
		{
			"_id" : 2,
			"host" : "192.168.1.35:21000"
		}
	]
}
> rs.initiate(config)
{
	"ok" : 0,
	"errmsg" : "Attempting to initiate a replica set with name config, but command line reports configs; rejecting",
	"code" : 93,
	"codeName" : "InvalidReplicaSetConfig"
}
> rs.status();
{
	"info" : "run rs.initiate(...) if not yet done for the set",
	"ok" : 0,
	"errmsg" : "no replset config has been received",
	"code" : 94,
	"codeName" : "NotYetInitialized"
}
> 
```



```
mongo 192.168.1.31:27001
```

```
use admin
```

```
config = {_id : "shard1",members : [{_id : 0, host : "192.168.1.31:27001" },{_id : 1, host : "192.168.1.32:27001" },{_id : 2, host : "192.168.1.33:27001" }] }
```











三台部署

```
config = { _id : "config",members : [{_id : 0, host : "192.168.1.31:21000" },{_id : 1, host : "192.168.1.32:21000" }, {_id : 2, host : "192.168.1.33:21000" }]}
```





```
> config = { _id : "config",members : [{_id : 0, host : "192.168.1.31:21000" },{_id : 1, host : "192.168.1.32:21000" }, {_id : 2, host : "192.168.1.33:21000" }]}
{
	"_id" : "config",
	"members" : [
		{
			"_id" : 0,
			"host" : "192.168.1.31:21000"
		},
		{
			"_id" : 1,
			"host" : "192.168.1.32:21000"
		},
		{
			"_id" : 2,
			"host" : "192.168.1.33:21000"
		}
	]
}
> rs.initiate(config)
{
	"ok" : 1,
	"operationTime" : Timestamp(1536787012, 1),
	"$gleStats" : {
		"lastOpTime" : Timestamp(1536787012, 1),
		"electionId" : ObjectId("000000000000000000000000")
	},
	"$clusterTime" : {
		"clusterTime" : Timestamp(1536787012, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```



```
config:SECONDARY> rs.status();
{
	"set" : "config",
	"date" : ISODate("2018-09-12T21:17:08.137Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"configsvr" : true,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1536787025, 3),
			"t" : NumberLong(1)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1536787025, 3),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1536787025, 3),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1536787025, 3),
			"t" : NumberLong(1)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.1.31:21000",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 134,
			"optime" : {
				"ts" : Timestamp(1536787025, 3),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-09-12T21:17:05Z"),
			"infoMessage" : "could not find member to sync from",
			"electionTime" : Timestamp(1536787023, 1),
			"electionDate" : ISODate("2018-09-12T21:17:03Z"),
			"configVersion" : 1,
			"self" : true
		},
		{
			"_id" : 1,
			"name" : "192.168.1.32:21000",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 15,
			"optime" : {
				"ts" : Timestamp(1536787025, 3),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1536787025, 3),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-09-12T21:17:05Z"),
			"optimeDurableDate" : ISODate("2018-09-12T21:17:05Z"),
			"lastHeartbeat" : ISODate("2018-09-12T21:17:07.419Z"),
			"lastHeartbeatRecv" : ISODate("2018-09-12T21:17:04.600Z"),
			"pingMs" : NumberLong(0),
			"syncingTo" : "192.168.1.31:21000",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "192.168.1.33:21000",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 15,
			"optime" : {
				"ts" : Timestamp(1536787025, 3),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1536787025, 3),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2018-09-12T21:17:05Z"),
			"optimeDurableDate" : ISODate("2018-09-12T21:17:05Z"),
			"lastHeartbeat" : ISODate("2018-09-12T21:17:07.423Z"),
			"lastHeartbeatRecv" : ISODate("2018-09-12T21:17:04.671Z"),
			"pingMs" : NumberLong(1),
			"syncingTo" : "192.168.1.31:21000",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1536787025, 3),
	"$gleStats" : {
		"lastOpTime" : Timestamp(1536787012, 1),
		"electionId" : ObjectId("7fffffff0000000000000001")
	},
	"$clusterTime" : {
		"clusterTime" : Timestamp(1536787025, 3),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
config:PRIMARY> 
```





```
config = { _id : "shard1", members : [ {_id : 0, host : "192.168.1.31:27001" }, {_id : 1, host : "192.168.1.32:27001" }  ]}
```



```
> config = { _id : "shard1", members : [ {_id : 0, host : "192.168.1.31:27001" }, {_id : 1, host : "192.168.1.32:27001" }  ]}
{
	"_id" : "shard1",
	"members" : [
		{
			"_id" : 0,
			"host" : "192.168.1.31:27001"
		},
		{
			"_id" : 1,
			"host" : "192.168.1.32:27001"
		}
	]
}
> rs.initiate(config)
{ "ok" : 1 }
shard1:SECONDARY> rs.status();
{
	"set" : "shard1",
	"date" : ISODate("2018-09-12T21:25:26.061Z"),
	"myState" : 2,
	"term" : NumberLong(0),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(0, 0),
			"t" : NumberLong(-1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1536787519, 1),
			"t" : NumberLong(-1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1536787519, 1),
			"t" : NumberLong(-1)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.1.31:27001",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 131,
			"optime" : {
				"ts" : Timestamp(1536787519, 1),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("2018-09-12T21:25:19Z"),
			"infoMessage" : "could not find member to sync from",
			"configVersion" : 1,
			"self" : true
		},
		{
			"_id" : 1,
			"name" : "192.168.1.32:27001",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 6,
			"optime" : {
				"ts" : Timestamp(1536787519, 1),
				"t" : NumberLong(-1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1536787519, 1),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("2018-09-12T21:25:19Z"),
			"optimeDurableDate" : ISODate("2018-09-12T21:25:19Z"),
			"lastHeartbeat" : ISODate("2018-09-12T21:25:24.127Z"),
			"lastHeartbeatRecv" : ISODate("2018-09-12T21:25:21.225Z"),
			"pingMs" : NumberLong(1),
			"configVersion" : 1
		}
	],
	"ok" : 1
}
shard1:SECONDARY> 
```





```
config = {_id : "shard2", members : [{_id : 0, host : "192.168.1.32:27002" },{_id : 1, host : "192.168.1.33:27002" } ]}

```

```
> rs.initiate(config)
{ "ok" : 1 }
shard2:SECONDARY> rs.status()
{
	"set" : "shard2",
	"date" : ISODate("2018-09-12T21:30:09.570Z"),
	"myState" : 2,
	"term" : NumberLong(0),
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(0, 0),
			"t" : NumberLong(-1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1536787803, 1),
			"t" : NumberLong(-1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1536787803, 1),
			"t" : NumberLong(-1)
		}
	},
	"members" : [
		{
			"_id" : 0,
			"name" : "192.168.1.32:27002",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 157,
			"optime" : {
				"ts" : Timestamp(1536787803, 1),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("2018-09-12T21:30:03Z"),
			"infoMessage" : "could not find member to sync from",
			"configVersion" : 1,
			"self" : true
		},
		{
			"_id" : 1,
			"name" : "192.168.1.33:27002",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 5,
			"optime" : {
				"ts" : Timestamp(1536787803, 1),
				"t" : NumberLong(-1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1536787803, 1),
				"t" : NumberLong(-1)
			},
			"optimeDate" : ISODate("2018-09-12T21:30:03Z"),
			"optimeDurableDate" : ISODate("2018-09-12T21:30:03Z"),
			"lastHeartbeat" : ISODate("2018-09-12T21:30:08.668Z"),
			"lastHeartbeatRecv" : ISODate("2018-09-12T21:30:05.859Z"),
			"pingMs" : NumberLong(0),
			"configVersion" : 1
		}
	],
	"ok" : 1
}
shard2:SECONDARY> 
```

```
config = { _id : "shard3",members : [ {_id : 0, host : "192.168.1.31:27003" },{_id : 1, host : "192.168.1.33:27003" }] }

```



```
sh.addShard("shard1/192.168.1.31:27001,192.168.1.32:27001")
```

```
sh.addShard("shard2/192.168.1.32:27002,192.168.1.33:27002")
```

```
sh.addShard("shard3/192.168.1.33:27003,192.168.1.31:27003")
```





## 链接地址

https://www.cnblogs.com/ityouknow/p/7566682.html

