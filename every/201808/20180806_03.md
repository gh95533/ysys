[TOC]

# postgresql savepoint

​	在生产环境下，不论是在oracle还是在postgresql,普通都是用的begin commit两个事务标记，一个代表事务开始，一个代表事务提交，或者还有一个是rollback,事务回滚

​	那么这篇文章来介绍一下savepoint在postgresql的作用

## savepoint

​	在官网文档的定义:define a new savepoint within  the current transaction (在当前会话中定义一个新的保存点)。

​	savepoint 在当前事务里建立一个新的保存点。

​	保存点是事务中的一个特殊记号，它允许将那些在它建立后执行的命令全部回滚， 把事务的状态恢复到保存点所在的时刻。

### 语法

```
savepoint savepoint_name
```

### 参数

`savepoint_name`赋予新保存点的名字



### 注意

​	使用[ROLLBACK TO SAVEPOINT](http://www.postgres.cn/docs/9.4/sql-rollback-to.html)回滚到一个保存点。使用[RELEASE SAVEPOINT](http://www.postgres.cn/docs/9.4/sql-release-savepoint.html) 删除一个保存点，但是保留该保存点建立后执行的命令的效果 



### 例子

1、建立一个保存点，稍后撤销到这个保存点建立后执行的所有命令的结果

```
BEGIN;
    INSERT INTO test1 VALUES (1);
    SAVEPOINT my_savepoint;
    INSERT INTO test1 VALUES (2);
    ROLLBACK TO SAVEPOINT my_savepoint;
    INSERT INTO test1 VALUES (3);
COMMIT;
```

```
tutorial=# select * from test1;
 id 
----
  1
  3
(2 rows)

```

上面的事务回滚到my_savepoint后，再次执行insert into test1 values(3)，就不会插入2



2、建立两个同名的保存点，稍后撤销到这个保存点建立后执行的所有命令的结果

```
BEGIN;
    INSERT INTO test1 VALUES (1);
    SAVEPOINT my_savepoint;
    INSERT INTO test1 VALUES (2);
    SAVEPOINT my_savepoint;
    INSERT INTO test1 VALUES (3);
    ROLLBACK TO SAVEPOINT my_savepoint;
    INSERT INTO test1 VALUES (4);
COMMIT;
```

```
tutorial=# select * from test1;
 id 
----
  1
  2
  4
(3 rows)

```

在postgresql里，出现多个同名保存点，但是在回滚或者释放的时候，只使用最近的一个



3、建立并稍后删除一个保存点

```
BEGIN;
    INSERT INTO test1 VALUES (3);
    SAVEPOINT my_savepoint;
    INSERT INTO test1 VALUES (4);
    RELEASE SAVEPOINT my_savepoint;
COMMIT;
```

```
tutorial=# select * from test1;
 id 
----
  3
  4
(2 rows)

```

删除一个保存点，但是保留该保存点建立后执行的命令的效果 ,就等于没有出现过



## 链接地址

https://www.postgresql.org/docs/9.1/static/sql-savepoint.html

