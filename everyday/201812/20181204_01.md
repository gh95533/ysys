[TOC]

# oracle asm copy datafile to local path

**document support**

ysys

**date**

2018-12-04

**label**

oracle asm,copy datafile to local path



## background

​	同事给我打电话，说是否可以从asm上拷贝数据文件到本地环境下，我觉得是可以的，后面又给我打电话说，不行，我就自己做了一个测试文档



## document

​	

### su - grid

​	切换到grid用户下，默认从节点一进入

```
# su - grid
$ export ORACLE_SID=+ASM1
$ asmcmd
```



### cp filename filepath

​	进入到datafile目录下，执行拷贝命令

```
> cp ts_cs_gh_data_1g /home/grid/ts_cs_gh_data_1g.dbf
copying +DATADG/GHDB/DATAFILE/ts_cs_gh_data_1g -> /home/grid/ts_cs_gh_data_1g.dbf
```

​	注意copy的文件夹必须可以被grid用户写入才可以





## 链接地址

https://www.linuxidc.com/Linux/2017-05/144203.htm

