[TOC]

# Studio 3T for Mongodb

​	文章来自nyfedit7

​	Studio 3T for Mongodb是mongodb的客户端软件，官网上好像只有x64，所以尽量使用64的操作系统(后期发现有32位安装包，只是没有测试是否可以使用)。

## 下载地址

官网地址：

https://studio3t.com/download/

百度地址：

https://pan.baidu.com/s/1Du4XxMVXuYnl_rN8AsMntg （x64）password:85mi

https://pan.baidu.com/s/1DQDR_nlQrQQWPBO_WfihtA   (x32) password:havl



建议使用64位客户端



## 安装步骤

傻瓜式操作，一步一步的执行下一步

## 连接数据库

一般而言，数据库默认都是没有用户名，密码；

点击桌面上图标后，点击"Connect"，出现"New Collection"

在"New Connection"输入connection name ,server ,port;后面有Authentication中是有输入user/password;记得点击保存。


连接成功后，重新点击连接地址后，

安装软件完成

## sql语句

如果打开sql界面，右击表名，出现"Open Sql",右侧工作界面出现sql界面，填写完sql后，点击按钮执行

```
1、查询语句

select * from tablename;

select *

from col;  

2、where语句

select * from tablename where column_name = 'column_value';

select *

from col where title = 'ddd';

3、and or

select * from tablename where column_name_A='column_value_A' and column_name_B='column_value_B';

select * from tablename where column_name_A='column_value_A' or column_name_B='column_value_B';

select *

from col where title = 'ddd' and "by" ='菜鸟教程';

select *

from col where title = 'ddd' or "by" ='菜鸟教程';

4、like 语句

select * from tablename where column_name like '%column_value%';

select *

from col where title like '%d%' ;

5、in 语句

select * from tablename where column_name in ( column_value_A , column_value_B）

select *

from col where title in ('ddd','dfdfd') ;

6、between and

select * from tablename where column_name between column_value_A and column_value_B;

select *

from col where likes between 150 and 1111;

支持number和varchar,date类型

7、date,time

select *

from dates_example

where d > date('2017-03-22T00:00:00.000Z');

select *

from dates_example

where d > date(‘2017-03-22T00:00:00.000+0000’);

select *

from dates_example

where d > date(‘2017-03-22T00:00:00.000’);

select *

from dates_example

where d > date(‘2017-03-22T00:00:00’);

select *

from dates_example

where d > date(‘2017-03-22T00:00’);

select *

from dates_example

where d > date(‘2017-03-22 00:00:00.000Z’);

select *

from dates_example

where d > date(‘2017-03-22 00:00:00.000+0000’);

select *

from dates_example

where d > date(‘2017-03-22 00:00:00.000’);

select *

from dates_example

where d > date(‘2017-03-22 00:00:00’);

select *

from dates_example

where d > date(‘2017-03-22 00:00’);

select *

from dates_example

where d > date(‘20170322T000000.000Z’);

select *

from dates_example

where d > date(‘20170322T000000.000+0000’);

select *

from dates_example

where d > date(‘20170322T000000.000’);

select *

from dates_example

where d > date(‘20170322T000000’);

select *

from dates_example

where d > date(‘20170322T0000’);

select *

from dates_example

where d > date(‘2017-03-22’);

select *

from dates_example

where d > date(‘20170322’);

8、> = <= >=

select column_nameX from tablename where column_name > 'column_value';

9、order by  column_name (desc)

select * from tablename order by column_name 

10、group by  having count(*） 

select  column_name, count() group by tablename group by column_name having count() > num;

select name,count(*)

from col  group by name having count(*) >1

11、inner|left|cross join on 

select col2.name

from col2  inner join col  on col2.name = col.name;



```

​	经过上述测试发现,Studio 3T兼容的普通sql语句，建议使用Studio 3T for Mongodb来跳过普通的mongodb查询。

备注：

mongodb数据库对于大小写特别敏感，如果表名，字段名不是小写，那么就要按照它的展示字段来查询。
mongodb数据库对于关键字时，如果是查询时，可能需要添加""才可以使用

## 链接地址

https://studio3t.com/whats-new/how-to-query-mongodb-with-sql/

https://www.cnblogs.com/yuechaotian/archive/2013/02/02/2889824.html

