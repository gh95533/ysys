[TOC]

# linux redhat 7.4 install gpload 4.3

**文档整理**

ysys

**日期**

2018-10-23

**标签**

redhat 7.4 install gpload 4.3



## 背景

​	在HT建设过程中发现操作系统都已经升级到了redhat7.4,之前在redhat6.5(centos6.5)安装部署成功了gpload4.3，现在需要在redhat7.4部署一下，查看是否可用



## 环境版本

​	操作系统：redhat 7.4 x64

​	软件包：PyYAML-3.10.tar.gz；yaml-0.1.4.tar.gz；greenplum-loader-4.3.8.1.zip



## 安装手册



###  检查python环境



```
# python
Python 2.7.5 (default, May  3 2017, 07:55:04) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-14)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
```

**python版本要求2.4.4以上**



### 将软件上传到/software下

如果`/software`不存在需要手动创建一下`mkdir /software`



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



###  解压yaml软件

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







## 报错信息

1、python setup.py install 报错：`unable to execute gcc: No such file or directory`

```
# python setup.py install
running install
running build
running build_py
creating build
creating build/lib.linux-x86_64-2.7
creating build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/emitter.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/scanner.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/representer.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/parser.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/nodes.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/tokens.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/cyaml.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/loader.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/resolver.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/events.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/constructor.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/error.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/dumper.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/composer.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/reader.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/serializer.py -> build/lib.linux-x86_64-2.7/yaml
copying lib/yaml/__init__.py -> build/lib.linux-x86_64-2.7/yaml
running build_ext
creating build/temp.linux-x86_64-2.7
checking if libyaml is compilable
gcc -pthread -fno-strict-aliasing -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -D_GNU_SOURCE -fPIC -fwrapv -DNDEBUG -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -D_GNU_SOURCE -fPIC -fwrapv -fPIC -I/usr/include/python2.7 -c build/temp.linux-x86_64-2.7/check_libyaml.c -o build/temp.linux-x86_64-2.7/check_libyaml.o
unable to execute gcc: No such file or directory

libyaml is not found or a compiler error: forcing --without-libyaml
(if libyaml is installed correctly, you may need to
 specify the option --include-dirs or uncomment and
 modify the parameter include_dirs in setup.cfg)
running install_lib
creating /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/emitter.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/scanner.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/representer.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/parser.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/nodes.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/tokens.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/cyaml.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/loader.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/resolver.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/events.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/constructor.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/error.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/dumper.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/composer.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/reader.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/serializer.py -> /usr/lib64/python2.7/site-packages/yaml
copying build/lib.linux-x86_64-2.7/yaml/__init__.py -> /usr/lib64/python2.7/site-packages/yaml
byte-compiling /usr/lib64/python2.7/site-packages/yaml/emitter.py to emitter.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/scanner.py to scanner.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/representer.py to representer.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/parser.py to parser.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/nodes.py to nodes.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/tokens.py to tokens.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/cyaml.py to cyaml.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/loader.py to loader.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/resolver.py to resolver.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/events.py to events.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/constructor.py to constructor.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/error.py to error.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/dumper.py to dumper.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/composer.py to composer.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/reader.py to reader.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/serializer.py to serializer.pyc
byte-compiling /usr/lib64/python2.7/site-packages/yaml/__init__.py to __init__.pyc
running install_egg_info
Writing /usr/lib64/python2.7/site-packages/PyYAML-3.10-py2.7.egg-info
```

​	中间出现报错信息`unable to execute gcc: No such file or directory`执行命令

```
# yum -y install gcc
```



2、gpload.py not found

```
# cd ~
# vim .bash_profile

source /usr/local/greenplum-loaders-4.3.8.1-build-1/greenplum_loaders_path.sh
PYTHONPATH=/usr/local/lib

# source .bash_profile
```





## 链接地址

[linux redhat 6.5 install gpload 4.3](../20170102/linux_redhat_6.5_install_gpload_4.3.md)

