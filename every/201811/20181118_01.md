[TOC]

# oracle awr manager

**文档整理**

ysys

**日期**

2018-11-18

**标签**

oracle,oracle awr,oracle awr manager



## oracle  awr

### 查看当前环境下awr生成间隔和保留时间

```

>  select * from dba_hist_wr_control;
      DBID SNAP_INTERVAL                           RETENTION                               TOPNSQL
---------- --------------------------------------- --------------------------------------- ----------
2281919943 +00000 01:00:00.0                       +00008 00:00:00.0                       DEFAULT
```

​	其中snap_interval的间隔时间为一小时，retention保留时间为8天；

### 修改awr生成间隔和保留时间

```
> exec DBMS_WORKLOAD_REPOSITORY.MODIFY_SNAPSHOT_SETTINGS( retention => 8*24*60,interval => 30);
PL/SQL procedure successfully completed
>  select * from dba_hist_wr_control;
      DBID SNAP_INTERVAL                           RETENTION                               TOPNSQL
---------- --------------------------------------- --------------------------------------- ----------
2281919943 +00000 00:30:00.0                       +00008 00:00:00.0                       DEFAULT
```

​	在修改时，可能会报错ORA-13541: 系统移动窗口基线大小 (xxxxxx) 大于保留时间 (xxxxxx),这是因为默认的`moving_window_size`可能大于新修改的`retention`

​	在集群环境pl/sql下修改的发现节点一和节点二都已经修改成功。





## error

1、ORA-13541:系统移动窗口基线大小 (xxxxxx) 大于保留时间 (xxxxxx)

https://yq.aliyun.com/articles/28320





## 链接地址

https://www.linuxidc.com/Linux/2016-06/132170.htm