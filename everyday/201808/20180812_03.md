[TOC]

# linux 开机 fsck

## 现象

​	今天早上在启动服务器的时候发现服务器已经被启动了，这是个神奇的情况，因为昨天晚上要停电，现场将应用关闭后就关机了，oracle服务器是远程执行的shutdown -h now,今早来电后，才到机房的，就发现机器已经运行起来，但是并没有对系统做来电启动的操作，这其实只是一个小的插曲，问题是存储没有启动起来，服务器就先启动了起来，这里解释一下，这边做了一套虚拟化操作外加一个oracle集群都是在存储上实现硬盘挂载。

​	首先在机房将虚拟机的服务器全部关掉

​	第二步远程关闭oracle两台服务器，发现无法ping通，在机房检查系统发现有fsck的报错，当时执行命令几乎都不好使，切换用户都报环境变量不存在，之前是以为是没有挂载存储导致的，执行命令shutodnw -h now关闭服务器

​	第三步开启存储，记得当时的关闭顺序是先关扩展柜，再关主柜，后面启动时先启动主柜，在启动扩展柜，前后都要硬盘指示灯亮才好执行下一步启动（这个具体问浪潮？）

​	第四步开启各台服务器

​	第五步发现oracle集群的两台服务器还是无法启动，检查日志发现是

```
/dev/mapper/vg_freeswitch-lv_root UNEXPECTED INCONSISTENCY;RUN fsck MANUALLY(ie.,without -a pr -p options)

an error occured during the file system check

droping you to a shell;the system will reboot

when you leave teh shell.

warning -- selinux is active

disabing security enforcement for system recovery

run 'setenforce 1' to reenable
```

​	当时在机房没带手机和拍照设备，上面是在网上找的类似内容

## 采取步骤(慎重)



​	第一步：分别检查两台服务器是否都是出现这种情况

​	发现都发生了

​	第二步：执行命令 df -Th

​	记得是boot分区

​	第三步：执行命令 fsck

```
# fsck
...
```

​	第四步：重启

​	经过两到三遍重启操作启动完成

​	第五步：检查时间

​	许多重新使用fsck修复的文件系统所在服务器的时间可能会发生变化。

​	第六步：反思shutdown -h now

​	可能与服务器操作系统cpu等兼容问题

​	第七步：修改BIOS时间

```
#  hwclock -r --查看当前BIOS时间
#  date --查看系统时间
#  hwclock -w  --BIOS同步系统时间
```

​	写到这里手都是颤动，就怕服务器没有启动起来(下一周白加黑)。后面在想为什么会出现fsck，fsck又是什么？这个下一篇文档里写。



## 链接地址

https://blog.csdn.net/okhelp/article/details/44220755

http://www.91ctc.com/article/article-439.html

http://blog.chinaunix.net/uid-20940095-id-3239618.html

http://codingstandards.iteye.com/blog/804830



​	

