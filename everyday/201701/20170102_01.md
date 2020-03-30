[TOC]

# linux redhat 6.5 install gpload 4.3

**文档整理**

ysys

**日期**

2017-01-02(2018-10-24)

**标签**

redhat 6.5 install gpload 4.3



## 背景

​	kettle的表输出，插入更新控件对于greenplum的支持力度较小，在生产库中直接无法使用，建议使用kettle调用gp的load软件来处理数据；这样就需要在kettle服务器上装一套greenplum loader软件；或者将kettle服务器直接部署在gp的主节点上；建议采用第一种方案；避免对greenplum整个环境的影响



## 环境版本

​	操作系统：redhat 6.5 x64

​	软件包：PyYAML-3.10.tar.gz；yaml-0.1.4.tar.gz；greenplum-loader-4.3.8.1.zip



## 安装部署

​	网上查找资料只有几份关于greenplum loader install的文档，它的部分软件部署是源代码（二进制方式这个说法是错误，之前的文档都是以二级制来命名这个方式，请注意是错误的，应该是源代码形式），所以安装过程可能有这种或者那种的报错，要做好准备。



### 检查python环境

```
# python

Python 2.6.6 (r266:84292, Nov 22 2013, 12:16:22) 

[GCC 4.4.7 20120313 (Red Hat 4.4.7-4)] on linux2

Type "help", "copyright", "credits" or "license" for more information.

>>>exit()

```

**python版本要求2.4.4以上**



### 软件上传到/software

```
# mkdir /software
```



### 解压软件pyyaml文件

```
# cd /software/
# tar -zxvf PyYAML-3.10.tar.gz 
```



### 配置python

```
# cd /software/PyYAML-3.10/
# python setup.py install
---success--
python setup.py install
running install
running build
running build_py
running build_ext
running install_lib
running install_egg_info
Removing /usr/lib64/python2.7/site-packages/PyYAML-3.10-py2.7.egg-info
Writing /usr/lib64/python2.7/site-packages/PyYAML-3.10-py2.7.egg-info
```



### 解压yaml软件

```
# cd /software/
# tar -zxvf yaml-0.1.4.tar.gz
```



### 检查编译环境并生成makefile

```
# cd /software/yaml-0.1.4/
# ./configure
```



### 安装编译软件

```
# cd /software/yaml-0.1.4/
# make && make install 
```



### 解压gp软件

```
# unzip greenplum-loader-4.3.8.1.zip
```



### 安装greenplum-loader

```
# cd /software/greenplum-loader-4.3.8.1/
# ./greenplum-loaders-4.3.8.1-build-1-RHEL5-x86_64.bin 
```

**一路yes,它的文件路径在/usr/local/greenplum-loaders-4.3.8.1-build-1**



### 检查文件内容是否准确

```
# cat /usr/local/greenplum-loaders-4.3.8.1-build-1/greenplum_loaders_path.sh
```

**检查的就是GPLOAD_LOADERS=/usr/local/greenplum-loaders-4.3.8.1-build-1**,如果不对就需要修改了。



### 修改环境变量

```
# vim /root/.bash_profile

source /usr/local/greenplum-loaders-4.3.8.1-build-1/greenplum_loaders_path.sh
PYTHONPATH=/usr/local/lib

# source /root/.bash_profile
```



## 错误

1、python setup.py install 执行报错 

```
# python setup.py install 

python: can't open file 'setup.py': [Errno 2] No such file or directory
```

​	检查一下路径下是否有setup.py文件

2、./configure 执行报错

```
# ./configure 

checking for a BSD-compatible install... /usr/bin/install -c

checking whether build environment is sane... yes

checking for a thread-safe mkdir -p... /bin/mkdir -p

checking for gawk... gawk

checking whether make sets $(MAKE)... yes

checking for gcc... no

checking for cc... no

checking for cl.exe... no

configure: error: in `/opt/yaml-0.1.4':

configure: error: no acceptable C compiler found in $PATH

See `config.log' for more details

```

在报错信息提示中`checking for gcc... no`没有gcc相关的依赖包

```
# yum -y install gcc
```



## 链接地址

<http://blog.csdn.net/feng12345zi/article/details/52046430>

<http://blog.csdn.net/duguduchong/article/details/8699774>

<https://discuss.pivotal.io/hc/zh-cn/community/posts/206424348-Error-Message-gpfdist-error-while-loading-shared-libraries-libyaml-0-so-1>