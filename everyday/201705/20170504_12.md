[TOC]

# kettle 配置oracle集群方式



kettle 在配置数据库链接时，只能配置一个hostname，或者可以这样说，如果是oracle数据库集群的话，要么是配置单机，单机就没有意义了，如何配置oracle集群？

第一种方案是配置jdbc.properties文件，将oracle集群数据库链接按照模板配置

第二种方案是直接在kettle数据库链接中配置