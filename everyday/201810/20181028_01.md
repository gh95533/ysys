[TOC]

# oracle rac 软链接

**文档整理**

ysys

**日期**

20181028

**标签**

oracle rac,机器空间不够用，软链接



## 背景

​	昨天同事给我打电话，说数据库所在的路径磁盘空间又快满了，并介绍一下这段时间数据库告警日志或者日志类文件增长较快，只要覆盖不及时，数据库所在的服务器空间所挂载的盘就可能就接近100%，其他的空间却并没有使用，想到使用软链接这种方式，并想测试一下使用软链接，是否可行。



## 概念介绍

### 软链接

​	软链接类似windows上面的快捷方式，可能是某个文件也可能是某个目录



## 操作步骤

### 停止数据库

```
$ srvctl stop database -d ghdb
# ./crs_stop -all
```

​	后期测试发现执行命令

```
# ./crs_start -all
```

​	无法成功，需要好好查看，可能需要重启服务器来可行。



### 拷贝当前很大的目录

​	在工作之前需要做这样一件事情，查看当地是那个文件夹所占的空间比较大，可以通过命令`# du -sh`查看当前文件夹下的大小空间。

​	此次使用的是测试目录，它的路径在`/u01/app/oracle/diag/rdbms/ghdb/ghdb1/`的trace文件夹

#### 检查当前拷贝文件夹的所属用户，组

```
# ls -ls
total 80
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 alert
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 cdump
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 hm
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 incident
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 incpkg
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 ir
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 lck
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 metadata
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 metadata_dgif
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 metadata_pv
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 stage
 4 drwxr-x---. 2 oracle asmadmin  4096 Dec 24  2017 sweep
32 drwxr-x---. 2 oracle asmadmin 28672 Oct 28 16:47 trace
```

​	发现需要备份的trace文件夹的用户组是oracle,asmadmin

#### 在磁盘较大的路径创建迁移目录并备份文件

```
# mkdir /opt/trace
# cp /u01/app/oracle/diag/rdbms/ghdb/ghdb1/trace/* /opt/trace/
# chown -R oracle:asmadmin /opt/trace
```

​	这里需要仔细查看两者的大小，记得数据库一定要关闭



#### 删除原始文件夹

```
# cd /u01/app/oracle/diag/rdbms/ghdb/ghdb1
# rm -rf trace
```

​	删除文件夹，一定要谨慎，要多次检查



#### 建立软链接

```
# ln -s /opt/trace/ /u01/app/oracle/diag/rdbms/ghdb/ghdb1/
```

​	

#### 对软链接重新授权

```
# chown -R oracle:asmadmin /u01/app/oracle/diag/rdbms/ghdb/ghdb1/trace
# chown -R oracle:asmadmin /u01/app/oracle/diag/rdbms/ghdb/ghdb1/trace/
```



**本地方案只是不得已而为之，最好安装数据库时，就好好考虑，而且不建议完全拷贝，可能某些lib是一些快捷方式。**



## 链接地址

https://www.cnblogs.com/kex1n/p/5193826.html