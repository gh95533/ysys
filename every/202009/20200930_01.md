[TOC]

# greenplum synchronization time

**文档整理**

ysys

**日期**

2018-11-27

**标签**

greenplum,sysnchronization time



## 背景

​	在HT项目中，有一次机房异常断电，当操作系统和gp数据库启动后，发现一个问题，就是操作时间变前了。之前正常时间是早上8点，它变成了下午8点了。



## 处理

​	

### 停止数据库

```
$ gpstop -af
```

### 主节点同步时间

```
# ntpdate -s ip
```

### 所有节点同步时间

```
# gpssh -f {path}/seg_hosts -e 'ntpdate -s {主节点ip}'
```

### 启动数据库

```
$ gpstart -a
```

### 检查数据库时间

```
select now()
```

​	修改完成，在近一步的时间检查时间问题





