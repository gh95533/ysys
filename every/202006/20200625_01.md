[TOC]

# kettle  support greenplum gpload

**文档整理**

ysys

**日期**

20181030(20170101)

**标签**

kettle,greenplum gpload



## 背景

​	之前在TT,XJ做方案时，遇到了Oracle到Gp数据库的迁移，当时就使用了gpload插件来完成数据迁移，今天从新整理一下文档，以便可以更好的使用;kettle的表输出，插入或者更新控件对于gp数据库来说，如果数据量都在万以下，不考虑时间，还可以接受，如果万级以上，时间要求比较及时，那么只能使用greenplum loader控件来完成



## kettle greenplum loader

操作环境：centos6.5

软件：greenplum-loader（installed）;kettle(5.4)

### 步骤

打开kettle界面

新建一个转换

查找到BULK LOADING下的GREENPLUM LOADER控件

如下图

![img](../img_src/5a8f1c3a3bd4424f9ddf03c68aeb3f4b/clipboard.png)

greenplum bulk loader 与 greenplum loader 区别还有联系，暂时还是不懂，所以现在使用greenplum load

介绍：本次转换将从mysql数据库中获取数据并传入到greenplumdatabase

大致图形如下

![img](../img_src/e741f78cbe28460980a0a141de4102b1/clipboard.png)

查看greenplum load配置

![img](../img_src/428796e266174b9186fcacfa22786b28/clipboard.png)

分为

字段；本地主机名；gp的配置参数

字段不需要过多解释

本地主机名

![img](../img_src/5abf94f00132440ba026cd85c661b8b8/clipboard.png)

端口号最好填写3333不是关键端口

主机名：有的时候无法解析，建议直接使用ip地址

gp参数配置

![img](../img_src/0dfd4f8604944d7fbab73cb0980baceb/clipboard.png)

path to the gpload:本地部署gpload的路径

/usr/local/greenplum-loaders-4.3.8.1-build-1/bin/gpload

control file：此文件可以只创建文件名

data file:此文件可以只创建文件名

Encoding：选择UTF8

现在开始比较表插入，更新或者插入以及使用greenplum loader的效率

查询mysql下的test001数据量为5242880；

执行mysql到gp通过greenplum loader方式，总共用时2分钟；

使用mysql到gp直接使用表输出模式

![img](../img_src/c93e2f1d44dd48429bc2d3f015122b10/clipboard.png)

时间比较长，3min跑完20万，其实在生产库中，由于字段较多，每秒钟的速度只是到100条左右；

20min后截图

![img](../img_src/21382f4a9484410a86e9f5270d1dbfac/clipboard.png)

40min后截图

![img](../img_src/3f84f6f57a2a4227a8e20a25661a5ec7/clipboard.png)

放弃了最终时间，不过时间确实是超长的

比较插入更新；

这次选择1000条数据左右

greenplum loader用时1min

kettle自带的插入更新

好像5.4没有插入更新这个控件，其实在生产库中，使用kettle插入更新gp数据库超级慢，后来由负责gp的人使用gpload写入加载数据，考虑到脚本不可确定性；使用kettle通过greenplum load控件更加易于管理



**在实际测试中发现，如果后期使用crontab来调用方案调用的，发现环境变量配置后很多环境都已经报错了，建议使用如下命令,其实不建议直接在环境变量配置相关环境**

```
# cat gp_insert.sh
source /usr/local/greenplum-loaders-4.3.8.1-build-1/greenplum_loaders_path.sh
export PYTHONPATH=/usr/local/lib
cd /../data-integration/
./kitchen.sh /res:{repname} /user:admin /pass:admin /job:{jobname}
```





方案附件

[MYSQL_TRAN_GREENPLUM](../img_src/133fe193d8f14553b39f809896f2ed33/MYSQL_TRAN_GREENPLUM.ktr)

====================================================================

错误

1.1、Driver class 'org.gjt.mm.mysql.Driver' could not be found, make sure the 'MySQL' driver (jar file) is installed.

org.gjt.mm.mysql.Driver

![img](../img_src/34845b59273c4d5fa81a90c28a8ef150/clipboard.png)

kettle没有自带mysql驱动，需要下载后安装就可以了

​       2.1、启动greenplum loader报错

主要问题是找不到gpload.py，查看环境配置

添加

source /usr/local/greenplum-loaders-4.3.8.1-build-1/greenplum_loaders_path.sh

export PYTHONPATH=/usr/local/lib

记得重新打开kettle软件