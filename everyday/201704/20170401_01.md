[TOC]

# install oracle 12c single in linux 

​	当前测试环境都是在自己电脑的虚拟机上，可能与真实环境不是很一致，这点要先明确出来

1、修改主机名，网络

参考链接：[linux  virtualbox copy.note](note://B0EAA5B5DCCF4FF598CAD462978BAC01)

2、关闭防火墙和selinux

参考链接:[ install oracle 11g single in linux.note](note://ADBDD57535C94C7D88740ECD7CD93173)

3、添加yum并安装rpm包

yum安装:[linux yum configuration.note](note://1E3BD86AF6AE450A8369A011F03FB0EB)

vim pkg_run.sh

chmod 777 pkg_run.sh

4、修改内核相关配置

[root@localhost /]# vi /etc/sysctl.conf

[root@localhost /]# /sbin/sysctl -p

[root@localhost /]# vi /etc/security/limits.conf

5、添加用户并修改用户配置

6、在oracle修改参数

su - oracle

vim .bash_profile

source .bash_profile

7、图形化安装

![_](../img_src/06a505462ba34c3ab60ba43988dbff5b/clipboard.png)

![_](../img_src/48486dbafcd5428bb03185da8d815a68/clipboard.png)

![_](../img_src/86d05b91243b4d0fba9c6a3690b048f3/clipboard.png)

![_](../img_src/c0c7fd552b1d4f99b7e435c8c0d243bd/clipboard.png)

![_](../img_src/4d0e633b6f614716be5e87a5d8f6eddc/clipboard.png)

![_](../img_src/1ed0978168214e759ea9e75ac980d550/clipboard.png)

![_](../img_src/158caefabb8e43a1ba5c01065e4c0d0e/clipboard.png)

![_](../img_src/358c3f6adde947ff8faeac8f8f22587e/clipboard.png)

参考文档：

<https://oracle-base.com/articles/12c/oracle-db-12cr2-installation-on-oracle-linux-6-and-7>