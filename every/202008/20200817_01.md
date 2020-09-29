[TOC]

# LINUX 挂载 卸载



​	挂载在每个操作系统都是有的，默认在windows，是自动挂载的，而linux需要手动进行挂载，或者配置系统进行自动挂载



	## mount 

​	mount:将格式化好的磁盘或者分区挂载到一个目录下

```
mount /dev/sdb1 /mnt
```

​	不带参数的mount可以显示当前挂载多少个磁盘

```
mount
```

​	-t 指定文件系统的类型

​	-o 指定挂载选项

​		ro,rw 以只读的方式或者读写方式挂载

```
mount -o remount,ro /dev/sdb1 /mnt
```

​		sync 代表不适用缓存，而对所有操作直接写入磁盘

​		async 代表使用缓存，默认是async

​		noatime 代表每次访问文件时不更新文件的访问时间

​		atime  代表每次访问文件时更新文件的访问时间

​		remount 重新挂载文件系统



## umount

​	umount 卸载已挂载的文件系统

```
umount /dev/sdb1
umount /mnt
```

​	如果出现busy时，可以使用命令 fuser -m /mnt 代表那个进程正在使用该路径，或者使用命令 lsof /mnt 列出打开的文件 



## 自动挂载

​	在/etc/fstab用来定义自动挂载的文件系统，fstab中每一行代表一个挂载配置，格式如下

| /dev/sda3      | /mnt       | ext4     | defaults            | 0 0            |
| -------------- | ---------- | -------- | ------------------- | -------------- |
| 需要挂载的设备 | 挂载的路径 | 文件系统 | 默认配置（noatime） | 暂时不需要管它 |



mount -a:命令将fatab自动挂载文件系统挂载

```
mount -a
```

