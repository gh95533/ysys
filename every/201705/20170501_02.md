[TOC]

# datastage install in linux

## 环境准备

​	

​	在生产环境下，普遍采用的是centos系列，目前常用的是centos6.5_x64,不过现在centos7.3以及开始在普及中。该文档现在以centos6.5为例来安装部署datastage,后续会在centos7.3上测试是否安装成功。



linux版本：centos6.5X64

oracle版本：oracle11gr2

datastage版本：datagstage9.1

[操作系统安装](../20170601/centos_6.5_install.md)

oracle安装

## 操作步骤

​	

​	假设当前操作系统和oracle都已经部署完成



### 切换到oracle环境并在oracle数据库上创建dswas



sqlplus / as sysdba

create user dswas identified by 123456;

grant dba to dswas;



```
[oracle@ds91 ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on Tue Nov 7 03:54:44 2017

Copyright (c) 1982, 2009, Oracle.  All rights reserved.


Connected to:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> create user dswas identified by 123456;

User created.

SQL> grant dba to dswas;

Grant succeeded.

SQL> 
```



### 切换到root用户并且上传安装包 



cd /opt/tool

tar -xzvf IIS_V9.1_LINUX_X86_64_ML.tar.gz 

chmod 777 /opt/tool/is-suite/DatabaseSupport/UNIX_Linux/MetadataRepository/Oracle11g

cd /opt/tool/is-suite/DatabaseSupport/UNIX_Linux/MetadataRepository/Oracle11g

chmod 775 create_xmeta_db.sh



```
[root@ds91 tool]# chmod 777 /opt/tool/is-suite/DatabaseSupport/UNIX_Linux/MetadataRepository/Oracle11g
[root@ds91 tool]# cd /opt/tool/is-suite/DatabaseSupport/UNIX_Linux/MetadataRepository/Oracle11g/
[root@ds91 Oracle11g]# ls -ls
total 24
4 -rw-rw-rw- 1 root root 1034 Nov  8  2012 configure_xmeta_db.sql
8 -rw-rw-rw- 1 root root 6102 Nov  8  2012 create_xmeta_db.sh
4 -rw-rw-rw- 1 root root 1444 Nov  8  2012 create_xmeta_db_tablespace.sql
4 -rw-rw-rw- 1 root root 1323 Nov  8  2012 create_xmeta_db_user.sql
4 -rw-rw-rw- 1 root root  316 Nov  8  2012 README
[root@ds91 Oracle11g]# chmod 775 create_xmeta_db.sh
[root@ds91 Oracle11g]# 
```



### 切换到oracle用户并执行脚本



su - oracle

cd /opt/tool/is-suite/DatabaseSupport/UNIX_Linux/MetadataRepository/Oracle11g

./create_xmeta_db.sh system dragon orcl ds91meta 123456 xmetats /u01/app/oracle/product/11.2.0/db_1



```
[oracle@ds91 Oracle11g]$ ./create_xmeta_db.sh system dragon orcl ds91meta 123456 xmetats /u01/app/oracle/product/11.2.0/db_1
Checking Required files....
setting datafile path ...


The IBM Information Server metadata repository database will be created:

Oracle SID : orcl
Tablespace : xmetats
Schema name : ds91meta
Data file : /u01/app/oracle/product/11.2.0/db_1/xmetats.dbf

Press [CTRL/C]; to abort or [ENTER] to continue
Creating tablespace .....
Tablespace created successfully
Modifying DB config
Database configuration modified successfully
Creating metadata repository user .....
User created successfully
./create_xmeta_db.sh completed
[oracle@ds91 Oracle11g]$ ./create_xmeta_db.sh system dragon orcl ds91meta 123456 xmetats /u01/app/oracle/product/11.2.0/db_1
```



### 登录到sys用户并给ds91meta授权



grant dba to ds91meta



### 解压zip文件并将授权文件并在指定路径



unzip Bundle.suite.workgroup.zip 

mv license/ is-suite/

mv image.properties is-suite/



```
[root@ds91 tool]# mv license/ is-suite/
[root@ds91 tool]# mv image.properties is-suite/
```



### 使用root用户并且开始安装图形化界面



cd /opt/tool/is-suite/

./setup



```
[root@ds91 is-suite]# ./setup 
INFO: Installation program will be running in this session, please do not close installation window until installation program completes.
Installation program started at 11/7/17 4:28 AM
Nov 7, 2017 4:28:35 AM java.util.prefs.FileSystemPreferences$2 run
INFO: Created user preferences directory.


======> Enter one of the following URLs to your web browser to begin the installation process:
http://ds91:8084/ISInstall
https://ds91:8445/ISInstall
```



### 在firefox上输入`http://ds91:8084/ISInstall`

可能会报错，将ds91直接换成ip地址，输入后，选择语言是中文

![img](../img_src/bdb4ec0fe5984ca7b834c3c38cbe6f81/clipboard.png)

下一步》下一步

![img](../img_src/82e1e09aac2a41618a053762b87f3271/clipboard.png)

下一步

![img](../img_src/19bcc740370844db90e951af4d48ada6/clipboard.png)

下一步选择服务引擎

![img](../img_src/88833a8f17a847f499f83c4b5c982d48/clipboard.png)

下一步选择english+ibm infosphere datastage

![img](../img_src/c13d320b9ede46ad9f35456f96f1637b/clipboard.png)

下一步选择接受协议

![img](../img_src/c47d38b4f370436ba5b98b42284263b0/clipboard.png)

下一步选择DataStage版本，勾选第一项（parallel job及server job）：

![img](../img_src/6264ca4988414413b83455541b855020/clipboard.png)

下一步Information Server自带的主/备模式HA配置，本例中不选用，直接点击下一步

![img](../img_src/9ebd9df7d2af42699b6c40d309888ea7/clipboard.png)

下一步选择安装

![img](../img_src/fa45d0ca74b04073bc4895c54a419a75/clipboard.png)

下一步设置WAS安装路径，默认安装在/opt/IBM目录下，默认勾选端口列表

![img](../img_src/eb26b940838549a3bbfe5499ae9def45/clipboard.png)

下一步端口列表中，是当前WAS使用的端口号，左侧是描述及默认端口，右侧是实际值，如发现实际端口与默认不符，则可能是/etc/services文件中被其它条目占用，安装程序自动选择了相邻的端口，可根据需要对/etc/services文件进行修改，或沿用安装程序自动适配的端口号，确认后

![img](../img_src/da0e681614a2471f933d180a7b1f8bc4/clipboard.png)

下一步设置WAS账号及密码 wasadmin/123456

![img](../img_src/18d28937873045b7a6b1181252b346c1/clipboard.png)

下一步设置IS管理用户及密码 isadmin/123456

![img](../img_src/1ea389c145524083a4b8652ee63155a8/clipboard.png)

下一步修改数据库相关信息

![img](../img_src/3cd2c4ba03894511849f0c72e2a060e5/clipboard.png)

下一步修改数据库相关信息

![img](../img_src/b239d62524d64c4c8804482a53ff4067/clipboard.png)

下一步：IS ASB登录代理及日志端口设置，默认是31533和31538

![img](../img_src/9235d8e31585400cb07e66393b35daef/clipboard.png)

下一步：作业监视器端口配置：DataStage job monitor端口，默认是13400和13401

![img](../img_src/b358145f4bf04d43abfb43ab958e9a22/clipboard.png)

下一步：用于一台服务器上安装多个DataStage引擎的情况，本例不涉及此场景，留空，继续下一步

![img](../img_src/2fc8a5054df846728b558a2aa42d49eb/clipboard.png)

下一步：建立DataStage管理员账号dsadm及密码

![img](../img_src/c7a6ab0946c24afe83e8ccbf9e80916b/clipboard.png)

![img](../img_src/96c3cab12fb64e36bd89c91a34148869/clipboard.png)

下一步：安装NLS支持，用于支持处理中文

![img](../img_src/2769d5c3e35943b59d1fa4deb9b00288/clipboard.png)

下一步：MQ插件，本例不涉及，留空，不安装，继续下一步

![img](../img_src/11171513b170422aab17f5cf8b6f10ce/clipboard.png)

下一步：SAS配置，本例不涉及，留空不安装，继续下一步

![img](../img_src/cc0a2c2cf8f8467c9a3225a314a00154/clipboard.png)

下一步：1. 创建操作数据库，仍然使用oracle，设置相应用户名及密码，数据库位置留空，继续下一步：dsodb/123456

![img](../img_src/88626c2a416f46ae886ba691490354d3/clipboard.png)

下一步默认的DataStage project，暂保留，不做更改，继续下一步：

![img](../img_src/7cc0f4e82a1f4015a9e3d54e6d98880b/clipboard.png)

下一步检查根据相关警告修改

![img](../img_src/4e758b6b23f441ebbc762f38e2f5eec2/clipboard.png)

yum -y install libXp*;yum -y install compat-libstdc*;yum -y install libXmu*;

修改配置参数

vim /etc/sysctl.conf 

kernel.sem = 250 128000 32 1024

sysctl -p

重新检查除了内存外

下一步：安装程序将前面所有配置好的内容放入一个响应文件，作为模板供下次安装时使用

![img](../img_src/22ed1723a51c4485b93475e13643d6a1/clipboard.png)

下一步

![img](../img_src/802874d6fd5343178efbebe748812b37/clipboard.png)

下一步：安装进程开始

![img](../img_src/e32787cfa90b48d993324b7dc743249d/clipboard.png)

![img](../img_src/9280f1d652644777a1930e97c3dfb307/clipboard.png)

<http://ds91:9080/ibm/iis/console>

<http://ds91:9080/infocenter/index.jsp>

<http://ds91:9080/ibm/imam/console>

`下一步输入(http://ds91:9080/ibm/imam/console)`，在浏览器打开Information server管理页面，使用wasadmin用户登录

![img](../img_src/c34472cf83674394b644c58f78583d79/clipboard.png)

下一步：欢迎界面

![img](../img_src/5709b13dab76453580f9fedc20c60b7d/clipboard.png)

下一步：1. 找到Administration界面，选择Users and Groups，点击Users可看见当前服务器上的用户列表，这时我们需要新建一个用户用于DataStage client的登录，点击右侧New User：

![img](../img_src/b7822ceb64d94a8c9783331172869aaf/clipboard.png)

下一步：1. 在新用户创建界面，输入用户名密码等信息，红色星号为必填，在中间的Roles窗口，为此用户指定权限，Suite是指此用户在当前web界面中的权限，下方Suite Component是指此用户对于Information Server所有产品的使用权限，可根据实际需要进行设定：

![img](../img_src/a3667fa29bc44491b050260314ce88b7/clipboard.png)

下一步设置完成后，点击右下方的Save and close：

下一步此时可见用户列表中，多出了新建的test用户

![img](../img_src/ed536713352d42c1bdc2993ee85bdfe8/clipboard.png)

下一步：1. 在Domain Management菜单里，我们需要对此新建的用户进行后台引擎的授权，让此用户拥有启动DataStage引擎，执行ETL job的权力；

选择Engine Credential，在中间选择提供ETL服务的主机名（列表内容根据登录的ETL服务器而不同），点击右侧的Open User Credential

![img](../img_src/8a6b23f3a60f4018902d28afdd66ec8b/clipboard.png)

下一步：这里的列表是空的，点击中下部的Browse按钮

![img](../img_src/73e0ecf2f3734d9d85a535d9748885f8/clipboard.png)

下一步：选择之前建立的新用户test，并点击ok

![img](../img_src/26b7256b41084064afab9be60beeee0b/clipboard.png)

![img](../img_src/7b2a24ad92894d73b88d9eec438b5311/clipboard.png)

下一步在右侧输入ETL服务器dsadm的用户名和密码，并点击Apply

![img](../img_src/6200e71f59d8474da1b90d643cb58d21/clipboard.png)

下一步：用户名密码验证正确后，可以看到中部的test用户已经与服务器操作系统的dsadm账号建立了映射关系：

![img](../img_src/47a7ca5406b44177b4c351c7afde8a07/clipboard.png)

至此，Information Server的client用户创建结束，可以关闭web窗口

## 关闭流程

[root@ds91 share]# cd /opt/IBM/InformationServer/Server/DSEngine/bin

[root@ds91 bin]# ./uv -admin -stop

Stopping JobMonApp

JobMonApp has been shut down.

resource_tracker has been shutdown.

DataStage Engine 9.1.0.0 instance "ade" has been brought down.

[root@ds91 bin]# cd /opt/IBM/InformationServer/ASBNode/bin

[root@ds91 bin]# ./NodeAgents.sh stop

Agent stopped.

AgentService stopped.

LoggingAgent stopped.

[root@ds91 bin]# su - oracle

[oracle@ds91 ~]$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on Tue Nov 7 06:54:10 2017

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to:

Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production

With the Partitioning, OLAP, Data Mining and Real Application Testing options

SQL> shutdown immediate;

Database closed.

Database dismounted.

ORACLE instance shut down.

SQL> exit

Disconnected from Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production

With the Partitioning, OLAP, Data Mining and Real Application Testing options

[oracle@ds91 ~]$ lsnrctl stop

LSNRCTL for Linux: Version 11.2.0.1.0 - Production on 07-NOV-2017 06:54:42

Copyright (c) 1991, 2009, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))

The command completed successfully

[oracle@ds91 ~]$ 