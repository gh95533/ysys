
# postgresql 外部表

- 外部表的定义

不是自己本身数据库的数据，但是可以通过普通的sql语句查询(自己理解)

- 外部表不保存数据，数据还是原始数据库的

## sql/med

- sql/med是sql语言中管理外部数据的一个扩展标准

- med:management of  external data 

- 定义一个外部数据包装器和数据库连接类型管理外部数据

- postgresql9.1开始支持sql/med标准

- 注意使用源码安装postgreql

### FDW

- foreign data wrapper:外部数据包装器，相当于定义外部数据库驱动

在实际操作过程中，"create extension"命令会自动创建外部数据包装器

### SERVER

- 外部数据库服务器，相当于定义一个外部数据源，需要指定外部数据源的FDW

CREATE SERVER server_name [TYPE 'server_type'] [VERSION 'server_version']  FOREIGN DATA WRAPPER fdw_name [OPTIONS (option 'value'[,...])]);

server_name:外部server的名称

server_type:可选项，指定外部服务器的类型，是否使用此选项与具体的外部数据包装器有关，如果外部数据包装器没有此选项，则不需要定义此选项。

server_version:外部服务器的版本，也与具体的外部数据包装器有关

fdw_name:指定此外部服务器的外部数据包装器

OPTIONS：这些选项主要用于连接外部数据源，如连接外部数据源的ip地址，端口以及其他参

### USER MAPPING 

- 用户映射，主要把外部数据源的用户映射到本地用户，用于控制权限

CREATE USER MAPPING FOR user_name SERVER server_name [OPTIONS (option 'value'[,...])]);

user_name:本地管理员用户

server_name:指定一个服务名称，来源于create server

OPTIONS:通常定义映射的外部数据源上的实际用户名和密码

### Foreign table

- 外部表，把外部数据源映射成数据库中的一张外部表

CREATE FOREIGN TABLE table_name (column_name data_type) server server_name [OPTIONS (option 'value'[,...])]);

server_name:指定一个服务名称，来源于create server

OPTIONS：不同的数据库参数类型不一致

-----------

上面步骤就是创建外部表的流程，而为了更好的使用数据(外部表不保存数据)，建议和物化视图联系起来，通过物化视图来更新数据，已达到最新数据，物化视图也支持索引创建。

-----------

链接地址:

foreign_data_wrapper:https://pgxn.org/tag/foreign%20data%20wrapper/

file:

[pg_file_外部表](pg_file_外部表.md)

rdbms:

[pg_mysql_外部表](pg_mysql_外部表.md)

[pg_oracle_外部表](pg_oracle_外部表.md)

[pg_postgres_外部表](pg_postgres_外部表.md)

nosql:

[pg_mongo_外部表](pg_mongo_外部表.md)

[pg_redis_外部表](pg_redis_外部表.md)
