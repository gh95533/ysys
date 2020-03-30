[TOC]

# pg 修改级联视图

**文档整理**

ysys

**日期**

2018-10-17

**标签**

pg,view,replace view



## 背景

​	今天晚上同事给我打电话，问了一个关于视图关联表的前提下，修改表的字段不会报错，想到当时修改视图时，如果修改字段长度，都会报错。那么这个问题是否可以解决呢？



## 视图修改



```
ysys=# create table test(id int,note varchar(20),info text);
CREATE TABLE
ysys=# create view view_test as select * from test;
CREATE VIEW
ysys=# insert into test values(1,'guohui','guokai');
INSERT 0 1
ysys=# select * from view_test;
 id |  note  |  info  
----+--------+--------
  1 | guohui | guokai
(1 row)

ysys=# alter table test alter note type varchar(1000);
ERROR:  cannot alter type of a column used by a view or rule
DETAIL:  rule _RETURN on view view_test depends on column "note"
```

**找到报错信息：rule _RETURN on view view_test depends on column "note"**

[pg_rule相关资料](../20180627/pg_rule.md)

- 视图语句创建

  ```
  create view vw_name as select column_name1,column_name2 ,... from table_name ...
  ```

  上面创建的语句等价于下面语句

- SELECT RULE 创建

  ```
  create table table_name_a as select *　from table_name where 1=2;
  CREATE RULE "_RETURN" AS ON SELECT TO table_name_a DO INSTEAD SELECT * FROM table_name;
  ```

- 备注

  1. SELECT 规则的后续操作只能是"INSTEAD SELECT"
  2. SELECT 规则名称只能是"_RETURN"

  每一个视图都是隐射成一张目标表(这张目标表在映射后创建出来了，后面我们需改原始表的字段长度，而映射表的没有变化，理论上来说是查询出来还是映射目标表，数据字段没有修改，这样是不可以的，所以会报错)

  现在如果想修改一张表的话，她所关联的视图可能都要修改，这样的话，是否可以通过类似命令

  ```
  ysys=# drop table test ;
  ERROR:  cannot drop table test because other objects depend on it
  DETAIL:  view view_test depends on table test
  HINT:  Use DROP ... CASCADE to drop the dependent objects too.
  ```

  删除表，那么依赖的所有相关的视图，函数都要删除，这个关系在数据库中哪里有体现呢？

## 解决方案

```
create or replace function get_dep_oids(oid) returns oid[] as $$
declare
  res oid[];
begin
  select array_agg(unnest::oid) into res from 
  (
    select unnest(regexp_matches(ev_action::text,':relid (\d+)', 'g')) from pg_rewrite where ev_class = $1 
  union 
    select unnest(regexp_matches(ev_action::text,':resorigtbl (\d+)','g')) from pg_rewrite where ev_class = $1 
  EXCEPT 
    select oid::text from pg_class where oid=$1 
  ) t;
return res;
end;
$$ language plpgsql strict;
```



```
create or replace function recursive_get_deps(IN tbl oid, OUT oid oid, OUT relkind "char", OUT nspname name, OUT relname name, OUT deps oid[], OUT ori_oid oid, OUT ori_relkind "char", OUT ori_nspname name, OUT ori_relname name ) returns setof record as
$$
declare
begin
return query 
with recursive a as (
  select * from (
    select t1.oid,t1.relkind,t2.nspname,t1.relname,get_dep_oids(t1.oid) deps,(select t1.oid from pg_class t1,pg_namespace t2 where t1.relnamespace=t2.oid and t1.oid=tbl) as ori_oid from pg_class t1, pg_namespace t2 where t1.relnamespace=t2.oid and t1.relkind in ('m','v')
  ) t where t.ori_oid = any(t.deps)
union 
  select * from (
    select t1.oid,t1.relkind,t2.nspname,t1.relname,get_dep_oids(t1.oid) deps, a.oid as ori_oid from pg_class t1,pg_namespace t2,a where t1.relnamespace=t2.oid and t1.relkind in ('m','v')
  ) t where t.ori_oid = any(t.deps)
)
select a.oid,a.relkind,a.nspname,a.relname,a.deps,a.ori_oid,b.relkind ori_relkind, c.nspname ori_nspname,b.relname ori_relname from a,pg_class b,pg_namespace c where a.ori_oid=b.oid and b.relnamespace=c.oid order by a.nspname,a.relkind,a.relname;
end;
$$ language plpgsql strict;
```



```
ysys=# select * from recursive_get_deps('test'::regclass);
  oid  | relkind | nspname |  relname   |  deps   | ori_oid | ori_relkind | ori_nspname | ori_relname 
-------+---------+---------+------------+---------+---------+-------------+-------------+-------------
 16467 | v       | public  | view_test  | {16453} |   16453 | r           | public      | test
 16471 | v       | public  | view_test2 | {16467} |   16467 | v           | public      | view_test
 16475 | v       | public  | view_test3 | {16453} |   16453 | r           | public      | test
 16479 | v       | public  | view_test4 | {16453} |   16453 | r           | public      | test
```

**谢谢德哥**



## 链接地址

https://github.com/digoal/blog/blob/master/201607/20160725_01.md