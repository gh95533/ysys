[TOC]

# LINUX XMANAGER VNC

​	一般来说，在linux安装oracle时可能会使用到linux图形化界面，当时使用的是XMANAGER的xstart软件，后面开始使用VNC来远程连接linux图形化界面。那么两者有什么相同点和不同点。



## 相同点

​	都是linux图形化界面的远程连接工具，只有这一点

## 不同点

​	1、使用的协议不同。

​	XMANAGER使用的SSH协议，通过端口将主机服务器的UI界面引导到本地展现，如果网络中断，操作将失败，而VNC使用的RFB协议，即便网络中断，也不影响操作的进行，等于说其是在操作系统进行操作 （在windows远程桌面后，如果网络断开，之前跑的东西并不是影响）



​	2、vnc 其实是c/s架构，需要在服务器上安装vnc软件，在客户端安装vnc-client软件，而xmanager默认只需要服务安装的xterm依赖包

​	

​	3、针对第二点，vnc安装比较麻烦，而xterm则较为简单

​	

1. 4、xmanager图形化界面要想运行，就要求linux操作系统必须运行在级别5上(yum 	-y xterm*) 