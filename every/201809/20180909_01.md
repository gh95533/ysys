[TOC]

# pg 德哥 宣讲会 

## 主要内容

​	宣讲会当前主要内容 postgresql,ppas,hdb for pg，云生态,产品指南，企业全栈应用案例，开发管理实践

## 关注点



​	关于pg新版本与老版本选型(10？9？)

​	pg的json,jsonb与mongo的bjson的对比，查询，插入，更新，分布式，索引

​	pg对于其他数据库的fdw支持，是否随着版本迭代后期不支持(9.6,9.4|jar)

​	pg的索引,部分索引,空间索引

​	pg的全文搜索ts_vector,ts_query

​	pg的函数三态

​	mysql,oracle,sqlserver,db2迁移到pg的特性是否保留

​	pg的xid,xmax,xmin,cmax,cmin,current_xid

​	pg的隔离级别

​	pg的内存管理，checkpoint,xlog,clog

​	集成pg的mpp数据库 gp(高并发?) Libra(查询)

​	pg继承:表 分区表  分区索引   索引

​	pg 级联复制	

​	cstone fdw:列式数据库,最新的版本支持wal日志



​	误删数据

​	数据库级别 实例 pg 基础备份+归档备份

​	数据页 pg 多版本 

​	

​	神州飞象

​	12306 短信 高铁 订餐 

​	分布式数据库 pg-xl

​	高可用  postgresql+同步流+异步流	



## 内容

### 阿里云产品	

​	阿里云 RDS for PG 开源增强版

​	云数据库 PPAS 版

​	HybridDB for PostgreSQL

​	PolarDB ofr PostgreSQL

### 产品体系

​	处理空间

​	no-sql(mongo,redis)

​	全文搜索

​	数据仓库

### 传统架构痛点

​	跨数据库痛点:

​	成本痛点:

### 全栈式数据库

​	流式计算

​	外部表

​	sharding

​	时空

#### 插件

​	pg-strom/..

#### 单表管理能力

​	单表十亿性能几乎无衰减

​	插入，更新（更新?）

#### 垂直扩展能力

​	资源隔离(mpp)

​	

#### 跨机房容灾能力

​	逻辑订阅

​	一写多读

#### sharding

​	citus

​	fdw



###  postgresql ap 特性

​	单机

​	 parallel cpu加速

​	gpu 加速

​	多机

​	citus

​	pg shared

​	...

​	向量计算

​	JIT





case array 权重 数据	

case 流式计算





















​	





​	



​	







