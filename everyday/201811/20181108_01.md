[TOC]

# ibm datastage not load library

**文档整理**

ysys

**日期**

2018-11-08

**标签**

datastage,error,not load library



## 背景

​	昨天同事给我说了这样一个情况，一台运行了有一年的datastage服务器(windows版本)突然无法执行，并且一直在报一个错误（Error loading connector library ccora10g.dll...)



## 处理过程

​	1、让同事检查是否有ccora10g.dll这个文件

​	2、将环境变量转给我看



​	同事给我反馈说有这个文件，将环境变量粘贴给我看，我发现没有明显问题，和他说今天帮他看一下。

​	上去到那台电脑首先发现特别慢，特别慢，可能是个通病；

​	运行一下报错的方案，发现确实是这个问题，一直报错(Error loading connector library ccora10g.dll...)

​	好好检查了一下环境变量，觉得需要将datastage环境变量放在最前面比较好，就和同事商量说修改环境变量，重启服务器操作后看一下。

​	重启服务器后，同事打电话过来说不行，我好好检查了一些，发现数据库可能重装，和他说，可能需要将之前的环境找到才行。





## 链接地址

https://www.ibm.com/support/knowledgecenter/zh/SSZJPZ_11.3.0/com.ibm.swg.im.iis.ds.trouble.ref.doc/topics/connectorlibrary.html



