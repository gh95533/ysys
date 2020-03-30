[TOC]



# linux raid  destory



​	在学习raid原理和操作时，对于raid 0 ,raid 1, raid 5, raid 6的创建，损害，新加的操作步骤还不是很清楚，现在专门整理一下这些文档。

## raid 

### 创建raid

创建raidx

mdadm -C /dev/md0 -a yes -l x -n x /dev/sdb /dev/sdc

-C :创建磁盘路径

-a:自动创建对应设备

-l:raid级别

-n:硬盘数

创建RAID 1

mdadm -C /dev/md0 -a yes -l 1 -n 2 /dev/sdb /dev/sdc

创建RAID 5

mdadm -C /dev/md0 -a yes -l 5 -n 3 /dev/sdb /dev/sdc /dev/sdd

创建RAID 6

mdadm -C /dev/md0 -a yes -l 6 -n 4 /dev/sdb /dev/sdc /dev/sdd /dev/sde

重建RAID 5 多添加一块硬盘:如果有一块坏了，自动替补

mdadm -C /dev/md0 -a yes -l 5 -n 3 -x 1  /dev/sdb /dev/sdc /dev/sdd /dev/sde

### 文件系统并挂载

mkfs.ext4 /dev/md0

mount /dev/md0 /mnt



### 配置启动文件

mdadm -D --scan > /etc/mdadm.conf 

### 配置挂载文件

vi /etc/fstab



### RAID关闭打开

关闭(关闭之前必做umount)：mdadm -S /dev/md0

打开：mdadm -R /dev/md0



### 查看raid信息

cat /proc/mdstat



### 将某块盘置为坏盘

mdadm /dev/md0 -f /dev/sdc

注：raid0执行会提示资源正忙，没有方案执行，不过可以关机拔掉硬盘



### 将坏盘移除

mdadm /dev/md0 -r /dev/sdd 



### 添加新硬盘

mdadm /dev/md0 -a /dev/sdc



### 卸载并将设备变为重新可用设备

​	

​	卸载

​	umount /mnt

​	停止

​	mdadm -S /dev/md0

​	变为可用设备

​	mdadm --zero-superblock /dev/sdb

​	mdadm --zero-superblock /dev/sdc

```
# umount /mnt
# mdadm -S /dev/md0
mdadm: stopped /dev/md0
# mdadm --zero-superblock /dev/sdb
# mdadm --zero-superblock /dev/sdc

```







## raid 0

​	raid 0 使用最少2块硬盘，在读写时将数据分开读写到多块硬盘的方式来提高读写性能,不过没有冗余性，坏了一块，就无法正常工作了



### 创建





mdadm -C /dev/md0 -a yes -l 0 -n 2 /dev/sdb /dev/sdc



```
# mdadm -C /dev/md0 -a yes -l 0 -n 2 /dev/sdb /dev/sdc
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
# cat /proc/mdstat
Personalities : [raid0] 
md0 : active raid0 sdc[1] sdb[0]
      8387584 blocks super 1.2 512k chunks
      
unused devices: <none>
# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Sun Aug  5 13:29:01 2018
     Raid Level : raid0
     Array Size : 8387584 (8.00 GiB 8.59 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

    Update Time : Sun Aug  5 13:29:01 2018
          State : clean 
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

     Chunk Size : 512K

           Name : centos65:0  (local to host centos65)
           UUID : 3b2c7b07:c75f7d46:f6c8ec26:4cc44508
         Events : 0

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc

```





添加到配置文件(重启后依然有效)

mdadm -D --scan > /etc/mdadm.conf 

```
# mdadm -D --scan > /etc/mdadm.conf 
```



创建文件系统并挂载

mkfs.ext4 /dev/md0

mount /dev/md0 /mnt

```
# mkfs.ext4 /dev/md0
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=128 blocks, Stripe width=256 blocks
524288 inodes, 2096896 blocks
104844 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2147483648
64 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 21 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
# mount /dev/md0 /mnt

```

### 模拟环境，当某块盘坏了时

​	mdadm /dev/md0 -f /dev/sdc

```
# mdadm /dev/md0 -f /dev/sdc
mdadm: set device faulty failed for /dev/sdc:  Device or resource busy
```

 	这个命令可能对于raid0这种不适合，那么直接关机，删掉sdc硬盘，重新启动

```
# df -Th
Filesystem                       Type   Size  Used Avail Use% Mounted on
/dev/mapper/vg_centos65-LogVol01 ext4    28G  3.4G   23G  13% /
tmpfs                            tmpfs  499M   72K  499M   1% /dev/shm
/dev/sda1                        ext4   194M   30M  155M  16% /boot
# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Tue Jan 23 06:02:47 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/vg_centos65-LogVol01 /                       ext4    defaults        1 1
UUID=ec8a5405-5b82-4098-bc47-ba7a7368f9c0 /boot                   ext4    defaults        1 2
/dev/mapper/vg_centos65-LogVol00 swap                    swap    defaults        0 0
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5,mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
/dev/md0		/mnt			ext4	defaults	0 0
# mdadm -D /dev/md0
mdadm: cannot open /dev/md0: No such file or directory
# mdadm /dev/md0
mdadm: cannot open /dev/md0: No such file or directory

```

​	发现md0无法启动

```
# mdadm -R /dev/md0
mdadm: error opening /dev/md0: No such file or directory
# cat /proc/mdstat
Personalities : [raid0] 
md127 : inactive sdb[0]
      4193792 blocks super 1.2
       
unused devices: <none>
# mdadm -D /dev/md0
mdadm: cannot open /dev/md0: No such file or directory
# ls -ls md127
0 brw-rw---- 1 root disk 9, 127 Aug  5 14:13 md127

```

​	尝试mdadm -R 启动也没有成功，查看相关参数，发现md0的状态为inactive,从这个测试可以发现，如果raid0环境下，任何一块硬盘的损害都会导致数据丢失。

​	重启并将另外一块硬盘启动，发现依然无法启动/dev/md0,哪怕是环境已经变好

```
# cat /proc/mdstat 
Personalities : [raid0] 
md127 : active raid0 sdc[1] sdb[0]
      8387584 blocks super 1.2 512k chunks
      
unused devices: <none>
# mdadm -D /dev/md0
mdadm: cannot open /dev/md0: No such file or directory

```

### 结论

raid0 确实一块都不能坏掉，坏掉一块就无法使用







## raid 1

### 创建

```
# mdadm -C /dev/md0 -a yes -l 1 -n 2 /dev/sdb /dev/sdc
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? 
Continue creating array? (y/n) y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

```

上面的警告信息是说raid1不可成为/boot分区的

```
# mkfs.ext4 /dev/md0
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
262144 inodes, 1048048 blocks
52402 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1073741824
32 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 37 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
# mount /dev/md0 /mnt

```

挂载完后，在mnt目录下创建文件并写入数据



### 模拟环境，当某块盘坏了时

```
# mdadm /dev/md0 -f /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Sun Aug  5 14:42:17 2018
     Raid Level : raid1
     Array Size : 4192192 (4.00 GiB 4.29 GB)
  Used Dev Size : 4192192 (4.00 GiB 4.29 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

    Update Time : Sun Aug  5 14:46:31 2018
          State : clean, degraded 
 Active Devices : 1
Working Devices : 1
 Failed Devices : 1
  Spare Devices : 0

           Name : centos65:0  (local to host centos65)
           UUID : 4c20fd8a:606ce144:0b954a11:922b3924
         Events : 19

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       0        0        1      removed

       1       8       32        -      faulty   /dev/sdc
# cd /mnt/test
# ls
gh.txt
# vi gh.txt

```

### 结论

​	

​	故意让一块变坏后，依然可以写入读取数据，说明raid1确实只要还有1块可以使用，数据就会保留。



## raid 5

### 创建

```
# mdadm -C /dev/md0 -a yes -l 5 -n 3 /dev/sdb /dev/sdc /dev/sdd
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdd[3] sdc[1] sdb[0]
      4191232 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [==============>......]  recovery = 73.3% (1537664/2095616) finish=0.0min speed=139787K/sec
      
unused devices: <none>
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdd[3] sdc[1] sdb[0]
      4191232 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [===================>.]  recovery = 97.3% (2040684/2095616) finish=0.0min speed=145763K/sec
      
unused devices: <none>
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdd[3] sdc[1] sdb[0]
      4191232 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>

```

### 创建文件系统挂载

```
# mkfs.ext4 /dev/md0
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=128 blocks, Stripe width=256 blocks
262144 inodes, 1047808 blocks
52390 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1073741824
32 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 35 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
# mount /dev/md0 /mnt
# cd /mnt
# mkdir test
# cd test
# vi gh.txt
# cat gh.txt
dfjka;fdkja;


```

### 查看状态

```
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdd[3] sdc[1] sdb[0]
      4191232 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Mon Aug  6 03:27:19 2018
     Raid Level : raid5
     Array Size : 4191232 (4.00 GiB 4.29 GB)
  Used Dev Size : 2095616 (2046.84 MiB 2145.91 MB)
   Raid Devices : 3
  Total Devices : 3
    Persistence : Superblock is persistent

    Update Time : Mon Aug  6 03:31:03 2018
          State : clean 
 Active Devices : 3
Working Devices : 3
 Failed Devices : 0
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 512K

           Name : centos65:0  (local to host centos65)
           UUID : a8b30958:cabce362:7f80d951:f2fec6ff
         Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       3       8       48        2      active sync   /dev/sdd


```

### 模拟坏盘一

#### 模拟一块坏了

```
# mdadm /dev/md0 -f /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdd[3] sdc[1](F) sdb[0]
      4191232 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [U_U]
      
unused devices: <none>
# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Mon Aug  6 03:27:19 2018
     Raid Level : raid5
     Array Size : 4191232 (4.00 GiB 4.29 GB)
  Used Dev Size : 2095616 (2046.84 MiB 2145.91 MB)
   Raid Devices : 3
  Total Devices : 3
    Persistence : Superblock is persistent

    Update Time : Mon Aug  6 03:39:40 2018
          State : clean, degraded 
 Active Devices : 2
Working Devices : 2
 Failed Devices : 1
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 512K

           Name : centos65:0  (local to host centos65)
           UUID : a8b30958:cabce362:7f80d951:f2fec6ff
         Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       0        0        1      removed
       3       8       48        2      active sync   /dev/sdd

       1       8       32        -      faulty   /dev/sdc
# ls
gh.txt
# cat gh.txt
dfjka;fdkja;
# vi gh.txt
# 

```

raid5下当有一块坏盘时，本身还是可以写入数据的，只不过新的数据可能并没有备份出来，比较危险，这是就要重新插入一块硬盘，让数据备份及时生成。

#### 移除坏盘

```
#  mdadm /dev/md0 -r /dev/sdc
mdadm: hot removed /dev/sdc from /dev/md0
# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Mon Aug  6 03:27:19 2018
     Raid Level : raid5
     Array Size : 4191232 (4.00 GiB 4.29 GB)
  Used Dev Size : 2095616 (2046.84 MiB 2145.91 MB)
   Raid Devices : 3
  Total Devices : 2
    Persistence : Superblock is persistent

    Update Time : Mon Aug  6 03:47:36 2018
          State : clean, degraded 
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 512K

           Name : centos65:0  (local to host centos65)
           UUID : a8b30958:cabce362:7f80d951:f2fec6ff
         Events : 33

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       0        0        1      removed
       3       8       48        2      active sync   /dev/sdd

```

#### 将坏盘换为新盘

​	测试环境中只装了3块硬盘，导致将卸载的/dev/sdc需要重新安装上去，不过要先做处理

```
# mdadm --zero-superblock /dev/sdc
# mdadm /dev/md0 -a /dev/sdc
mdadm: added /dev/sdc
# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Mon Aug  6 03:27:19 2018
     Raid Level : raid5
     Array Size : 4191232 (4.00 GiB 4.29 GB)
  Used Dev Size : 2095616 (2046.84 MiB 2145.91 MB)
   Raid Devices : 3
  Total Devices : 3
    Persistence : Superblock is persistent

    Update Time : Mon Aug  6 03:49:53 2018
          State : clean, degraded, recovering 
 Active Devices : 2
Working Devices : 3
 Failed Devices : 0
  Spare Devices : 1

         Layout : left-symmetric
     Chunk Size : 512K

 Rebuild Status : 40% complete

           Name : centos65:0  (local to host centos65)
           UUID : a8b30958:cabce362:7f80d951:f2fec6ff
         Events : 43

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       4       8       32        1      spare rebuilding   /dev/sdc
       3       8       48        2      active sync   /dev/sdd
# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdc[4] sdd[3] sdb[0]
      4191232 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Mon Aug  6 03:27:19 2018
     Raid Level : raid5
     Array Size : 4191232 (4.00 GiB 4.29 GB)
  Used Dev Size : 2095616 (2046.84 MiB 2145.91 MB)
   Raid Devices : 3
  Total Devices : 3
    Persistence : Superblock is persistent

    Update Time : Mon Aug  6 03:50:03 2018
          State : clean 
 Active Devices : 3
Working Devices : 3
 Failed Devices : 0
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 512K

           Name : centos65:0  (local to host centos65)
           UUID : a8b30958:cabce362:7f80d951:f2fec6ff
         Events : 62

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       4       8       32        1      active sync   /dev/sdc
       3       8       48        2      active sync   /dev/sdd


```

​	重新写入数据是没有问题的

### 模拟两块坏了

#### 模拟两块坏了

```
[root@centos65 test]# mdadm /dev/md0 -f /dev/sdc /dev/sdd
mdadm: set /dev/sdc faulty in /dev/md0
mdadm: set /dev/sdd faulty in /dev/md0
[root@centos65 test]# cat /proc/dmstat
cat: /proc/dmstat: No such file or directory
[root@centos65 test]# cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4] 
md0 : active raid5 sdc[4](F) sdd[3](F) sdb[0]
      4191232 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/1] [U__]
      
unused devices: <none>
[root@centos65 test]# mdadm -D /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Mon Aug  6 03:27:19 2018
     Raid Level : raid5
     Array Size : 4191232 (4.00 GiB 4.29 GB)
  Used Dev Size : 2095616 (2046.84 MiB 2145.91 MB)
   Raid Devices : 3
  Total Devices : 3
    Persistence : Superblock is persistent

    Update Time : Mon Aug  6 03:55:01 2018
          State : clean, FAILED 
 Active Devices : 1
Working Devices : 1
 Failed Devices : 2
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 512K

           Name : centos65:0  (local to host centos65)
           UUID : a8b30958:cabce362:7f80d951:f2fec6ff
         Events : 65

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       0        0        1      removed
       2       0        0        2      removed

       3       8       48        -      faulty   /dev/sdd
       4       8       32        -      faulty   /dev/sdc

```

#### 无法写入文件

```
# touch ghh.txt
touch: cannot touch `ghh.txt': Read-only file system

```

### 结论

raid5可以坏掉一块，并且可以正常的运行存储查询操作，不过要及时更换磁盘，如果是同时坏掉两块，那么就无法抢救回来了

## raid 6

### 结论

参考raid5的操作，raid6可以坏掉两块，但是如果是三块的话，就无法抢救回来了



