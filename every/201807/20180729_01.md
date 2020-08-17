[TOC]

# rpm noarch.rpm src.rpm



​	在今天学习rpm时，发现一些有意思的rpm包，就觉得有意思，需要了解一些rpm的命令规范

​	

## 通用命令规范

​	name-version-arch.rpm 

​	name-version-arch.src.rpm 

name:指的是软件名称

version:指的是软件版本版本号的格式通常为“主版本号.次版本号.修正号”

arch:如:i386,表示包的适用的硬件平台，目前RPM支持的平台：i386,i586,i686,sparc和alpha

.rpm或者.src.rpm,是RPM包类型的后缀，.rpm是编译好的二进制包，可用rpm命令直接安装；.src.rpm表示源代码包，需要安装源码包生成源码，并对源码编译生成.rpm格式的RPM包，就可以对这个RPMBA包进行安装了



特殊名称：

1、el* 表示这个软件包的发行商版本，el5表示这个软件包在RHEL 5.x/Centos 5.x下使用，当然目前linux并不只是redhat/centos系列，还有SUSE..,这样它们的简称可不是el

2、devel:表示这个RPM包是软件的开发包

3、noarch:说明这个软件包可以在任何平台上安装，不需要特定的硬件平台。在任何硬件平台上都可以运行。

4、manual:手册文档



例子：

httpd-2.2.3-29.el5.i386.rpm 

httpd-devel-2.2.3-29.el5.i386.rpm 

httpd-manual-2.2.3-29.el5.i386.rpm 

system-config-httpd-1.3.3.3-1.el5.noarch.rpm 

yum-utils-1.1.30-14.el6.src.rpm



httpd-2.2.3-29.el5.i386.rpm 

httpd:软件名

2.2.3:主版本号.次版本号.修正号

29：是发布版本号，表示这个RPM包是第几次编译生成的

el5:reahat 5.x

i386:32位操作系统



httpd-devel-2.2.3-29.el5.i386.rpm 

devel:软件开发包



httpd-manual-2.2.3-29.el5.i386.rpm 

manual：手册文档



system-config-httpd-1.3.3.3-1.el5.noarch.rpm 

noarch.rpm:代表多少位操作系统都可以



yum-utils-1.1.30-14.el6.src.rpm

src.rpm:源代码rpm



src.rpm 源码rpm，需要编译后才能使用

rpm:普通rpm，对应相依的平台使用

noarch.rpm：不过是什么平台都可以使用(操作系统还是要兼顾)

