[TOC]

# wps excel 超多行数据导入

## 背景

​	这两天同事给我打电话说有一个excle表格的数据量特别大，无法导入到数据库

## 解决过程

### 修改spoon.bat的内存最大值

​	当前操作系统的内存是32g，设置了最大的值，发现依然报内存不足

### 将excel中的一列拷贝出来的txt文件

​	拷贝一列数据到txt文件，可以写入到数据库，但是拷贝sfzh列时，无法拷贝到txt文件。

### 下载office软件，使用office软件打开文件

​	通过office拷贝文件到txt中，依然无法存入到txt

### 通过office软件另存为csv文件

​	office文件转化为csv时，发现只能转换大部分数据(当时excel中有80多万而csv只有60万)，通过kettlecsv文件写入，大部分数据可以存入

### 通过office软件另外为txt文件

​	office文件转换为txt时，不但可以将数据全部存入txt数据，记得使用"tab"作为分隔符



