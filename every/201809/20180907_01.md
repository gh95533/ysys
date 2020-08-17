[TOC]

# selinux 基础

​	

![_](../img_src/000/2018-09-07_003121.png)

​	selinux安全增强LINUX是由NSA针对计算机基础架构安全开发的一个全新的linux安全策略机制。SElinux允许管理员更加灵活的定义安全策略。

​	selinux是一个内核级的安全机制，从2.6内核之后集成在内核当中

​	主流的linux发型版本都会集成Selinux机制，Centos/RHEL 默认开启SElinux

​	Selinux是内核机制，对Selinux的修改一般重新启动



![_](../img_src/000/2018-09-07_075456.png)

​	所有的安全机制都是对两样东西做出限制：进程和系统资源(文件，网络套接字，系统调用等)

​	Selinux针对这两种类型定义了两个基本概念:域和上下文

**域用来对进程进行限制**

**上下文用来对系统资源进行限制**

```
# ls -Z
-rw-------. root root system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Desktop
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Documents
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Downloads
-rw-r--r--. root root system_u:object_r:admin_home_t:s0 install.log
-rw-r--r--. root root system_u:object_r:admin_home_t:s0 install.log.syslog
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Music
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Pictures
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Public
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Templates
drwxr-xr-x. root root unconfined_u:object_r:admin_home_t:s0 Videos
```

```
# ps -Z
LABEL                             PID TTY          TIME CMD
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 1869 pts/0 00:00:00 bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 1890 pts/0 00:00:00 ps
```





![_](../img_src/000/2018-09-07_075935.png)

​	

![_](../img_src/000/2018-10-06_131446.png)

![_](../img_src/000/2018-10-06_131746.png)

![_](../img_src/000/2018-10-06_155411.png)



当从disable改为enforcing时

![_](../img_src/000/2018-09-07_131629.png)

