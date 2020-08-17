[TOC]

# postgresql 行转列



## 要求



​	postgresql数据库要求源码安装并且需要保留原来安装包

​	

## 编译安装



```
$ cd $install_path/contrib/tablefunc
$ make
$ make install
```



## 创建extension

```
osdba=# CREATE EXTENSION tablefunc; 
osdba=# \df *cross*
                               List of functions
 Schema |   Name    |      Result data type      | Argument data types |  Type  
--------+-----------+----------------------------+---------------------+--------
 public | crosstab  | SETOF record               | text                | normal
 public | crosstab  | SETOF record               | text, integer       | normal
 public | crosstab  | SETOF record               | text, text          | normal
 public | crosstab2 | SETOF tablefunc_crosstab_2 | text                | normal
 public | crosstab3 | SETOF tablefunc_crosstab_3 | text                | normal
 public | crosstab4 | SETOF tablefunc_crosstab_4 | text                | normal
(6 rows)
```

## sql 语句

```
select * from crosstab('select  subject_id,code,name from t_md_label_element   order by 1','select distinct code  from t_md_label_element order by 1') as t_md_label_element(subject_id text,"appSystem_right_dept" text,"businessCode_create_time"       text,
"dataObject_application_system_classify"      text,
"dataObject_business_classify"           text,
"dataObject_classify"           text,
"dataObject_dbType"          text,
"dataObject_industry_classify"     text,
"dataObject_level_1_element_classify"     text,
"dataObject_level_2_element_classify"      text,
"dataObject_level_3_element_classify"   text,
"dataObject_lifecycle_unit"      text,
"dataObject_need_comparison"      text,
"dataObject_obj_update_interval"         text,
"dataObject_resource_attribute"     text,
"dataObject_table_use_classify"   text,
"dataObject_update_type"       text,
"datasource_cluster_name"      text,
"datasource_connection_flag"      text,
"datasource_create_time"      text,
"datasource_domain_type"      text,
"datasource_link"     text,
"datasource_source_first_classify"     text,
"datasource_source_second_classify"   text) 


select * from crosstab('select  subject_id,code,name from t_md_label_element  order by 1','select distinct code  from t_md_label_element  where code not like ''dataObject%''order by 1') as t_md_label_element(subject_id text,"appSystem_right_dept" text,"businessCode_create_time"       text,
"datasource_cluster_name"      text,
"datasource_connection_flag"      text,
"datasource_create_time"      text,
"datasource_domain_type"      text,
"datasource_link"     text,
"datasource_source_first_classify"     text,
"datasource_source_second_classify"   text)
```



