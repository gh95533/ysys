[TOC]

# linux 创建文件系统

​	文件系统非常重要，在linux中绝大多数的应用都是依赖于文件系统。



## 文件系统

​	创建文件系统的过程又称之为格式化。	

​	没有文件系统的设备称之为裸设备

​	常见的文件系统有fat32,nfts,ext2,ext3,ext4,xfs,hfs等等

​	文件系统之间的区别：日志，支持的分区大小，支持的单个文件大小，性能等	

​	windows下的主流系统是NTFS

​	linux的主流文件系统是ext3,ext4

​	

​	linux支持的文件系统：proc,gfs,jfs,iso9660,nfs,vfat,fat(msdos),ext2,ext3,ext4



## 创建文件系统



### mke2fs

​	mke2fs:用来创建文件系统

​	mke2fs -t  ext4 /dev/sdb1

​	常用参数

​	-b blocksize 指定文件系统块大小

​	-c                    建立文件系统时检查块损害

​	-L  label         指定卷标

​	-j                     建立文件系统日志

​	ext3,ext4默认带日志系统，不需要参数j



### mkfs

​	mkfs:是mke2fs的简化版

​	mkfs也可以用来创建文件系统，相对于mke2fs命令更加简单，但是支持命令较少

```
mkfs.ext4 /dev/sdb1

mkfs.ext3 /dev/sdb1

mkfs.vfat /dev/sdb1

```

### dumpe2fs

​	dumpe2fs:查看分区的文件系统

```
dumpe2fs /dev/sdb1
```

​	dumpe2fs：主要针对的是ext2,ext3,ext4,对于xfs无法显示，而且无法对逻辑卷中pv的/dev/sda2,无法显示结果

​	

### journal 日志



​	journal 日志

​	带日志的文件系统(ext3,ext4)拥有较强的稳定性，在出现错误时进行恢复。

​	使用带日志的文件系统，文件系统会使用一个叫做“两阶段提交”的方式进行磁盘操作，当进行磁盘操作

​	时，文件系统进行一下操作

​	

​	1）文件系统将准备执行的事务的具体内容写入日志

​	2）文件系统进行操作

​	3） 操作成功后，将事务的具体内容从日志删除



​	这样做的好处时，当断电或者磁盘故障时，可以通过查询日志进行恢复操作，缺点就是

​	丧失一定的性能（额外的日志读写操作）



###	 E2LABLE



​	命令e2lable可以用来对文件系统打标签

```
# 查看
e2lable   /dev/sdb1 

# 设置
e2lable  /dev/sdb1   GUOHUI

```

​	打标签主要用于管理，打得标签最好是大写字母



### FSCK

​        FSCK:命令fsck用来检查并修复损坏的文件系统

```
 fsck /dev/sdb1
```

​	使用-y参数不提示而直接进行修复

​	默认fsck会自动判断文件系统，如果文件系统损害较为严重，请使用-t参数指定文件系统类型

​	对于识别为文件系统的损害数据（文件系统无记录),fsck会将该文件放入到lost+found目录

​	系统启动时会对磁盘进行fdisk操作



​	**注意：检查文件系统需要将文件系统卸载(挂载反义词)**



​	



​	





