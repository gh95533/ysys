[TOC]

# LINUX YUM



## yum 来源

​	rpm是全新的封装形式，取代了很多通过源代码发布软件，但是rpm无法解决依赖关系，而yum解决rpm的不足，解决rpm的依赖关系

## yum 优势/劣势

优势：自动解决依赖关系

​            可以对rpm进行分组，并基于组进行安装操作

​            引入仓库概念

​            配置简单

劣势：如果仓库中没有对应rpm,那么安装也是会失败



## yum 配置

[yum 配置](../20170601/linux_yum_配置.md)





## yum常用命令



1>yum install software

2>yum remove software

3>yum update software

4>其他命令

4-1>yum list all:显示配置所有配置的rpm软件

​        yum info tigervnc：类似rpm -qi rpm软件

​        yum search software:查询rpm软件

​	yum whatprovides filename 查询rpm中

```
[root@mysql45 Packages]# yum search vnc
Loaded plugins: downloadonly, fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
=========================================== N/S Matched: vnc ============================================
gtk-vnc.i686 : A GTK widget for VNC clients
gtk-vnc.x86_64 : A GTK widget for VNC clients
gtk-vnc-devel.i686 : Libraries, includes, etc. to compile with the gtk-vnc library
gtk-vnc-devel.x86_64 : Libraries, includes, etc. to compile with the gtk-vnc library
gtk-vnc-python.x86_64 : Python bindings for the gtk-vnc library
libvncserver.i686 : Library to make writing a vnc server easy
libvncserver.x86_64 : Library to make writing a vnc server easy
libvncserver-devel.i686 : Development files for libvncserver
libvncserver-devel.x86_64 : Development files for libvncserver
tigervnc.x86_64 : A TigerVNC remote display system
tigervnc-server.x86_64 : A TigerVNC server
tigervnc-server-applet.noarch : Java TigerVNC viewer applet for TigerVNC server
tigervnc-server-module.x86_64 : TigerVNC module to Xorg
tsclient.x86_64 : Client for VNC and Windows Terminal Server
vinagre.x86_64 : VNC client for GNOME
xorg-x11-server-source.noarch : Xserver source code required to build VNC server (Xvnc)

  Name and summary matches only, use "search all" for everything.
#yum list all/installed

[root@mysql45 Packages]# yum whatprovides /etc/yum
Loaded plugins: downloadonly, fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
yum-3.2.29-40.el6.centos.noarch : RPM package installer/updater/manager
Repo        : ysys
Matched from:
Filename    : /etc/yum



yum-3.2.29-40.el6.centos.noarch : RPM package installer/updater/manager
Repo        : installed
Matched from:
Other       : Provides-match: /etc/yum



[root@mysql45 Packages]# 

```



4-2>yum clean all:清空缓存（配置yum后，清空缓存）

​    

## 创建REPO



1、首先将rpm拷贝到某个目录下

2、安装createrepo软件包

3、执行命令 createrepo -v 路径

4、配置yum仓库

```
#rpm -ivh python-deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm  createrepo-0.9.9-18.el6.noarch.rpm deltarpm-3.5-0.5.20090913git.el6.x86_64.rpm 
# createrepo -v .
# ls -ls repodata
# vim /etc/yum.repos.d/**.repo

```

5、yum分组信息

如果想要添加分组信息时，需要在创建repo时

createrepo -g /tmp/*.xml /repo

