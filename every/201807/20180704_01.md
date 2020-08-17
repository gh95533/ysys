[TOC]

# pg_模糊匹配和正则表达式

​	在oracle接触过like和正则表达式来查询数据，postgresql同样也是如此，这里为大家介绍三种实现模式匹配的方法。



## postgresql中模式匹配和正则表达式介绍

- 传统类型like语句

- sql99新增的similar to 操作符

- posix风格的正则表达式

  另外还有一种匹配函数substring可用。

## like



​	其中百分号"%"代表0个或者任意多个字符，而下划线"_"代表任意个字符

​	

​	初始化脚本

```plsql
tutorial=# create table test01(id int4,note text);
CREATE TABLE
tutorial=# insert into test01 values(1,'abcabefg'),(2,'abxyz'),(3,'123abe');
INSERT 0 3
tutorial=# insert into test01 values(4,'ab_abefg');
INSERT 0 1
tutorial=# insert into test01 values(5,'ab%abefg');
INSERT 0 1
tutorial=# insert into test01 values(6,'ababefg');
INSERT 0 1
```

​	一般查询

```plsql
tutorial=# select * from test01 where note like 'ab%';
 id |   note   
----+----------
  1 | abcabefg
  2 | abxyz
  4 | ab_abefg
  5 | ab%abefg
  6 | ababefg
(5 rows)

tutorial=# select * from test01 where note like'ab_ab%';
 id |   note   
----+----------
  1 | abcabefg
  4 | ab_abefg
  5 | ab%abefg
(3 rows)

```

​	匹配字符串中的百分号自身或者下划线"_",可以在字符串前加转义字符反斜杠"\\"

```plsql
tutorial=# select * from test01 where note like'ab\_ab%';
 id |   note   
----+----------
  4 | ab_abefg
(1 row)

tutorial=# select * from test01 where note like'ab\%ab%';
 id |   note   
----+----------
  5 | ab%abefg
(1 row)

```

​	转义字符也可以使用ESCAPE子句来指定成其他的字符

```plsql
tutorial=# select * from test01 where note like'ab#%ab%' escape '#';
 id |   note   
----+----------
  5 | ab%abefg
(1 row)

tutorial=# select * from test01 where note like'ab#_ab%' escape '#';
 id |   note   
----+----------
  4 | ab_abefg
(1 row)


```

## SIMILAR TO 正则表达式

​	

​	similar to 操作符只有在匹配整个字符串时才能成功，这一点和like相同，而这与普通的正则表达式只匹配部分的习惯是不同的。similar to与like一样，也是用下划线和百分好分别匹配单个字符和任意字符串。

​	similar to还支持如下与POSIX正则表达式相同的模糊匹配元字符

 - |

   表示选择两个候选项之一

- *

  表示重复前面的项零次或更多次

- +

  表示重复前面的项一次或更多次

- ？

  表示重复前面的项零次或一次

- {m}

  表示重复前面的项m次

- {m,}

  表示重复前面的项m次或更多次

- {m,n}

  表示重复前面的项至少m次，不超过n次[m,n)

- ()

  表示可以作为项目分组到一个独立的逻辑项中

- [....]

  声明一个字符类，就像POSIX正则表达式

  ```
  tutorial=# select 'osdba' similar to 'a';
   ?column? 
  ----------
   f
  (1 row)
  
  tutorial=# select 'osdba' similar to '%a%';
   ?column? 
  ----------
   t
  (1 row)
  
  tutorial=# select 'osdba' similar to '%(b|a)';
   ?column? 
  ----------
   t
  (1 row)
  
  tutorial=# select 'osdba' similar to '(b|a)%';
   ?column? 
  ----------
   f
  (1 row)
  
  ```



## POSIX 正则表达式



​	posix正则表达式的模式匹配操作符有如下几种：

​	~ :匹配正则表达式，区分大小写

​	~*：匹配正则表达式，不分大小写

​	!~:不匹配正则表达式，区分大小写

​        !~*:不匹配正则表达式，不分大小写

​	

```
tutorial=# select 'osdba' ~ 'a';
 ?column? 
----------
 t
(1 row)

tutorial=# select 'osdba' ~ '{b|a}*';
 ?column? 
----------
 t
(1 row)
osdba=# select 'osdb' ~ '.*(b|a).*';
 ?column? 
----------
 t
(1 row)

osdba=# select 'osdba' ~ '(s|d).*';
 ?column? 
----------
 t
(1 row)


```



## substring



**略过**



​	



