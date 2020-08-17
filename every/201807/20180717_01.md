[TOC]

# LINUX 磁盘分区 fdisk

​	

​	fdisk是来自于IBM的老牌分区工具，支持绝大多数操作系统，几乎所有的LINUX的发行版本都装有fdisk

​	fdisk是一个基于MBR的分区工具，所以如果需要使用GPT,则无法使用fdisk进行分区

​	

​	fdisk 命令只有具有超级用户权限才能运行

​	fdisk -l可以列出所有安装的磁盘及其分区信息

​	fdisk 设备名 ：fdisk /dev/sdb 可以对其进行分区

​	分区后，可能需要执行partprobe命令更新一下分区表，否则需要重启来识别分区信息

​	/proc/partitions可以查看分区信息

​	

​	fdisk 分区分为主分区和扩展分区，因为fdisk默认只能分配四个主分区，超过四个就没有办法创建主分区，只有通过扩展分区来实现逻辑分区，默认第三个后就是扩展分区，在对扩展分区进行逻辑分区来分配空间。

​	

## fdisk 命令



### fdisk -l 

​	命令查看当前所有安装的磁盘及其分区信息,需要使用root用户来执行

```
# fdisk -l

Disk /dev/sda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0001c390

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          64      512000   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2              64        2611    20458496   8e  Linux LVM

Disk /dev/sdb: 8589 MB, 8589934592 bytes
255 heads, 63 sectors/track, 1044 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/mapper/vg_centos65-LogVol01: 20.7 GB, 20736638976 bytes
255 heads, 63 sectors/track, 2521 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/mapper/vg_centos65-LogVol00: 209 MB, 209715200 bytes
255 heads, 63 sectors/track, 25 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
```

在/dev/sdb下没有任何信息，说明没有分区 (一般)



### fdisk /dev/sdb

fdisk /dev/sdb 将sdb设备分区

```
# fdisk /dev/sdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF diskl
abelBuilding a new DOS disklabel with disk identifier 0x0aad2f73.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').
```



#### m

m:将所有的命令打印出来

```
Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)
```

#### 主要其他命令

n:添加一个分区

p:将分区信息打印出来

w:将修改的变量保存

```
Command (m for help): p

Disk /dev/sdb: 8589 MB, 8589934592 bytes
255 heads, 63 sectors/track, 1044 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0aad2f73

   Device Boot      Start         End      Blocks   Id  System
```

```
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-1044, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-1044, default 1044): +2G
```

在n中要创建分区时，有两种方式，1种是p,主分区模式，1种是e，扩展分区，扩展分区不可直接使用，需要在扩展分区的基础上使用逻辑分区才可以使用。

```
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
e
Partition number (1-4): 2
First cylinder (263-1044, default 263): 
Using default value 263
Last cylinder, +cylinders or +size{K,M,G} (263-1044, default 1044): 
Using default value 1044

Command (m for help): p

Disk /dev/sdb: 8589 MB, 8589934592 bytes
255 heads, 63 sectors/track, 1044 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0aad2f73

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1         262     2104483+  83  Linux
/dev/sdb2             263        1044     6281415    5  Extended

```





```
--创建出来扩展分区不能被使用，需要创建逻辑分区 
--l表示逻辑分区号
Command (m for help): n
Command action
   l   logical (5 or over)
   p   primary partition (1-4)
l
First cylinder (263-1044, default 263): 
Using default value 263
Last cylinder, +cylinders or +size{K,M,G} (263-1044, default 1044): +2G

Command (m for help): p

Disk /dev/sdb: 8589 MB, 8589934592 bytes
255 heads, 63 sectors/track, 1044 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0aad2f73

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1         262     2104483+  83  Linux
/dev/sdb2             263        1044     6281415    5  Extended
/dev/sdb5             263         524     2104483+  83  Linux
```



划分完执行fdisk -l未必 出现分区信息，执行partprobe 

```
# partprobe
Warning: WARNING: the kernel failed to re-read the partition table on /dev/sda
 (Device or resource busy).  As a result, it may not reflect all of your changes until after reboot.Warning: Unable to open /dev/sr0 read-write (Read-only file system).  /dev/sr0
 has been opened read-only.Warning: Unable to open /dev/sr0 read-write (Read-only file system).  /dev/sr0
 has been opened read-only.Error: Invalid partition table - recursive partition on /dev/sr0.
```

```
# fdisk -l

Disk /dev/sda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0001c390

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1          64      512000   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2              64        2611    20458496   8e  Linux LVM

Disk /dev/sdb: 8589 MB, 8589934592 bytes
255 heads, 63 sectors/track, 1044 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0aad2f73

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1         262     2104483+  83  Linux
/dev/sdb2             263        1044     6281415    5  Extended
/dev/sdb5             263         524     2104483+  83  Linux

Disk /dev/mapper/vg_centos65-LogVol01: 20.7 GB, 20736638976 bytes
255 heads, 63 sectors/track, 2521 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/mapper/vg_centos65-LogVol00: 209 MB, 209715200 bytes
255 heads, 63 sectors/track, 25 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

[root@centos65 ~]# cat /proc/partitions
major minor  #blocks  name

   8        0   20971520 sda
   8        1     512000 sda1
   8        2   20458496 sda2
   8       16    8388608 sdb
   8       17    2104483 sdb1
   8       18          1 sdb2
   8       21    2104483 sdb5
 253        0   20250624 dm-0
 253        1     204800 dm-1
```



划分完分区后暂时是裸设备，如果不为其创建文件系统，可能无法使用。有些应用可能需要裸设备，有的应用就需要文件系统了，这个就需要后面看了