[TOC]

# postgresql 全文搜索

​	

## 文本搜索类型

​	postgresql提供了两种数据类型用于支持全文搜索，即通过自然语言documents的集合来找到那些匹配一个query的检索。tsvector类型产生一个文档，tsquery类型表示一个文本查询。

###  tsvector

​	tsvector的值是一个无重复值的lexemes排序列表，即一些同一个词的不同变种的标准化

```
tutorial=# select $$a fat cat sat on a mat and ate a fat rat$$::tsvector;
                      tsvector                      
----------------------------------------------------
 'a' 'and' 'ate' 'cat' 'fat' 'mat' 'on' 'rat' 'sat'
```

​	'fat'字段出现了2，'a'值出现了3次

​	

​	在这个例子中，使用'$$'字符串文本来区分子弹

```
tutorial=# select $$the lexeme ' ' contains spacess$$::tsvector;
                tsvector                 
-----------------------------------------
 ' ' 'contains' 'lexeme' 'spacess' 'the'
(1 row)
tutorial=# select $$the lexeme 'Joe''s' contains a quote$$::tsvector;
                    tsvector                    
------------------------------------------------
 'Joe''s' 'a' 'contains' 'lexeme' 'quote' 'the'
(1 row)
```



​	整型positions也可以放到词汇中

```
tutorial=# SELECT 'a:1 fat:2 cat:3 sat:4 on:5 a:6 mat:7 and:8 ate:9 a:10 fat:11 rat:12'::tsvector;
                                   tsvector                                    
-------------------------------------------------------------------------------
 'a':1,6,10 'and':8 'ate':9 'cat':3 'fat':2,11 'mat':7 'on':5 'rat':12 'sat':4
(1 row)


```

​	位置通常表示文档中的源头的位置，位置信息可用在proximity ranking ;位置值得范围从1到16383；相同词的重复位会被忽略掉

```
tutorial=# SELECT 'a:1 fat:2 cat:3 sat:4 on:5 a:6 mat:7 and:8 ate:200000 a:10 fat:16900 rat:12'::tsvector;
                                       tsvector                                       
--------------------------------------------------------------------------------------
 'a':1,6,10 'and':8 'ate':16383 'cat':3 'fat':2,16383 'mat':7 'on':5 'rat':12 'sat':4
(1 row)

```

​	

​	拥有位置的词汇甚至可以用一个权来标记，这个权可以是A,B,C或D,默认是D，因此输出中不会出现

```
tutorial=# SELECT 'a:1A fat:2B,4C cat:5D'::tsvector;
          tsvector          
----------------------------
 'a':1A 'cat':5 'fat':2B,4C
(1 row)


```

​	权可以用来反映文档结构，如：标记标题以与主体相区别。 全文检索排序函数可以为不同的权标记来分配不同的优先级。  

​	

​	tsvector类型不能自己标准化自己这一点是很重要的，它假设传递给它的单词对应程序来说都是恰到标准化的。

```
tutorial=# select 'The Fat Rats'::tsvector;
      tsvector      
--------------------
 'Fat' 'Rats' 'The'
(1 row)

tutorial=# select to_tsvector('The Fat Rats');
   to_tsvector   
-----------------
 'fat':2 'rat':3
(1 row)

tutorial=# select to_tsvector('english','The Fat Rats');
   to_tsvector   
-----------------
 'fat':2 'rat':3
(1 row)

```



### tsquery

​	`tsquery`存储用于检索的词汇，并且使用布尔操作符 `&`(AND)，`|`(OR)和`!`(NOT) 来组合它们 

​	括号用来强调操作符的分组 

```
SELECT 'fat & rat'::tsquery;
    tsquery    
---------------
 'fat' & 'rat'

SELECT 'fat & (rat | cat)'::tsquery;
          tsquery          
---------------------------
 'fat' & ( 'rat' | 'cat' )

SELECT 'fat & rat & ! cat'::tsquery;
        tsquery         
------------------------
 'fat' & 'rat' & !'cat'
```

​	在没有括号的情况下，!(NOT)结合的最紧密，而&(AND)结合的比|(OR)紧密

​	

​	tsquery中的词汇可以被一个或者多个权字母来标记，这些权字母用来限制它们只能带有匹配权的tsvector词汇进行匹配

```
tutorial=# select 'fat:ab & cat '::tsquery;
     tsquery      
------------------
 'fat':AB & 'cat'
(1 row)
```

​	

​	tsquery中的词汇可以用*进行标记来指定前缀匹配

```
tutorial=# select 'super:*'::tsquery;
  tsquery  
-----------
 'super':*

```

​	这个查询可以匹配tsvector中以"super"开始的任意单词。请注意，前缀首先被文本搜索配置处理，这也就意味着下面的结果为真： 

```

tutorial=# SELECT to_tsvector( 'postgraduate' );
  to_tsvector  
---------------
 'postgradu':1
(1 row)
tutorial=# select to_tsquery('postgres:*');
 to_tsquery 
------------
 'postgr':*
tutorial=# SELECT to_tsvector( 'postgraduate' ) @@ to_tsquery( 'postgres:*' );
 ?column? 
----------
 t
(1 row)

```

​	

​	带上语言配置

```
select to_tsquery('english','Fat:ab & Cats');
    to_tsquery    
------------------
 'fat':AB & 'cat'
(1 row)


```



​	tsquery对于操作符@@的使用

```
tutorial=# select to_tsvector('spanish', $$Hello world, I'm digoal.$$);
                to_tsvector                
-------------------------------------------
 'digoal':5 'hell':1 'i':3 'm':4 'world':2
(1 row)

tutorial=# select to_tsvector('spanish', $$Hello world, I'm digoal.$$) @@ 'i:b'::tsquery;
 ?column? 
----------
 f
(1 row)

tutorial=# select to_tsvector('spanish', $$Hello world, I'm digoal.$$) @@ 'i:d'::tsquery;
 ?column? 
----------
 t
(1 row)

tutorial=# select to_tsvector('spanish', $$Hello world, I'm digoal.$$) @@ 'i:*'::tsquery;
 ?column? 
----------
 t
(1 row)

tutorial=# select to_tsvector('spanish', $$Hello world, I'm digoal.$$) @@ 'i:d*'::tsquery;
 ?column? 
----------
 t
(1 row)

tutorial=# select to_tsvector('spanish', $$Hello world, I'm digoal.$$) @@ 'i:a*'::tsquery;
 ?column? 
----------
 f
(1 row)

```





### 文本检索函数和操作符





**文本检索操作符**

| 操作符 | 描述                        | 示例                                                         | 结果                        |
| ------ | --------------------------- | ------------------------------------------------------------ | --------------------------- |
| `@@`   | `tsvector` 匹配 `tsquery` ? | `to_tsvector('fat cats ate rats') @@ to_tsquery('cat & rat')` | `t`                         |
| `@@@`  | 弃用的`@@`的同义词          | `to_tsvector('fat cats ate rats') @@@ to_tsquery('cat & rat')` | `t`                         |
| `||`   | 连接`tsvector`s             | `'a:1 b:2'::tsvector || 'c:1 d:2 b:3'::tsvector`             | `'a':1 'b':2,5 'c':3 'd':4` |
| `&&`   | `tsquery`与                 | `'fat | rat'::tsquery && 'cat'::tsquery`                     | `( 'fat' | 'rat' ) & 'cat'` |
| `||`   | `tsquery`或                 | `'fat | rat'::tsquery || 'cat'::tsquery`                     | `( 'fat' | 'rat' ) | 'cat'` |
| `!!`   | `tsquery`非                 | `!! 'cat'::tsquery`                                          | `!'cat'`                    |
| `@>`   | `tsquery` 包含另一个?       | `'cat'::tsquery @> 'cat & rat'::tsquery`                     | `f`                         |
| `<@`   | `tsquery` 包含于 ?          | `'cat'::tsquery <@ 'cat & rat'::tsquery`                     | `t`                         |





**文本检索函数**

| 函数                                                         | 返回类型    | 描述                               | 示例                                                         | 结果                            |
| ------------------------------------------------------------ | ----------- | ---------------------------------- | ------------------------------------------------------------ | ------------------------------- |
| `get_current_ts_config()`                                    | `regconfig` | 获取文本检索的默认配置             | `get_current_ts_config()`                                    | `english`                       |
| `length(tsvector)`                                           | `integer`   | `tsvector`的单词数                 | `length('fat:2,4 cat:3 rat:5A'::tsvector)`                   | `3`                             |
| `numnode(tsquery)`                                           | `integer`   | `tsquery`中的单词加上操作符的数量  | ` numnode('(fat & rat) | cat'::tsquery)`                     | `5`                             |
| `plainto_tsquery([ *config* regconfig , ] *query* text)`     | `tsquery`   | 产生`tsquery`忽略标点              | `plainto_tsquery('english', 'The Fat Rats')`                 | `'fat' & 'rat'`                 |
| `querytree(*query* tsquery)`                                 | `text`      | 获取`tsquery`可索引的部分          | `querytree('foo & ! bar'::tsquery)`                          | `'foo'`                         |
| `setweight(tsvector, "char")`                                | `tsvector`  | 给`tsvector`的每个元素赋予权值     | `setweight('fat:2,4 cat:3 rat:5B'::tsvector, 'A')`           | `'cat':3A 'fat':2A,4A 'rat':5A` |
| `strip(tsvector)`                                            | `tsvector`  | 删除`tsvector`中的位置和权值       | `strip('fat:2,4 cat:3 rat:5A'::tsvector)`                    | `'cat' 'fat' 'rat'`             |
| `to_tsquery([ *config* regconfig , ] *query* text)`          | `tsquery`   | 标准化单词并转换为`tsquery`        | `to_tsquery('english', 'The & Fat & Rats')`                  | `'fat' & 'rat'`                 |
| `to_tsvector([ *config* regconfig , ] *document* text)`      | `tsvector`  | 减少文档中的文本到 `tsvector`      | `to_tsvector('english', 'The Fat Rats')`                     | `'fat':2 'rat':3`               |
| `ts_headline([ *config* regconfig, ] *document* text, *query* tsquery [, *options* text ])` | `text`      | 显示一个查询的匹配项               | `ts_headline('x y z', 'z'::tsquery)`                         | `x y <b>z</b>`                  |
| `ts_rank([ *weights* float4[], ] *vector* tsvector, *query* tsquery [, *normalization* integer ])` | `float4`    | 文档查询排名                       | `ts_rank(textsearch, query)`                                 | `0.818`                         |
| `ts_rank_cd([ *weights* float4[], ] *vector* tsvector, *query* tsquery [, *normalization* integer ])` | `float4`    | 排序文件查询使用覆盖密度           | `ts_rank_cd('{0.1, 0.2, 0.4, 1.0}', textsearch, query)`      | `2.01317`                       |
| `ts_rewrite(*query* tsquery, *target* tsquery, *substitute* tsquery)` | `tsquery`   | 替换带有查询的替代目标             | `ts_rewrite('a & b'::tsquery, 'a'::tsquery, 'foo|bar'::tsquery)` | `'b' & ( 'foo' | 'bar' )`       |
| `ts_rewrite(*query* tsquery, *select* text)`                 | `tsquery`   | 从一条`SELECT`命令的替代目标做替换 | `SELECT ts_rewrite('a & b'::tsquery, 'SELECT t,s FROM aliases')` | `'b' & ( 'foo' | 'bar' )`       |
| `tsvector_update_trigger()`                                  | `trigger`   | 自动更新`tsvector`列的触发器函数   | `CREATE TRIGGER ... tsvector_update_trigger(tsvcol, 'pg_catalog.swedish', title, body)` | ``                              |
| `tsvector_update_trigger_column()`                           | `trigger`   | 自动更新`tsvector`列的触发器函数   | `CREATE TRIGGER ... tsvector_update_trigger_column(tsvcol, configcol, title, body)` | ``                              |





**文本检索调试函数**

| 函数                                                         | 返回类型       | 描述                       | 示例                                               | 结果                                                         |
| ------------------------------------------------------------ | -------------- | -------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| `ts_debug([ *config* regconfig, ] *document* text, OUT *alias* text, OUT *description* text, OUT *token* text, OUT *dictionaries* regdictionary[], OUT *dictionary* regdictionary, OUT *lexemes* text[])` | `setof record` | 测试一个配置               | `ts_debug('english', 'The Brightest supernovaes')` | `(asciiword,"Word, all ASCII",The,{english_stem},english_stem,{}) ...` |
| `ts_lexize(*dict* regdictionary, *token* text)`              | `text[]`       | 测试一个数据字典           | `ts_lexize('english_stem', 'stars')`               | `{star}`                                                     |
| `ts_parse(*parser_name* text, *document* text, OUT *tokid* integer, OUT *token* text)` | `setof record` | 测试一个解析               | `ts_parse('default', 'foo - bar')`                 | `(1,foo) ...`                                                |
| `ts_parse(*parser_oid* oid, *document* text, OUT *tokid* integer, OUT *token* text)` | `setof record` | 测试一个解析               | `ts_parse(3722, 'foo - bar')`                      | `(1,foo) ...`                                                |
| `ts_token_type(*parser_name* text, OUT *tokid* integer, OUT *alias* text, OUT *description* text)` | `setof record` | 由解析器获取分词类型的定义 | `ts_token_type('default')`                         | `(1,asciiword,"Word, all ASCII") ...`                        |
| `ts_token_type(*parser_oid* oid, OUT *tokid* integer, OUT *alias* text, OUT *description* text)` | `setof record` | 由解析器获取分词类型的定义 | `ts_token_type(3722)`                              | `(1,asciiword,"Word, all ASCII") ...`                        |
| `ts_stat(*sqlquery* text, [ *weights* text, ] OUT *word* text, OUT *ndoc* integer, OUT *nentry* integer)` | `setof record` | 获取`tsvector`列的统计数据 | `ts_stat('SELECT vector from apod')`               | `(foo,10,15) ...`                                            |



## 全文搜索

### 全文检索

​	和全文检索有关数据类型就是tsvector和tsquery

​	tsvector--文本在经过全文检索标准化处理后得到类型，处理后的文本包括(分词去重后排序,分词的位置,分词的权重结构(一共有4个权重ABCD,D默认不显示))

​	tsquery--需要检索的分词组合，组合类型包括’ &,|,!‘,同时还支持分词的权重，分词的前导匹配



​	使用to_tsvector可以指定不同的语言配置，把文本根据指定的语言配置进行分词

​	例如，使用西班牙语言和英语得到tsvector值是不同的

```
tutorial=# select to_tsvector('english', $$Hello world, I'm guohui.$$);
             to_tsvector              
--------------------------------------
 'guohui':5 'hello':1 'm':4 'world':2
(1 row)

tutorial=# select to_tsvector('spanish', $$Hello world, I'm guohui.$$);
                to_tsvector                
-------------------------------------------
 'guohui':5 'hell':1 'i':3 'm':4 'world':2
(1 row)

tutorial=# select to_tsvector('french', $$Hello world, I'm guohui.$$);
             to_tsvector              
--------------------------------------
 'guohui':5 'hello':1 'i':3 'world':2
(1 row)

```



​	查看系统中已经安装的全文检索配置

```
tutorial=# \dF *
               List of text search configurations
   Schema   |    Name    |              Description              
------------+------------+---------------------------------------
 pg_catalog | danish     | configuration for danish language
 pg_catalog | dutch      | configuration for dutch language
 pg_catalog | english    | configuration for english language
 pg_catalog | finnish    | configuration for finnish language
 pg_catalog | french     | configuration for french language
 pg_catalog | german     | configuration for german language
 pg_catalog | hungarian  | configuration for hungarian language
 pg_catalog | italian    | configuration for italian language
 pg_catalog | norwegian  | configuration for norwegian language
 pg_catalog | portuguese | configuration for portuguese language
 pg_catalog | romanian   | configuration for romanian language
 pg_catalog | russian    | configuration for russian language
 pg_catalog | simple     | simple configuration
 pg_catalog | spanish    | configuration for spanish language
 pg_catalog | swedish    | configuration for swedish language
 pg_catalog | turkish    | configuration for turkish language
 public     | testzhcfg  | 
(17 rows)


```

​	testzhcfg是中文分词	



​	全文检索的索引使用

```
tutorial=# create table test1(id int ,info tsvector,crt_time timestamp(0));
CREATE TABLE
tutorial=# insert into test1 values(1,$$Hello world,I'm guohui.$$,now());
INSERT 0 1
tutorial=# select * from test1;
 id |              info              |      crt_time       
----+--------------------------------+---------------------
  1 | 'Hello' 'guohui.' 'world,I''m' | 2018-08-08 08:03:09
(1 row)

tutorial=# create index idx_test1_info on test1 using gin(info);
CREATE INDEX
tutorial=# select * from test1 where info @@ 'guohui.'::tsquery;
 id |              info              |      crt_time       
----+--------------------------------+---------------------
  1 | 'Hello' 'guohui.' 'world,I''m' | 2018-08-08 08:03:09
(1 row)

tutorial=# select * from test1 where info @@ 'guohui'::tsquery;
 id | info | crt_time 
----+------+----------
(0 rows)
```

​	全文检索的索引使用(tsvector支持的索引策略gin,gist,btree)

​	gin索引策略，可用于tsvector包含tsquery的查询匹配

```
tutorial=# explain analyze select * from test1 where info @@ 'guohui'::tsquery;
                                           QUERY PLAN                                           
------------------------------------------------------------------------------------------------
 Seq Scan on test1  (cost=0.00..1.01 rows=1 width=44) (actual time=0.005..0.005 rows=0 loops=1)
   Filter: (info @@ '''guohui'''::tsquery)
   Rows Removed by Filter: 1
 Planning time: 0.043 ms
 Execution time: 0.016 ms
(5 rows)

tutorial=# set enable_seqscan = off;
SET
tutorial=# explain analyze select * from test1 where info @@ 'guohui'::tsquery;
                                                      QUERY PLAN                                       
                
-------------------------------------------------------------------------------------------------------
----------------
 Bitmap Heap Scan on test1  (cost=8.00..12.01 rows=1 width=44) (actual time=0.009..0.009 rows=0 loops=1
)
   Recheck Cond: (info @@ '''guohui'''::tsquery)
   ->  Bitmap Index Scan on idx_test1_info  (cost=0.00..8.00 rows=1 width=0) (actual time=0.008..0.008 
rows=0 loops=1)
         Index Cond: (info @@ '''guohui'''::tsquery)
 Planning time: 0.068 ms
 Execution time: 0.056 ms
(6 rows)


```



​	GIST索引策略，可用于包含匹配

```
tutorial=# drop index idx_test1_info;
DROP INDEX
tutorial=# create index idx_test1_info on test1 using gist(info);
CREATE INDEX
tutorial=# explain analyze select * from test1 where info @@ 'guohui'::tsquery;
                                                      QUERY PLAN                                       
                
-------------------------------------------------------------------------------------------------------
----------------
 Index Scan using idx_test1_info on test1  (cost=0.12..8.14 rows=1 width=44) (actual time=0.007..0.007 
rows=0 loops=1)
   Index Cond: (info @@ '''guohui'''::tsquery)
 Planning time: 0.138 ms
 Execution time: 0.076 ms
(4 rows)

tutorial=# 

```



###  中文检索

#### 创建中文检索

https://github.com/digoal/blog/blob/master/201206/20120621_01.md

#### 创建中文检索二

```
# wget -q -O - http://www.xunsearch.com/scws/down/scws-1.2.2.tar.bz2 | tar xjf
# cd scws-1.2.2/
# ./configure --prefix=/opt/scws-1.2.2
# make
# make install
```



```
# cd zhparser
# export PATH=/usr/local/pgsql/bin:$PATH
# SCWS_HOME=/opt/scws-1.2.2 make
# make install

```



```
# su - osdba
[osdba@mysql45 ~]$ psql -d tutorial
psql (9.4.1)
Type "help" for help.

tutorial=# create extension zhparser;  
CREATE EXTENSION  
  
tutorial=# select * from pg_ts_parser ;  
 prsname  | prsnamespace |  prsstart   |    prstoken     |  prsend   |  prsheadline  |  prslextype     
----------+--------------+-------------+-----------------+-----------+---------------+---------------  
 default  |           11 | prsd_start  | prsd_nexttoken  | prsd_end  | prsd_headline | prsd_lextype  
 zhparser |        25956 | zhprs_start | zhprs_getlexeme | zhprs_end | prsd_headline | zhprs_lextype  
(2 rows)  
  
tutorial=# CREATE TEXT SEARCH CONFIGURATION testzhcfg (PARSER = zhparser);  
CREATE TEXT SEARCH CONFIGURATION  
  
tutorial=# select * from pg_ts_config where cfgname='testzhcfg';  
  cfgname  | cfgnamespace | cfgowner | cfgparser   
-----------+--------------+----------+-----------  
 testzhcfg |        25956 |       10 |     26134  
 
 
 
tutorial=# ALTER TEXT SEARCH CONFIGURATION testzhcfg ADD MAPPING FOR n,v,a,i,e,l WITH simple;  
ALTER TEXT SEARCH CONFIGURATION  
tutorial=# select * from pg_ts_config_map where mapcfg=(select oid from pg_ts_config where cfgname='testzhcfg');  
 mapcfg | maptokentype | mapseqno | mapdict   
--------+--------------+----------+---------  
  26135 |           97 |        1 |    3765  
  26135 |          101 |        1 |    3765  
  26135 |          105 |        1 |    3765  
  26135 |          108 |        1 |    3765  
  26135 |          110 |        1 |    3765  
  26135 |          118 |        1 |    3765  
(6 rows)  


SELECT * FROM ts_parse('zhparser', 'hello world! 2010年保障房建设在全国范围内获全面启动，从中央到地方纷纷加大 了保障房的建设和投入力度 。2011年，保障房进入了更大规模的建设阶段。住房城乡建设部党组书记、部长姜伟新去年底在全国住房城乡建设工作会议上表示，要继续推进保障性安居工程建设。');


SELECT to_tsvector('testzhcfg','“今年保障房新开工数量虽然有所下调，但实际的年度在建规模以及竣工规模会超以往年份，相对应的对资金的需求也会创历>史纪录。”陈国强说。在他看来，与2011年相比，2012年的保障房建设在资金配套上的压力将更为严峻。');


SELECT to_tsquery('testzhcfg', '保障房资金压力');
```





## 链接地址



http://www.postgres.cn/docs/9.4/textsearch-intro.html

https://github.com/digoal/blog/blob/master/201403/20140324_01.md

https://github.com/digoal/blog/blob/master/201206/20120621_01.md

https://pgxn.org/dist/zhparser

