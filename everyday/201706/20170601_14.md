

[TOC]

# linux virtualbox copy 



当前一段时间经常需要将linux安装好几遍，特别浪费时间，通过VIRTUALbox复制将虚拟机复制多个，避免多次安装。可以有效的解决这个问题；只是这个样子需要重新设置ip以及主机名

## centos 6 系列

修改ip

1】修改/etc/udev/rules.d/70-persisten-net.rules的eth0,eth1等

2】修改ifcfg-eth0相关内容（做好在网络设置中修改）

3】重启网络配置 service network restart

![_](../img_src/56edbc6f85b242f3ae7d37dc5f95084b/clipboard.png)

修改hostname

1】修改/etc/sysconfig/network的主机名

2】在/etc/hosts 中添加 ip hostname

时间上不会太多，请注意/etc/udev/rules.d/70-persisten-net.rules的配置

上面是cnetos6系列

## centos 7

修改ip地址

cd  /etc/sysconfig/network-scripts/

ls -ls ifcfg*

vim  /etc/hostname

 

 