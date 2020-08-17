



[TOC]

# EPEL_使用

## 背景

​	在这次部署postgresql的cstore_fdw时，需要安装依赖包protobuf-c-devel，默认的镜像包没有protobuf-c-devel，觉得有必须要了解一下这个信息

## EPEL

### 概念

 	 **EPEL** (Extra Packages for Enterprise Linux)是基于Fedora的一个项目，为“红帽系”的操作**系统提供额外的软件包**，适用于RHEL、CentOS和Scientific Linux. 

### 安装

​	本次操作系统是centos6.5_x64，如果安装其他版本请参考链接信息

1. 检查epel-release是否存在

   ```
   # rpm -q epel-release
   package epel-release is not installed
   ```

2. 下载epel-release包

   ```
   # rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm  
   Retrieving http://mirrors.ustc.edu.cn/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm
   warning: /var/tmp/rpm-tmp.OizIze: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
   Preparing...                ########################################### [100%]
      1:epel-release           ########################################### [100%]
   ```

3. 执行安装依赖包命令

   ```
   # yum install protobuf-c-devel
   Loaded plugins: fastestmirror, refresh-packagekit, security
   Loading mirror speeds from cached hostfile
    * epel: mirrors.huaweicloud.com
   Setting up Install Process
   Resolving Dependencies
   --> Running transaction check
   ---> Package protobuf-c-devel.x86_64 0:0.15-2.el6 will be installed
   --> Processing Dependency: protobuf-c = 0.15-2.el6 for package: protobuf-c-devel-0.15-2.el6.x86_64
   --> Processing Dependency: libprotobuf-c.so.0()(64bit) for package: protobuf-c-devel-0.15-2.el6.x86_64
   --> Running transaction check
   ---> Package protobuf-c.x86_64 0:0.15-2.el6 will be installed
   --> Processing Dependency: libprotoc.so.6()(64bit) for package: protobuf-c-0.15-2.el6.x86_64
   --> Processing Dependency: libprotobuf.so.6()(64bit) for package: protobuf-c-0.15-2.el6.x86_64
   --> Running transaction check
   ---> Package protobuf.x86_64 0:2.3.0-9.el6 will be installed
   ---> Package protobuf-compiler.x86_64 0:2.3.0-9.el6 will be installed
   --> Finished Dependency Resolution
   
   Dependencies Resolved
   
   =========================================================================================================
    Package                         Arch                 Version                   Repository          Size
   =========================================================================================================
   Installing:
    protobuf-c-devel                x86_64               0.15-2.el6                epel                12 k
   Installing for dependencies:
    protobuf                        x86_64               2.3.0-9.el6               epel               289 k
    protobuf-c                      x86_64               0.15-2.el6                epel                93 k
    protobuf-compiler               x86_64               2.3.0-9.el6               epel               190 k
   
   Transaction Summary
   =========================================================================================================
   Install       4 Package(s)
   
   Total download size: 584 k
   Installed size: 1.8 M
   Is this ok [y/N]: y
   Downloading Packages:
   (1/4): protobuf-2.3.0-9.el6.x86_64.rpm                                            | 289 kB     00:00     
   (2/4): protobuf-c-0.15-2.el6.x86_64.rpm                                           |  93 kB     00:00     
   (3/4): protobuf-c-devel-0.15-2.el6.x86_64.rpm                                     |  12 kB     00:00     
   (4/4): protobuf-compiler-2.3.0-9.el6.x86_64.rpm                                   | 190 kB     00:00     
   ---------------------------------------------------------------------------------------------------------
   Total                                                                    342 kB/s | 584 kB     00:01     
   warning: rpmts_HdrFromFdno: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
   Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
   Importing GPG key 0x0608B895:
    Userid : EPEL (6) <epel@fedoraproject.org>
    Package: epel-release-6-8.noarch (installed)
    From   : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
   Is this ok [y/N]: y
   Running rpm_check_debug
   Running Transaction Test
   Transaction Test Succeeded
   Running Transaction
   Warning: RPMDB altered outside of yum.
     Installing : protobuf-2.3.0-9.el6.x86_64                                                           1/4 
     Installing : protobuf-compiler-2.3.0-9.el6.x86_64                                                  2/4 
     Installing : protobuf-c-0.15-2.el6.x86_64                                                          3/4 
     Installing : protobuf-c-devel-0.15-2.el6.x86_64                                                    4/4 
     Verifying  : protobuf-c-0.15-2.el6.x86_64                                                          1/4 
     Verifying  : protobuf-compiler-2.3.0-9.el6.x86_64                                                  2/4 
     Verifying  : protobuf-2.3.0-9.el6.x86_64                                                           3/4 
     Verifying  : protobuf-c-devel-0.15-2.el6.x86_64                                                    4/4 
   
   Installed:
     protobuf-c-devel.x86_64 0:0.15-2.el6                                                                   
   
   Dependency Installed:
     protobuf.x86_64 0:2.3.0-9.el6  protobuf-c.x86_64 0:0.15-2.el6  protobuf-compiler.x86_64 0:2.3.0-9.el6 
   
   Complete!
   ```

   其实配置对简单，但是如果前期配置yum源将repo全部删除，可能就需要直接下载镜像包。如果没有删除repo,那么可以直接执行脚本 yum install epel-release 

   

## 链接地址

﻿https://support.rackspace.com/how-to/install-epel-and-additional-repositories-on-centos-and-red-hat/

https://fedoraproject.org/wiki/EPEL

https://www.cnblogs.com/fps2tao/p/7580188.html

