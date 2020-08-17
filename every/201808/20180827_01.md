[TOC]

# gbase 库 统计表数据量

​	gbase 数据库 很坑，(8a)存储过程不允许写入到表中，有一天需要统计一个库下n张表的数据量。

​	请找到information_schema.tables中找到table_schema，table_rows，如果table_rows为0，说明数据没有收集统计信息，而且从目前索引支持来看，只是支持hash索引，这对于表关联，模糊查询，范围查询，动态语句可能有影响。

​	select table_schema,table_name,table_rows from tables where table_schema='' and  table_name=''

​	发现数据量为NULL,说明统计信息并没有有效收集



​	select ‘select count(1) from ’||table_schema||'.'||table_name||' union all' from tables where table_schema='tx_bzk'