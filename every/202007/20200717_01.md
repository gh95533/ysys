[TOC]

# linux  yum  配置





​	yum 是自动化解决rpm包依赖关系的工具，类似一个大型超市，如果你只是要安装电灯泡，但是本身你有没有电源线，电源开关，测线笔，那么就可以在超市中一站式购齐，之后就可以愉快的玩耍了，但是你如果需要在房子里按照一台发电机，不好意思超市没有，或者没有对应的零配件，你就需要自己去其他大型超市重新安装了或者自己去每个相对应的专卖店来购买。

​	yum是一个超市，但是普通的yum源只是能满足基本要求的超市，超出它的能力范围就不可以了,那么去一个更大的yum超市，要么就是手动去购买相应的配件。



​	yum源有多种配置方式：

​	1、本地操作系统安装的镜像文件

​	2、ftp文件

​	3、https文件



## 本地挂载

​	安装完操作系统后，建议将镜像文件拷贝到本地/software文件下，或者直接将镜像文件mount到/media目录下



​	查看挂载路径

```
[root@centos65 yum.repos.d]# df -h

Filesystem                        Size  Used Avail Use% Mounted on

/dev/mapper/vg_centos65-LogVol01   20G  3.3G   15G  19% /

tmpfs                             499M   76K  499M   1% /dev/shm

/dev/sda1                         485M   35M  426M   8% /boot

/dev/sr0                          4.2G  4.2G     0 100% /media

```

​	如果上面没有mount的镜像文件，执行如下命令

```
mount -o loop /dev/sr0 /media
```

​	默认的镜像文件都在/dev/sr0下，如果是手动拷贝到某一路径下，可以将路径替换

```
mount -0 /path/centos.iso /media
```

​	删除/etc/yum.repos.d的repo后缀文件并且新建一个repo后缀文件

```
[root@oracle11 media]# cd /etc/yum.repos.d/

[root@centos65 yum.repos.d]# ls

CentOS-Base.repo       CentOS-Media.repo  

CentOS-Debuginfo.repo  CentOS-Vault.repo

[root@centos65 yum.repos.d]touch ysys.repo

[root@centos65 yum.repos.d]vi ysys.repo

[ysys]

name=yum server configuration

baseurl=file:///media

enabled=1

gpgcheck=0

```

​	执行yum命令，测试yum是否配置成功

```
# yum update
# yum clean all
```

​	介绍*.repo文件中的各个信息

```
[ysys]  --------------->必须写的，中括号的内容可以随便写，但一定要有中括号

name = yum server  ----------->可写可不写，内容随便，主要是个提示作用

baseurl=file:///media  --------------->一定要写的，定义yum源的仓库所在

enabled=1 --------------------->数字1为启用当前yum源，0为禁用，默认为1。

gpgcheck=0  ----------------------->是否检查rpm包的数字签名，数字1为检查，0为不检查，可以不写

```

​	如果是在本地安装的镜像文件，gpgcheck=0,不需要开启，如果是通过网络的方式，那么可能需要开启gpgcheck,校验rpm是否被修改。



## ftp方式安装软件

​	ftp的方式非常简单了，不需要mount镜像文件，只需要参考后面几个小步骤和重点检查配置信息是否出错。

​	删除repo

​	重建repo

```
[root@qbname yum.repos.d]# vi ysys.repo 

[ysys]

name = yum install

baseurl = ftp://username:password@ipaddress/RHEL_6.5_x86_64

enabled=1

gpgcheck=0

```

​	执行yum命令

```
# yum update
# yum clean all
```

​	

​	注：有的ftp 上传下载限制了速度，所以可能较慢







## 问题记录





1-1.查找不到安装包



```
[root@centos65 yum.repos.d]# yum update

Loaded plugins: fastestmirror, refresh-packagekit, security

Loading mirror speeds from cached hostfile

_64&repo=os error was

14: PYCURL ERROR 6 - "Couldn't resolve host 'mirrorlist.centos.org'"

Error: Cannot find a valid baseurl for repo: base

```



将yum.repos.d下的其他文件全部删除，只保留自己新建的文件



2-1.ftp需要输入用户名密码

```
[root@qbname yum.repos.d]# yum update

Loaded plugins: product-id, refresh-packagekit, security, subscription-manager

This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.

ftp://10.201.66.58/RHEL_6.5_x86_64/repodata/repomd.xml: [Errno 14] PYCURL ERROR 67 - "Access denied: 530"

Trying other mirror.

Error: Cannot retrieve repository metadata (repomd.xml) for repository: dvd. Please verify its path and try again

```



在ip地址前面添加username:passwd@



3-1.另外一个正在拷贝

```
Existing lock /var/run/yum.pid: another copy is running as pid 2342
[root@qbsq1 Packages]# rm -rf /var/run/yum.pid
```



4.1.公钥出现问题

```
Public key for libXaw-1.0.11-2.el6.x86_64.rpm is not installed

[root@qbsq1 Packages]#rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

```



## 链接地址

http://www.cnblogs.com/JemBai/archive/2012/11/07/2759140.html