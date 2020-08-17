[TOC]

# kafka install

**文档整理**

ysys

**日期**

2018-11-24

**标签**

kafka,install



## kafka 

消息系统分类

![_](../img_src/000/2018-12-06_165236.png)

消息系统适用场景

![_](../img_src/000/2018-12-06_165455.png)

常用系统对比

![_](../img_src/000/2018-12-06_165527.png)



kafka 设计目标

![_](../img_src/000/2018-12-06_165908.png)

kafka 架构简介

![_](../img_src/000/2018-12-06_165951.png)



## kafka 2.11-0.8.2.2 install

### download uri

链接：https://pan.baidu.com/s/1KNo6yZcRNCgqtxPvewPVGQ 
提取码：fwen 



### environment

|           | version                    | note |
| --------- | -------------------------- | ---- |
| operation | CentOS release 6.5 (Final) | ..   |
| kafka     | 2.11-0.8.2.2               | ..   |
|           |                            |      |

### order

 	首先是启动zookeeper,其次启动kafka-server,之后创建topic，后面创建cunsumer和producer

```
#  ./zookeeper-server-start.sh ../config/zookeeper.properties
...
[2018-12-06 13:51:49,829] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
...
```

```
# ./kafka-server-start.sh ../config/server.properties
...
[2018-12-06 13:53:44,457] INFO Awaiting socket connections on 0.0.0.0:9092. (kafka.network.Acceptor)
[2018-12-06 13:53:44,458] INFO [Socket Server on Broker 0], Started (kafka.network.SocketServer)
...
```

```
# ./kafka-topics.sh --zookeeper localhost:2181 --create --topic ysys --partitions 3 --replication-factor 1
Created topic "ysys".
```

```
# ./kafka-topics.sh --zookeeper localhost:2181 --describe --topic ysys
Topic:ysys	PartitionCount:3	ReplicationFactor:1	Configs:
	Topic: ysys	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: ysys	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: ysys	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

```
# ./kafka-console-consumer.sh --zookeeper localhost:2181 --topic ysys
```

```
# ./kafka-console-producer.sh --broker-list localhost:9092 --topic ysys
[2018-12-06 14:35:44,007] WARN Property topic is not valid (kafka.utils.VerifiableProperties)
```





```
./zkServer.sh 
JMX enabled by default
Using config: /software/zookeeper-3.4.6/bin/../conf/zoo.cfg
grep: /software/zookeeper-3.4.6/bin/../conf/zoo.cfg: No such file or directory
mkdir: cannot create directory `': No such file or directory
Usage: ./zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}
```



```
# ./zkServer.sh  start ../conf/zoo_sample.cfg 
JMX enabled by default
Using config: ../conf/zoo_sample.cfg
Starting zookeeper ... STARTED
```



```
# lsof -i:2181
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    2827 root   21u  IPv6  18342      0t0  TCP *:eforward (LISTEN)
```









## kafka 2.11 install

### download uri

https://www.apache.org/dyn/closer.cgi?path=/kafka/2.1.0/kafka_2.11-2.1.0.tgz

### environment

|           | version       | note |
| --------- | ------------- | ---- |
| operation | centos7.4_x64 | ..   |
| kafka     | kafka2.11     | ..   |
|           |               |      |

### tar file

```
# tar zxvf kafka_2.11-2.1.0.tgz
```

###  start zookeeper

​	查看启动zookeeper参数

```
#  ./zookeeper-server-start.sh 
USAGE: ./zookeeper-server-start.sh [-daemon] zookeeper.properties
```

​	启动zookeeper

```
# ./zookeeper-server-start.sh ../config/zookeeper.properties 
...
[2018-11-24 09:01:46,194] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
[2018-11-24 09:01:46,204] INFO Reading snapshot /tmp/zookeeper/version-2/snapshot.0 (org.apache.zookeeper.server.persistence.FileSnap)
[2018-11-24 09:01:55,040] INFO Expiring session 0x167430f4e4f0000, timeout of 6000ms exceeded (org.apache.zookeeper.server.ZooKeeperServer)
[2018-11-24 09:01:55,041] INFO Processed session termination for sessionid: 0x167430f4e4f0000 (org.apache.zookeeper.server.PrepRequestProcessor)
[2018-11-24 09:01:55,041] INFO Creating new log file: log.1b (org.apache.zookeeper.server.persistence.FileTxnLog)
```

​	查看监听

```
# lsof -i:2181
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    5183 root   95u  IPv6  24589      0t0  TCP *:eforward (LISTEN)
```



### start kafba server

​	查看kafka-server-start.sh启动参数

```
# ./kafka-server-start.sh 
USAGE: ./kafka-server-start.sh [-daemon] server.properties [--override property=value]*
```

​	启动kafka-server

```
# ./kafka-server-start.sh h ../config/server.properties 
...
[2018-11-23 20:37:09,182] INFO Logs loading complete in 9 ms. (kafka.log.LogManager)
[2018-11-23 20:37:09,193] INFO Starting log cleanup with a period of 300000 ms. (kafka.log.LogManager)
[2018-11-23 20:37:09,196] INFO Starting log flusher with a default period of 9223372036854775807 ms. (kafka.log.LogManager)
[2018-11-23 20:37:09,673] INFO Awaiting socket connections on 0.0.0.0:9092. (kafka.network.Acceptor)
[2018-11-23 20:37:09,781] INFO [SocketServer brokerId=0] Started 1 acceptor threads (kafka.network.SocketServer)
[2018-11-23 20:37:09,822] INFO [ExpirationReaper-0-Produce]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-11-23 20:37:09,828] INFO [ExpirationReaper-0-Fetch]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-11-23 20:37:09,839] INFO [ExpirationReaper-0-DeleteRecords]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-11-23 20:37:09,858] INFO [LogDirFailureHandler]: Starting (kafka.server.ReplicaManager$LogDirFailureHandler)
[2018-11-23 20:37:09,872] INFO Creating /brokers/ids/0 (is it secure? false) (kafka.zk.KafkaZkClient)
[2018-11-23 20:37:09,883] INFO Result of znode creation at /brokers/ids/0 is: OK (kafka.zk.KafkaZkClient)
[2018-11-23 20:37:09,884] INFO Registered broker 0 at path /brokers/ids/0 with addresses: ArrayBuffer(EndPoint(centos72,9092,ListenerName(PLAINTEXT),PLAINTEXT)) (kafka.zk.KafkaZkClient)
[2018-11-23 20:37:09,886] WARN No meta.properties file under dir /tmp/kafka-logs/meta.properties (kafka.server.BrokerMetadataCheckpoint)
[2018-11-23 20:37:10,010] INFO [ExpirationReaper-0-topic]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-11-23 20:37:10,025] INFO [ExpirationReaper-0-Heartbeat]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-11-23 20:37:10,040] INFO [ExpirationReaper-0-Rebalance]: Starting (kafka.server.DelayedOperationPurgatory$ExpiredOperationReaper)
[2018-11-23 20:37:10,046] INFO [GroupCoordinator 0]: Starting up. (kafka.coordinator.group.GroupCoordinator)
[2018-11-23 20:37:10,057] INFO Successfully created /controller_epoch with initial epoch 0 (kafka.zk.KafkaZkClient)
[2018-11-23 20:37:10,068] INFO [GroupCoordinator 0]: Startup complete. (kafka.coordinator.group.GroupCoordinator)
[2018-11-23 20:37:10,089] INFO [GroupMetadataManager brokerId=0] Removed 0 expired offsets in 1 milliseconds. (kafka.coordinator.group.GroupMetadataManager)
[2018-11-23 20:37:10,102] INFO [ProducerId Manager 0]: Acquired new producerId block (brokerId:0,blockStartProducerId:0,blockEndProducerId:999) by writing to Zk with path version 1 (kafka.coordinator.transaction.ProducerIdManager)
[2018-11-23 20:37:10,159] INFO [TransactionCoordinator id=0] Starting up. (kafka.coordinator.transaction.TransactionCoordinator)
[2018-11-23 20:37:10,199] INFO [TransactionCoordinator id=0] Startup complete. (kafka.coordinator.transaction.TransactionCoordinator)
[2018-11-23 20:37:10,211] INFO [Transaction Marker Channel Manager 0]: Starting (kafka.coordinator.transaction.TransactionMarkerChannelManager)
[2018-11-23 20:37:10,382] INFO [/config/changes-event-process-thread]: Starting (kafka.common.ZkNodeChangeNotificationListener$ChangeEventProcessThread)
[2018-11-23 20:37:10,391] INFO [SocketServer brokerId=0] Started processors for 1 acceptors (kafka.network.SocketServer)
[2018-11-23 20:37:10,393] INFO Kafka version : 2.1.0 (org.apache.kafka.common.utils.AppInfoParser)
[2018-11-23 20:37:10,393] INFO Kafka commitId : 809be928f1ae004e (org.apache.kafka.common.utils.AppInfoParser)
[2018-11-23 20:37:10,394] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```

​	查看监听

```
# lsof -i:9092
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    8998 root  102u  IPv6  47389      0t0  TCP *:XmlIpcRegSvc (LISTEN)
java    8998 root  118u  IPv6  47395      0t0  TCP centos72:37382->centos72:XmlIpcRegSvc (ESTABLISHED)
java    8998 root  119u  IPv6  47396      0t0  TCP centos72:XmlIpcRegSvc->centos72:37382 (ESTABLISHED)
```

```
# lsof -i:2181
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    8643 root   97u  IPv6  46944      0t0  TCP *:eforward (LISTEN)
java    8643 root   98u  IPv6  47387      0t0  TCP localhost:eforward->localhost:47872 (ESTABLISHED)
java    8998 root   97u  IPv6  47386      0t0  TCP localhost:47872->localhost:eforward (ESTABLISHED)
```



###  create topic

​	创建一个主题

​	查看topics参数

```
./kafka-topics.sh 
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
Create, delete, describe, or change a topic.
Option                                   Description                            
------                                   -----------                            
--alter                                  Alter the number of partitions,        
                                           replica assignment, and/or           
                                           configuration for the topic.         
--config <String: name=value>            A topic configuration override for the 
                                           topic being created or altered.The   
                                           following is a list of valid         
                                           configurations:                      
                                         	cleanup.policy                        
                                         	compression.type                      
                                         	delete.retention.ms                   
                                         	file.delete.delay.ms                  
                                         	flush.messages                        
                                         	flush.ms                              
                                         	follower.replication.throttled.       
                                           replicas                             
                                         	index.interval.bytes                  
                                         	leader.replication.throttled.replicas 
                                         	max.message.bytes                     
                                         	message.downconversion.enable         
                                         	message.format.version                
                                         	message.timestamp.difference.max.ms   
                                         	message.timestamp.type                
                                         	min.cleanable.dirty.ratio             
                                         	min.compaction.lag.ms                 
                                         	min.insync.replicas                   
                                         	preallocate                           
                                         	retention.bytes                       
                                         	retention.ms                          
                                         	segment.bytes                         
                                         	segment.index.bytes                   
                                         	segment.jitter.ms                     
                                         	segment.ms                            
                                         	unclean.leader.election.enable        
                                         See the Kafka documentation for full   
                                           details on the topic configs.        
--create                                 Create a new topic.                    
--delete                                 Delete a topic                         
--delete-config <String: name>           A topic configuration override to be   
                                           removed for an existing topic (see   
                                           the list of configurations under the 
                                           --config option).                    
--describe                               List details for the given topics.     
--disable-rack-aware                     Disable rack aware replica assignment  
--exclude-internal                       exclude internal topics when running   
                                           list or describe command. The        
                                           internal topics will be listed by    
                                           default                              
--force                                  Suppress console prompts               
--help                                   Print usage information.               
--if-exists                              if set when altering or deleting       
                                           topics, the action will only execute 
                                           if the topic exists                  
--if-not-exists                          if set when creating topics, the       
                                           action will only execute if the      
                                           topic does not already exist         
--list                                   List all available topics.             
--partitions <Integer: # of partitions>  The number of partitions for the topic 
                                           being created or altered (WARNING:   
                                           If partitions are increased for a    
                                           topic that has a key, the partition  
                                           logic or ordering of the messages    
                                           will be affected                     
--replica-assignment <String:            A list of manual partition-to-broker   
  broker_id_for_part1_replica1 :           assignments for the topic being      
  broker_id_for_part1_replica2 ,           created or altered.                  
  broker_id_for_part2_replica1 :                                                
  broker_id_for_part2_replica2 , ...>                                           
--replication-factor <Integer:           The replication factor for each        
  replication factor>                      partition in the topic being created.
--topic <String: topic>                  The topic to be create, alter or       
                                           describe. Can also accept a regular  
                                           expression except for --create option
--topics-with-overrides                  if set when describing topics, only    
                                           show topics that have overridden     
                                           configs                              
--unavailable-partitions                 if set when describing topics, only    
                                           show partitions whose leader is not  
                                           available                            
--under-replicated-partitions            if set when describing topics, only    
                                           show under replicated partitions     
--zookeeper <String: hosts>              REQUIRED: The connection string for    
                                           the zookeeper connection in the form 
                                           host:port. Multiple hosts can be     
                                           given to allow fail-over.            
```



​	开启一个主题

```
# ./kafka-topics.sh --zookeeper localhost:2181 --create --topic ysys --partitions 3 --replication-factor 1
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
Created topic "ysys".
```

​	展示一个主题

```
# ./kafka-topics.sh --zookeeper localhost:2181 --describe --topic ysys 
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
Topic:ysys	PartitionCount:3	ReplicationFactor:1	Configs:
	Topic: ysys	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: ysys	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: ysys	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

​	

### crate cunsumer producer

​	创建生产者消费者

​	创建消费者命令参数

```
# ./kafka-console-consumer.sh 
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
The console consumer is a tool that reads data from Kafka and outputs it to standard output.
Option                                   Description                            
------                                   -----------                            
--bootstrap-server <String: server to    REQUIRED: The server(s) to connect to. 
  connect to>                                                                   
--consumer-property <String:             A mechanism to pass user-defined       
  consumer_prop>                           properties in the form key=value to  
                                           the consumer.                        
--consumer.config <String: config file>  Consumer config properties file. Note  
                                           that [consumer-property] takes       
                                           precedence over this config.         
--enable-systest-events                  Log lifecycle events of the consumer   
                                           in addition to logging consumed      
                                           messages. (This is specific for      
                                           system tests.)                       
--formatter <String: class>              The name of a class to use for         
                                           formatting kafka messages for        
                                           display. (default: kafka.tools.      
                                           DefaultMessageFormatter)             
--from-beginning                         If the consumer does not already have  
                                           an established offset to consume     
                                           from, start with the earliest        
                                           message present in the log rather    
                                           than the latest message.             
--group <String: consumer group id>      The consumer group id of the consumer. 
--isolation-level <String>               Set to read_committed in order to      
                                           filter out transactional messages    
                                           which are not committed. Set to      
                                           read_uncommittedto read all          
                                           messages. (default: read_uncommitted)
--key-deserializer <String:                                                     
  deserializer for key>                                                         
--max-messages <Integer: num_messages>   The maximum number of messages to      
                                           consume before exiting. If not set,  
                                           consumption is continual.            
--offset <String: consume offset>        The offset id to consume from (a non-  
                                           negative number), or 'earliest'      
                                           which means from beginning, or       
                                           'latest' which means from end        
                                           (default: latest)                    
--partition <Integer: partition>         The partition to consume from.         
                                           Consumption starts from the end of   
                                           the partition unless '--offset' is   
                                           specified.                           
--property <String: prop>                The properties to initialize the       
                                           message formatter. Default           
                                           properties include:                  
                                         	print.timestamp=true|false            
                                         	print.key=true|false                  
                                         	print.value=true|false                
                                         	key.separator=<key.separator>         
                                         	line.separator=<line.separator>       
                                         	key.deserializer=<key.deserializer>   
                                         	value.deserializer=<value.            
                                           deserializer>                        
                                         Users can also pass in customized      
                                           properties for their formatter; more 
                                           specifically, users can pass in      
                                           properties keyed with 'key.          
                                           deserializer.' and 'value.           
                                           deserializer.' prefixes to configure 
                                           their deserializers.                 
--skip-message-on-error                  If there is an error when processing a 
                                           message, skip it instead of halt.    
--timeout-ms <Integer: timeout_ms>       If specified, exit if no message is    
                                           available for consumption for the    
                                           specified interval.                  
--topic <String: topic>                  The topic id to consume on.            
--value-deserializer <String:                                                   
  deserializer for values>                                                      
--whitelist <String: whitelist>          Whitelist of topics to include for     
                                           consumption.                         
```

​	创建消费者

```
# ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ysys
```



​	创建生产者参数

```
# ./kafka-console-producer.sh 
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
Read data from standard input and publish it to Kafka.
Option                                   Description                            
------                                   -----------                            
--batch-size <Integer: size>             Number of messages to send in a single 
                                           batch if they are not being sent     
                                           synchronously. (default: 200)        
--broker-list <String: broker-list>      REQUIRED: The broker list string in    
                                           the form HOST1:PORT1,HOST2:PORT2.    
--compression-codec [String:             The compression codec: either 'none',  
  compression-codec]                       'gzip', 'snappy', 'lz4', or 'zstd'.  
                                           If specified without value, then it  
                                           defaults to 'gzip'                   
--line-reader <String: reader_class>     The class name of the class to use for 
                                           reading lines from standard in. By   
                                           default each line is read as a       
                                           separate message. (default: kafka.   
                                           tools.                               
                                           ConsoleProducer$LineMessageReader)   
--max-block-ms <Long: max block on       The max time that the producer will    
  send>                                    block for during a send request      
                                           (default: 60000)                     
--max-memory-bytes <Long: total memory   The total memory used by the producer  
  in bytes>                                to buffer records waiting to be sent 
                                           to the server. (default: 33554432)   
--max-partition-memory-bytes <Long:      The buffer size allocated for a        
  memory in bytes per partition>           partition. When records are received 
                                           which are smaller than this size the 
                                           producer will attempt to             
                                           optimistically group them together   
                                           until this size is reached.          
                                           (default: 16384)                     
--message-send-max-retries <Integer>     Brokers can fail receiving the message 
                                           for multiple reasons, and being      
                                           unavailable transiently is just one  
                                           of them. This property specifies the 
                                           number of retires before the         
                                           producer give up and drop this       
                                           message. (default: 3)                
--metadata-expiry-ms <Long: metadata     The period of time in milliseconds     
  expiration interval>                     after which we force a refresh of    
                                           metadata even if we haven't seen any 
                                           leadership changes. (default: 300000)
--producer-property <String:             A mechanism to pass user-defined       
  producer_prop>                           properties in the form key=value to  
                                           the producer.                        
--producer.config <String: config file>  Producer config properties file. Note  
                                           that [producer-property] takes       
                                           precedence over this config.         
--property <String: prop>                A mechanism to pass user-defined       
                                           properties in the form key=value to  
                                           the message reader. This allows      
                                           custom configuration for a user-     
                                           defined message reader.              
--request-required-acks <String:         The required acks of the producer      
  request required acks>                   requests (default: 1)                
--request-timeout-ms <Integer: request   The ack timeout of the producer        
  timeout ms>                              requests. Value must be non-negative 
                                           and non-zero (default: 1500)         
--retry-backoff-ms <Integer>             Before each retry, the producer        
                                           refreshes the metadata of relevant   
                                           topics. Since leader election takes  
                                           a bit of time, this property         
                                           specifies the amount of time that    
                                           the producer waits before refreshing 
                                           the metadata. (default: 100)         
--socket-buffer-size <Integer: size>     The size of the tcp RECV size.         
                                           (default: 102400)                    
--sync                                   If set message send requests to the    
                                           brokers are synchronously, one at a  
                                           time as they arrive.                 
--timeout <Integer: timeout_ms>          If set and the producer is running in  
                                           asynchronous mode, this gives the    
                                           maximum amount of time a message     
                                           will queue awaiting sufficient batch 
                                           size. The value is given in ms.      
                                           (default: 1000)                      
--topic <String: topic>                  REQUIRED: The topic id to produce      
                                           messages to.                         
```

​	创建生产者

```
# ./kafka-console-producer.sh --broker-list localhost:9092 --topic ysys
```



​	在生产者传入相关信息

```
# ./kafka-console-producer.sh --broker-list localhost:9092 --topic ysys
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
>2
>3
>4
>5
>ysys is cool   
>
```

​	在消费者消费相关信息

```
# ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic ysys
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
4
5
ysys is cool
```

​	

## learn 

链接：https://pan.baidu.com/s/1_1lh_8hRpwTmVLBSymNjhw 
提取码：4tfh 




## link

http://www.dataguru.cn/mycourse.php?lessonid=1731

http://kafka.apache.org/documentation.html#quickstart_download






## error

error 1:无法启动 kafka-server

```
# bin/kafka-server-start.sh config/server.properties 
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/apache/kafka/common/utils/KafkaThread : Unsupported major.minor version 52.0
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:800)
	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142)
	at java.net.URLClassLoader.defineClass(URLClassLoader.java:449)
	at java.net.URLClassLoader.access$100(URLClassLoader.java:71)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:361)
	at java.net.URLClassLoader$1.run(URLClassLoader.java:355)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:354)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:425)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:308)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:358)
	at kafka.utils.Log4jControllerRegistration$.<init>(Logging.scala:30)
	at kafka.utils.Log4jControllerRegistration$.<clinit>(Logging.scala)
	at kafka.utils.Logging$class.$init$(Logging.scala:47)
	at kafka.Kafka$.<init>(Kafka.scala:30)
	at kafka.Kafka$.<clinit>(Kafka.scala)
	at kafka.Kafka.main(Kafka.scala)
```

 jdk版本需要升级到jdk1.8









  