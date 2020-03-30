[TOC]

# linux centos 6.5 install docker

**document support**

ysys

**date**

2018-12-11

**label**

linux,centos 6.5,docker install



## install

​	

```
# cd /etc/yum.repos.d/
# wget http://www.hop5.in/yum/el6/hop5.repo
```



```
# yum install kernel-ml-aufs kernel-ml-aufs-devel
```



```
# vim /etc/grub.conf

default=0   ## modify 

title CentOS (3.10.5-3.el6.x86_64) ## modify
```

​	重启

```
# reboot
```

​	查看内核是否修改

```
# uname -r
3.10.5-3.el6.x86_64
```

​	查看是否支持aufx

```
# grep aufs /proc/filesystems 
nodev	aufs
```

​	修改selinux配置

```
# vim /etc/selinux/config

SELINUX=disbaled
```

​	下载rpm

链接：https://pan.baidu.com/s/1a7sFEJQFlqqW3sm2gF8x4Q 
提取码：xim5 

```
# rpm -ivh libcgroup-0.40.rc1-6.el6_5.1.x86_64.rpm
```

​	查看docker版本

```
# docker version
Client version: 1.7.1
Client API version: 1.19
Go version (client): go1.4.2
Git commit (client): 786b29d/1.7.1
OS/Arch (client): linux/amd64
Server version: 1.7.1
Server API version: 1.19
Go version (server): go1.4.2
Git commit (server): 786b29d/1.7.1
OS/Arch (server): linux/amd64
```

​	查看docker日志

```
# cat /var/log/docker
```









## 链接地址

https://www.linuxidc.com/Linux/2015-01/111091.htm

https://blog.csdn.net/qq_42679773/article/details/81066053

http://www.cnblogs.com/zhengah/p/4930292.html

https://www.linuxidc.com/Linux/2013-12/93740.htm

https://www.cnblogs.com/zhenyuyaodidiao/p/5464422.html

https://cloud.tencent.com/developer/article/1335014

http://www.bubuko.com/infodetail-1965086.html

http://blog.51cto.com/guanhaizhan/2050245

https://blog.csdn.net/u012066426/article/details/52188719