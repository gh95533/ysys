[TOC]

# linux rpm



## 总结

​	

​	rpm较源代码方式最突出的优势：就是不需要再次编译就可以运行，但不能改变当依赖包不存在时，无法正常安装的情况。



## 源代码

​	在介绍rpm包，之前为大家介绍一下源代码(突然想到电影了)

​	1、绝大多数开源软件都是以源代码形式发布

​	2、一般打包成tar.gz的归档压缩文件

​	3、源代码需要编译称为二进制形式才能使用

​	4、源代码基础编译流程:

```
./configure 检查编译环境并生成makefile

make 对源代码进行编译，生成可执行文件

make install 将生成的可执行文件安装到当前计算机中

```

​	好处：兼用性及可控制性较好,适用所有系统，可定制

​	坏处：使用起来比较麻烦，编译时间较长，极容易出现错误

## RPM

​	为了方便使用，开发了RPM(reahat package manager)

​	rpm通过将源代码基于平台系统编译为可执行文件，并保存依赖关系，来简化开源软件的安装管理

​	RPM设计目录如下：

```
1、使用简单
2、使用单一软件包格式文件发布
3、可升级
4、追踪软件依赖关系(不能自动解决)
5、基本信息查询
6、软件验证功能
7、支持多平台(不同的平台会发布不同的rpm包)
```

​	

### RPM 文件名解析

参考文档:[rpm 命名](../20180729/rpm_noarch.rpm_src.rpm.md)





### RPM常用命令

1>安装软件:rpm -i software.rpm

2>卸载软件：rpm -e software(只跟程序的名字）

3>升级形式安装:rpm -U software-new.rpm

注：rpm支持http,ftp协议安装软件

4>rpm参数

-v 显示详细信息

-h 显示进度条

rpm -ivh <http://www.baidu.com/software.rpm>

```
[root@mysql45 Packages]# rpm -ivh tigervnc-1.1.0-5.el6_4.1.x86_64.rpm 
warning: tigervnc-1.1.0-5.el6_4.1.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
Preparing...                ########################################### [100%]
   1:tigervnc               ########################################### [100%]

```



5>查询功能

--查询已安装软件包信息情况

-qa 安装所有安装的rpm软件

-qi 软件名：查询软件的名字，创建者，版本，所属组

-ql 软件名：列出所有该软件的文件

-qf 文件名：指出该文件属于那个安装包



```
[root@mysql45 Packages]# rpm -qa| grep yum
yum-plugin-downloadonly-1.1.30-14.el6.noarch
yum-utils-1.1.30-14.el6.noarch
yum-plugin-fastestmirror-1.1.30-14.el6.noarch
PackageKit-yum-0.5.8-21.el6.x86_64
yum-3.2.29-40.el6.centos.noarch
PackageKit-yum-plugin-0.5.8-21.el6.x86_64
yum-plugin-security-1.1.30-14.el6.noarch
yum-metadata-parser-1.1.2-16.el6.x86_64

[root@mysql45 Packages]# rpm -qi yum
Name        : yum                          Relocations: (not relocatable)
Version     : 3.2.29                            Vendor: CentOS
Release     : 40.el6.centos                 Build Date: Fri 22 Feb 2013 07:26:41 PM CST
Install Date: Tue 23 Jan 2018 06:05:16 AM CST      Build Host: c6b8.bsys.dev.centos.org
Group       : System Environment/Base       Source RPM: yum-3.2.29-40.el6.centos.src.rpm
Size        : 4735341                          License: GPLv2+
Signature   : RSA/SHA1, Sun 24 Feb 2013 01:50:22 AM CST, Key ID 0946fca2c105b9de
Packager    : CentOS BuildSystem <http://bugs.centos.org>
URL         : http://yum.baseurl.org/
Summary     : RPM package installer/updater/manager
Description :
Yum is a utility that can check for and automatically download and
install updated RPM packages. Dependencies are obtained and downloaded
automatically, prompting the user for permission as necessary.

[root@mysql45 Packages]# rpm -ql yum
/etc/bash_completion.d
/etc/bash_completion.d/yum.bash
/etc/logrotate.d/yum
/etc/yum
/etc/yum.conf
...
/var/lib/yum/uuid
/var/lib/yum/yumdb

[root@mysql45 Packages]# rpm -qf /var/lib/yum
yum-3.2.29-40.el6.centos.noarch
```

-- 查询未安装软件包信息

rpm -qip software.rpm:参考rpm -qi

rpm -qlp software.rpm:参考rpm -ql



```
[root@mysql45 Packages]# rpm -qip tigervnc-1.1.0-5.el6_4.1.x86_64.rpm 
warning: tigervnc-1.1.0-5.el6_4.1.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
Name        : tigervnc                     Relocations: (not relocatable)
Version     : 1.1.0                             Vendor: CentOS
Release     : 5.el6_4.1                     Build Date: Mon 29 Apr 2013 07:35:32 PM CST
Install Date: (not installed)               Build Host: c6b7.bsys.dev.centos.org
Group       : User Interface/Desktops       Source RPM: tigervnc-1.1.0-5.el6_4.1.src.rpm
Size        : 659349                           License: GPLv2+
Signature   : RSA/SHA1, Mon 29 Apr 2013 07:42:00 PM CST, Key ID 0946fca2c105b9de
Packager    : CentOS BuildSystem <http://bugs.centos.org>
URL         : http://www.tigervnc.com
Summary     : A TigerVNC remote display system
Description :
Virtual Network Computing (VNC) is a remote display system which
allows you to view a computing 'desktop' environment not only on the
machine where it is running, but from anywhere on the Internet and
from a wide variety of machine architectures.  This package contains a
client which will allow you to connect to other desktops running a VNC
server.
[root@mysql45 Packages]# rpm -qlp tigervnc-1.1.0-5.el6_4.1.x86_64.rpm 
warning: tigervnc-1.1.0-5.el6_4.1.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID c105b9de: NOKEY
/usr/bin/vncviewer
/usr/share/applications/vncviewer.desktop
/usr/share/doc/tigervnc-1.1.0
/usr/share/doc/tigervnc-1.1.0/LICENCE.TXT
/usr/share/doc/tigervnc-1.1.0/README.txt
/usr/share/icons/hicolor
/usr/share/icons/hicolor/16x16
/usr/share/icons/hicolor/16x16/apps
/usr/share/icons/hicolor/16x16/apps/tigervnc.png
/usr/share/icons/hicolor/24x24
/usr/share/icons/hicolor/24x24/apps
/usr/share/icons/hicolor/24x24/apps/tigervnc.png
/usr/share/icons/hicolor/48x48
/usr/share/icons/hicolor/48x48/apps
/usr/share/icons/hicolor/48x48/apps/tigervnc.png
/usr/share/locale/de/LC_MESSAGES/tigervnc.mo
/usr/share/locale/fr/LC_MESSAGES/tigervnc.mo
/usr/share/locale/pl/LC_MESSAGES/tigervnc.mo
/usr/share/locale/ru/LC_MESSAGES/tigervnc.mo
/usr/share/locale/sk/LC_MESSAGES/tigervnc.mo
/usr/share/locale/sv/LC_MESSAGES/tigervnc.mo
/usr/share/man/man1/vncviewer.1.gz
[root@mysql45 Packages]# 

```



## rpm 验证

​	软件在传播过程中可能会被恶意的修改，所以为了安全期间都加入了对软件的验证功能。

​	验证一般使用非对称加密算法

​	导入密钥:

​	rpm --import RPM-GPG-KEY-Centos-6

​	验证未安装的软件

​	rpm  -K software.rpm

​	验证已安装的软件

​	rpm -V software

​	

​	rpm 验证这一块知识点还不是很懂，可能需要后面更加深入了解。





​	

