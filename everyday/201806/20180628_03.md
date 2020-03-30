

[TOC]

# pg_列式存储\_2

​	在上一个章节中介绍了通过EPEL按照依赖包来支持cstore_fdw的安装，现在需要做的是将之前的依赖包之间下载，看看是否可以直接在本地安装(当前很多环境不能连接互联网)，或者将下载的包放到新环境中，使用rpm来安装rpm包

## 直接下载

​	此方案请在百度上直接搜索

```
protobuf-2.3.0-9.el6.x86_64.rpm   
protobuf-c-devel-0.15-2.el6.x86_64.rpm
protobuf-c-0.15-2.el6.x86_64.rpm  
protobuf-compiler-2.3.0-9.el6.x86_64.rpm

```



## 间接下载

​	首先通过在互联网安装系统时下载镜像包而非直接安装就可以了

### 配置本地yum源并且安装包yum-downloadonly

```
# mount -o loop <>.iso /media
# rm -rf /etc/yum.repos.d/*
# vim /etc/yum.repos.d/ysys.repo
[ysys]
name=ysys
baseurl=file:///media
enabled=1
gpgcheck=0
# yum install yum-downloadonly
```

### 下载epel文件

本次是使用centos6.5,使用命令

```
#  rpm -q epel-release
package epel-release is not installed
# rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm 
Retrieving http://mirrors.ustc.edu.cn/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm
warning: /var/tmp/rpm-tmp.7OLYWM: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Preparing...                ########################################### [100%]
   1:epel-release           ########################################### [100%]

```

### 执行下载命令

```
# yum install --downloadonly protobuf-c-devel
Loaded plugins: downloadonly, fastestmirror, refresh-packagekit, security
Loading mirror speeds from cached hostfile
epel/metalink                                                                     | 4.7 kB     00:00     
 * epel: mirrors.tuna.tsinghua.edu.cn
epel                                                                              | 3.2 kB     00:00     
epel/primary                                                                      | 3.2 MB     00:05     
epel                                                                                         12514/12514
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
Total                                                                    815 kB/s | 584 kB     00:00     


exiting because --downloadonly specified 


```

### 查找下载路径

本次是默认下载，路径在/var/cache/yum/.../packages

```
# cd /var/cache/yum/x86_64/6/epel/packages
# ls -ls
total 596
292 -rw-r--r-- 1 root root 296132 Dec 18  2014 protobuf-2.3.0-9.el6.x86_64.rpm
 96 -rw-r--r-- 1 root root  94992 Apr 26  2011 protobuf-c-0.15-2.el6.x86_64.rpm
 16 -rw-r--r-- 1 root root  12568 Apr 26  2011 protobuf-c-devel-0.15-2.el6.x86_64.rpm
192 -rw-r--r-- 1 root root 194752 Dec 18  2014 protobuf-compiler-2.3.0-9.el6.x86_64.rpm
```

### 记录本次安装rpm的步骤

```
# rpm -ivh protobuf-2.3.0-9.el6.x86_64.rpm 
# rpm -ivh protobuf-compiler-2.3.0-9.el6.x86_64.rpm
# rpm -ivh protobuf-c-0.15-2.el6.x86_64.rpm 
# rpm -ivh protobuf-c-devel-0.15-2.el6.x86_64.rpm 
```

安装完成后，后面的步骤请参考[pg_列式存储](pg_列式存储)



## 总结

​	其实本次安装部署就是到如果没有互联网的情况下，如何部署安装相关的依赖包，建议使用间接安装方式，此方案后期要推荐出去。



