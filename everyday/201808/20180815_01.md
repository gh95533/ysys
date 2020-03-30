[TOC]

# pg json jsonb 插入 查询



​	今天早上在赶飞机的时候，同事打过来电话，说了这样一个需求：JL的solr产品将一堆的标签存储到oracle的clob字段，由于标签各不相同，所以每条数据都是各不相同的，而用户需要统计某些字段或者某类字段信息数据，是否符合规范,各类信息统计



## 现状或要求

​	

​	1、每条数据的标签可能都不是一样

​	2、数据格式类似json

​	3、统计按照各个维度开展



## 解决思路

​	

​	首先不适合使用常规的数据软件解析数据

​	其次json结构存储数据库常用是mongo,但是mongo不适合实施使用

​	最后建议选择postgres,postgres版本是9.4.12及以上



## 插入



```
tutorial=# create table test_jsonb_info(id int , info jsonb);

tutorial=# create table test_jsonb_info(id int,info jsonb);
CREATE TABLE

tutorial=# insert into test_jsonb_info values(1,'{
    "guid": "9c36adc1-7fb5-4d5b-83b4-90356a46061a",
    "name": "Angela Barton",
    "is_active": true,
    "company": "Magnafone",
    "address": "178 Howard Place, Gulf, Washington, 702",
    "registered": "2009-11-07T08:53:22 +08:00",
    "latitude": 19.793713,
    "longitude": 86.513373,
    "tags": [
        "enim",
        "aliquip",
        "qui"
    ]
}');

tutorial=# select  * from test_jsonb_info ;
-[ RECORD 1 ]---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
id   | 1
info | {"guid": "9c36adc1-7fb5-4d5b-83b4-90356a46061a", "name": "Angela Barton", "tags": ["enim", "aliquip", "qui"], "address": "178 Howard Place, Gulf, Washington, 702", "company": "Magnafone", "latitude": 19.793713, "is_active": true, "longitude": 86.513373, "registered": "2009-11-07T08:53:22 +08:00"}

tutorial=# 

```



## 查询

查询info - > 'colum_name 1',info -> 'column_name 2'具体字段

```
tutorial=# select id,info -> 'guid' as guid,info-> 'name' as name,info -> 'tags',info -> 'unknown' as unknown,info from test_jsonb_info ;
-[ RECORD 1 ]-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
id       | 1
guid     | "9c36adc1-7fb5-4d5b-83b4-90356a46061a"
name     | "Angela Barton"
?column? | ["enim", "aliquip", "qui"]
unknown  | 
info     | {"guid": "9c36adc1-7fb5-4d5b-83b4-90356a46061a", "name": "Angela Barton", "tags": ["enim", "aliquip", "qui"], "address": "178 Howard Place, Gulf, Washington, 702", "company": "Magnafone", "latitude": 19.793713, "is_active": true, "longitude": 86.513373, "registered": "2009-11-07T08:53:22 +08:00"}

tutorial=# 

```



## 优化

--`jsonb`缺省的GIN操作符类支持使用`@>`、`?`、 `?&`和`?|`操作符查询 

```
tutorial=# create index idx_test_jsonb_info on test_jsonb_info  using gin(info);
CREATE INDEX

```



