[TOC]

# linux redhat7.4 install vsftp

**文档整理**

ysys

**日期**

2018-10-15

**标签**

linux,redhat,redhat7.4,vsftp,vsftp install



## 背景

​	在生产环境中搭建vsftp服务出现一系列问题，之前整理centos6.5部署vsftp，一是较老，二是比较繁琐，三是写的不明白，为此重新整理一份文档。



## 安装

### yum配置

参考[yum 配置](../20170601/linux_yum_配置.md)

### 安装vsftp包

```
# yum install vsftpd
...
Running transaction
  Installing : vsftpd-3.0.2-22.el7.x86_64                                   1/1 
ysys/productid                                           | 1.6 kB     00:00     
  Verifying  : vsftpd-3.0.2-22.el7.x86_64                                   1/1 

Installed:
  vsftpd.x86_64 0:3.0.2-22.el7                                                  

Complete!
```



### 配置



### 备份文件

```
# cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf2
```



### 重新设置参数文件

```
# mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf_bak
# grep -v "#" /etc/vsftpd/vsftpd.conf_bak > /etc/vsftpd/vsftpd.conf
```





### 查看参数文件

```
# vim /etc/vsftpd/vsftpd.conf

anonymous_enable=YES
local_enable=YES
allow_writeable_chroot=yes
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```



### 配置密码文件

```
# vim  /etc/vsftpd/vuser.list 
gh123
gh123
gh1234
gh1234
gh2345
gh2345
```

### 配置数据库文件

```
# db_load -T -t hash -f /etc/vsftpd/vuser.list /etc/vsftpd/vuser.db
# file /etc/vsftpd/vuser.db
# rm -rf /etc/vsftpd/vuser.list
```

### 创建用户

```
# useradd -d /home/data -s /sbin/nologin virtual
# ls -ld /home/data
# chmod -Rf 755 /home/data
```

### 修改PAM文件

```
# vim /etc/pam.d/vsftpd.vu
auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser
```



### 重新修改vsftpd.conf

```
# vim /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
guest_enable=YES
guest_username=virtual
allow_writeable_chroot=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES

pam_service_name=vsftpd.vu
userlist_enable=YES
tcp_wrappers=YES
user_config_dir=/etc/vsftpd/vuser_dir
anon_umask=022
pasv_max_port=21010
pasv_min_port=21001
```







### 为虚拟用户设置不同的权限

```
#  mkdir /etc/vsftpd/vuser_dir/
#  vim /etc/vsftpd/vuser_dir/gh123
#  vim /etc/vsftpd/vuser_dir/gh1234
#  vim /etc/vsftpd/vuser_dir/gh2345
```

```
# cat /etc/vsftpd/vuser_dir/gh123
guest_enable=YES
guest_username=virtual
local_root=/home/data/gh123
write_enable=yes
pam_service_name=vsftpd.vu
anon_umask=022
anon_world_readable_only=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
```

### 创建目录并授权

```
# mkdir /home/data/gh123
# mkdir /home/data/gh1234
# mkdir /home/data/gh2345
# chown -R virtual:virtual /home/data
# cd /home/data
# ls -ls /home/data
```



### 关闭防火墙和selinux安全策略

```
# systemctl stop firewalld.service
# systemctl disable firewalld.service
# vim /etc/selinux/config
disabled
```





## 错误

1、systemctl start vsftpd.service 服务无法启动

```
# systemctl start vsftpd.service
Job for vsftpd.service failed because the control process exited with error code. See "systemctl status vsftpd.service" and "journalctl -xe" for details.
```

http://bbs.51cto.com/thread-1156344-1.html



## 链接地址

https://www.cnblogs.com/lightnear/p/8952460.html

https://linuxconfig.org/how-to-setup-vsftpd-ftp-file-server-on-redhat-7-linux

http://www.firewall.cx/linux-knowledgebase-tutorials/system-and-network-services/875-linux-vsftpd-setup-configure.html

https://www.tecmint.com/install-ftp-server-in-centos-7/

https://www.itzgeek.com/how-tos/linux/centos-how-tos/install-and-configure-vsftpd-on-centos-7-rhel-7.html

https://blog.csdn.net/qq_36783142/article/details/73692596https://www.unixmen.com/install-configure-ftp-server-centos-7/

https://www.thegeekdiary.com/centos-rhel-7-how-to-install-and-configure-ftp-server-vsftpd/

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/s1-ftp

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/s2-ftp-servers-vsftpd

