

[TOC]

# LINUX 常用命令

## 查看硬件信息

lspci：查看PCI设备

lsusb:查看USB设备

lsmod:加载的驱动



## 查看内存

free -m

top



## 查看cpu

```
# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l

# 查看CPU信息（型号）
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq 
```



## 关机重启

shutdown -h now:马上挂机

shutdown -r now:马上重启

poweroff:立即关机

reboot:重启



## 归档压缩

zip file.zip file

unzip file.zip

gzip file

tar -cvf file.tar file

tar -xvf file.tar

tar -cvzf file.tar.gz file

tar -xzf file.tar.gz



## 查找

locate keyword:此命令需要预先建立数据库，数据库默认一天一次，可以执行updatedb后重新检查

find / -name ** :在当前目录下查看文件

find . -name **：在根路径下查看文件

find / -perm 777:权限为777文件

find / -type d：根据文件类型

find . -name "a" -exec ls -l {} \：查看a的文件并且执行ls -l；{} \ 固定格式

-name 文件名

-perm 权限

-type 类型

-group 组

-user 用户

-ctime  时间

-size 大小

```
# find  . -size +100k
./alert_orcl.log
超过100k的文件
```



## 日期时间

date:显示当前时间

date -s '20:20:20' 修改当前时间变为20小时20分钟20秒钟

hwclock(clock):硬件时钟时间

cal:日历

uptime:系统当前运行时间



## 输出查看

echo "gddd":返回gddd字段

cat：查看文档

more:向下翻页

less:上下翻页





## 查看Linux操作系统

linux查看版本当前操作系统内核信息

[root@centos65 ~]# uname -a

Linux centos65 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_6

4 x86_64 x86_64 GNU/Linux

linux查看当前操作系统版本信息

[root@centos65 ~]# cat /proc/version

Linux version 2.6.32-431.el6.x86_64 (mockbuild@c6b8.bsys.dev.centos.org) (gcc 

version 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) ) #1 SMP Fri Nov 22 03:15:09 UTC 2013

linux查看版本当前操作系统发行版信息

[root@centos65 ~]# cat /etc/issue

CentOS release 6.5 (Final)

[root@centos65 ~]# cat /etc/redhat-release

CentOS release 6.5 (Final)



## 文本处理

### 复制

在相同目录下复制，需要重新为目标文件起名

```
 cp gh.txt gh2.txt
```

在不同目录下，只需要跟着需要放置的路径

```
 cp gh2.txt ./filedir/
```

将文件夹下的内容拷贝到其他文件夹下

```
cp -r filedir/* ./filedir2/
```

### 移动

move

### 创建文件夹

mkdir  filedir

mkdir -p filedir/filedir2/filedir3

### 删除

rm -rf



