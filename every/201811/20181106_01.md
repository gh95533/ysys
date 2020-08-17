[TOC]

# linux ssh port

**文档整理**

ysys

**日期**

2018-11-06

**标签**

linux,ssh,port



## 背景

​	为了更好预防一下可能为了收集密码的操作，linux建议先从修改ssh端口开始预防。如果不知道端口，就更不容易了。



## 操作步骤



### centos6.x 修改配置参数

```
# cat /etc/redhat-release 
CentOS release 6.5 (Final)

# vim /etc/ssh/sshd_config 

Port 22 
Port 50000

# /etc/init.d/sshd restart

--关闭防火墙
# service iptables stop 
# chkconfig iptables off

```

​	第一种测试是重启一个xshell页面配置，输入端口号50000，查看是否可以连接

​	第二种测试是`ssh -p 50000 root@ip `

​	只要两种测试一种可以通过，那么可以将之前的`Port 22`这一行删除，防止因为直接修改端口后，可能因为端口号记错，或者修改出错。

​	**直接意味只能重启服务器才能管用，其实只要修改信息后执行重启服务就可以了**



### centos7.x 修改配置参数



```
# cat /etc/redhat-release 
CentOS Linux release 7.4.1708 (Core) 

# vim /etc/ssh/sshd_config 

Port 22 
Port 50000

# systemctl restart sshd.service 

# systemctl disable firewalld.service
# systemctl stop firewalld.service
```



​	请参考centos6.x测试





## 链接地址

https://www.cnblogs.com/silent2012/p/4682770.html

https://www.cnblogs.com/develop/p/4281338.html

https://blog.csdn.net/lovoo/article/details/78091383

https://blog.csdn.net/z69183787/article/details/76153247

https://blog.csdn.net/wangt5952/article/details/80811662