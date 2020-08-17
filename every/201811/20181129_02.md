[TOC]

# kettle ftp rar zip excel database

**document support**

ysys

**date**

2018-11-29

**label**

kettle,ftp,download,rar,zip,excel



## background

​	今天同事问我一个问题，如何设计从ftp上下载rar和zip文件，本地解析excel文件的这个场景实现自动入库。



## job design



### ftp download 

[ftp下载文件](../20170504/kettle下载ftp文件.md)



### unzip 

​	脚本

```
cd 文件路径
rar x *.rar
winrar x *.zip
```

​	脚本首先需要安装winrar软件，并且在环境变量中加入到path:`c:\program files\winrar`上面路径仅供参考。



### all files in txt

​	利用bat命令将某个文件夹下的所有文件信息写到文件中

```
@echo off
set "PathName=C:\ysys"
del list.txt
for %%a in (%PathName%) do for /f "delims=" %%b in ('dir /a-d/b/s *') do ( 
   echo %%b >>list.txt 
) 
end
```



### list.txt 

​	将list.txt解析到数据库中，并且后面需要判断那个文件需要解析



### excel input

​	注意工作表必须为固定值，如果不是固定值，无法动态解析



### del rar zip

​	当工作完成时，记得删除rar和zip文件。





​	可以微调这个先后顺序，不过rar和zip文件只是多了一步解压缩而已，并不是多么困难。











​	