[TOC]

# VIRTUALBOX  无法启动虚拟机

​	由于某些原因换了一台新的电脑，之前虚拟机都是存储到移动硬盘，当重新安装virtualbox软件，发现虚拟机无法启动，后来设置bios的Virtualization Technology参数变为enable,后面大多数虚拟机就可以使用了，还有几个虚拟机复制重新注册还是不行。

## 操作流程



### 设置virtualization technology参数

​	进入bios界面设置virtualization technology参数变为enable



### 检查虚拟机是否正常

​	虚拟机大都是正常了，还有几个虚拟机不是正常，其中1台是window7,另外3台都是linux的服务器，它们都是有两个网卡，第一个网卡都是网络地址转换(NAT),第二个网卡是`仅主机(Host-Only)网络，'VirtualBox Host-Only Ethernet Adapter'`

​	尝试进行如下操作：

​	1、将网络设置两个网卡变为一个网卡而且是`仅主机(Host-Only)网络，'VirtualBox Host-Only Ethernet Adapter'`

​	2、重新启动虚拟机

​	当时除了windows虚拟机以外还有一台linux服务器无法启动，实在是没有想到解决方案，虚拟机不是很重要，就将这两台虚拟机给删除掉了。



## 链接地址

https://blog.csdn.net/tai532439904/article/details/78527889

