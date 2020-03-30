[TOC]

# oracle partition lot's not check

**文档整理**

ysys

**日期**

2018-11-22

**标签**

oracle,partition,lot's not check



## oracle partition

​	今天在查看一个分区表结构时，发现无法打开，但是可以通过`select * from tableA where rownum < 10000`也是可以查询的，这是为什么?

​	查询相关文档，发现Oracle 分区表有限制



Oracle11g有关分区的限制如下：
Partitions Maximum length of linear partitioning key 4 KB - overhead
Partitions Maximum number of columns in partition key 16 columns
Partitions Maximum number of partitions allowed per table or index 1024K - 1











## 链接地址

https://www.oracle.com/technetwork/cn/articles/11g-partitioning-089831-zhs.html

https://blog.csdn.net/wyzxg/article/details/3959567

https://yq.aliyun.com/articles/34775

https://community.oracle.com/message/10847126

