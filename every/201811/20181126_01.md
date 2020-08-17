[TOC]

# centos 7  graphical interface input root correct password not login 

**文档**

ysys

**日期**

2018-11-27

**标签**

centos 7,root correct password,not login



## 背景

​	某一天，几台安装greenplum4.3的centos7服务器所在的机房异常断电，开机后，进入图形化界面输入root密码，图形变了以下后，重新进入到选择用户登录的图形上，gpadmin用户也是如此，而另外一个用户admin确实可以输入密码就可以进入到图形化界面。



## 处理

​	猜测可能与安装gp软件时修改了某些参数或者添加了某些参数导致的，现在就要重新为其复制一个新的环境来说明这种情况是否可以重现。







