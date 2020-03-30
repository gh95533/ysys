[TOC]

# postgresql with query update delete 递归



​	

​	with 提供了一种在更大的查询中编写辅助语句的方式。每个with子句中的辅助语句可以是一个SELECT,INSERT,UPDATE或DELETE；并且WITH子句本身附加到初级语句可以时一个SELECT,INSERT ,UPDATE或者DELETE



## with 	



### with 语法



```
Command:     WITH
Description: retrieve rows from a table or view
Syntax:
[ WITH [ RECURSIVE ] with_query [, ...] ]
SELECT [ ALL | DISTINCT [ ON ( expression [, ...] ) ] ]
    [ * | expression [ [ AS ] output_name ] [, ...] ]
    [ FROM from_item [, ...] ]
    [ WHERE condition ]
    [ GROUP BY expression [, ...] ]
    [ HAVING condition [, ...] ]
    [ WINDOW window_name AS ( window_definition ) [, ...] ]
    [ { UNION | INTERSECT | EXCEPT } [ ALL | DISTINCT ] select ]
    [ ORDER BY expression [ ASC | DESC | USING operator ] [ NULLS { FIRST | LA
ST } ] [, ...] ]
    [ LIMIT { count | ALL } ]
    [ OFFSET start [ ROW | ROWS ] ]
    [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } ONLY ]
    [ FOR { UPDATE | NO KEY UPDATE | SHARE | KEY SHARE } [ OF table_name [, ..
.] ] [ NOWAIT ] [...] ]

where from_item can be one of:

    [ ONLY ] table_name [ * ] [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
    [ LATERAL ] ( select ) [ AS ] alias [ ( column_alias [, ...] ) ]
    with_query_name [ [ AS ] alias [ ( column_alias [, ...] ) ] ]
    [ LATERAL ] function_name ( [ argument [, ...] ] )
                [ WITH ORDINALITY ] [ [ AS ] alias [ ( column_alias [, ...] ) 
] ]
    [ LATERAL ] function_name ( [ argument [, ...] ] ) [ AS ] alias ( column_d
efinition [, ...] )
    [ LATERAL ] function_name ( [ argument [, ...] ] ) AS ( column_definition 
[, ...] )
    [ LATERAL ] ROWS FROM( function_name ( [ argument [, ...] ] ) [ AS ( colum
n_definition [, ...] ) ] [, ...] )
                [ WITH ORDINALITY ] [ [ AS ] alias [ ( column_alias [, ...] ) 
] ]
    from_item [ NATURAL ] join_type from_item [ ON join_condition | USING ( jo
in_column [, ...] ) ]

and with_query is:

    with_query_name [ ( column_name [, ...] ) ] AS ( select | values | insert 
| update | delete )

TABLE [ ONLY ] table_name [ * ]

```



### with 查询

```
WITH regional_sales AS (
        SELECT region, SUM(amount) AS total_sales
        FROM orders
        GROUP BY region
     ), top_regions AS (
        SELECT region
        FROM regional_sales
        WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
     )
SELECT region,
       product,
       SUM(quantity) AS product_units,
       SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, 
```



### with 递归

```
 with recursive t1(id ,code ,name) as (
select id,code,name from test1 where code is null
union 
select f.id,f.code,f.name from test1 f,t1 where f.code=t1.id)
select * from t1;

```



要避免递归无限循环



### with  数据更新



例1

```
tutorial=# with t as ( delete from test1 where id = 1 returning *) insert into test2 select * from t;
INSERT 0 1
tutorial=# select * from test2;
 id | code | name  
----+------+-------
  1 |      | guo i
(1 row)

```

 delete删除的数据当作返回值返回给了表t，而表t的数据插入到test2中

在 with 中的数据修改语句通常都有RETURNING子句，就像上面的例子一样，它是RETURNING子句的输出。如果 WITH 中的数据修改语句缺少了 RETURNING 子句，那么将没有临时表生成，也就不可能被其他的查询引用。这样的语句当然可以执行，如下例

例2

```
tutorial=# select count(1) from test2;
 count 
-------
     1
(1 row)

tutorial=# select count(1) from test3;
 count 
-------
     0
(1 row)

tutorial=# with t as (delete from test2) delete from test3;
DELETE 0
tutorial=# select count(1) from test2;
 count 
-------
     0
(1 row)

tutorial=# select count(1) from test3;
 count 
-------
     0
(1 row)

tutorial=# 

```



执行完后发现test2,test3数据都被删除，报告给客户端的受影响航的数量将只包含从test3中删除的数据。



with 子句和主查询兼容的执行

当在WITH中使用数据修改语句时，其他的指定的更新实际上是不可预知发生的，所有的语句都在相同的快照中执行，所以他们不能“看到”彼此对目标表的影响。这样减轻了实际行更新的不可预知的影响，并且意味着RETURNING数据是唯一在不同的WITH子语句和主查询交流变化的方式。

例 3

```
tutorial=# with t as (update test2 set name = 'guo se yi yi' returning * ) select * from test2;
 id | code | name  
----+------+-------
  1 |      | guo i
 11 |    1 | guo i
 12 |    2 | guo i
 13 |    3 | guo i
 14 |    4 | guo i
 15 |    5 | guo i
 16 |    6 | guo i
 17 |    7 | guo i
 18 |    8 | guo i
 19 |    9 | guo i
(10 rows)

tutorial=# select * from test2;
 id | code |     name     
----+------+--------------
  1 |      | guo se yi yi
 11 |    1 | guo se yi yi
 12 |    2 | guo se yi yi
 13 |    3 | guo se yi yi
 14 |    4 | guo se yi yi
 15 |    5 | guo se yi yi
 16 |    6 | guo se yi yi
 17 |    7 | guo se yi yi
 18 |    8 | guo se yi yi
 19 |    9 | guo se yi yi
(10 rows)

tutorial=# 

```

外层select将update动作之前返回过来

```
tutorial=# with t as (update test2 set name = 'guo se' returning * ) select * from t;
 id | code |  name  
----+------+--------
  1 |      | guo se
 11 |    1 | guo se
 12 |    2 | guo se
 13 |    3 | guo se
 14 |    4 | guo se
 15 |    5 | guo se
 16 |    6 | guo se
 17 |    7 | guo se
 18 |    8 | guo se
 19 |    9 | guo se
(10 rows)

tutorial=# 

```

外层SELECT返回了更新了的数据



数据修改语句中不允许递归的自引用。在某些情况下通过引用递归的`WITH` 输出，可能绕开这个限制 

例四

```
WITH RECURSIVE included_parts(sub_part, part) AS (
    SELECT sub_part, part FROM parts WHERE part = 'our_product'
  UNION ALL
    SELECT p.sub_part, p.part
    FROM included_parts pr, parts p
    WHERE p.part = pr.sub_part
  )
DELETE FROM parts
  WHERE part IN (SELECT part FROM 
```





​	











## 链接地址

https://www.postgresql.org/docs/9.1/static/queries-with.html