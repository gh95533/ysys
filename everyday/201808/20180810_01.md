[TOC]

# pg 9.4.12 source install



​	postgresql源码编译并安装数据库，此文档可以支持postgresql-9.4.x,postgresql-9.5.x,postgresql-9.6.x系列，文档经过测试，可以放心使用。 



## 环境介绍

​	操作系统：centos6.5_x64

​	pg：postgresql-9.4.1.tar.gz 

## 创建用户关闭防火墙修改安装策略



```
# chkconfig --list iptables;
iptables       	0:off	1:off	2:on	3:on	4:on	5:on	6:off
# chkconfig iptables off;
# chkconfig --list iptables;
iptables       	0:off	1:off	2:off	3:off	4:off	5:off	6:off
```

```
# vim /etc/selinux/config 
SELINUX=disabled
```

```
# groupadd ysys
# useradd -g ysys ysys
# passwd ysys
```



## 配置yum源并安装依赖包

### yum源配置

​	略过

### 安装依赖包

```
# yum -y install readline-devel perl-ExtUtils-Embed bison flex zlib zlib-devel python python-devel gcc
```

## 上传文件并解压

```
# tar xvf postgresql-9.4.1.tar.gz 
```

## 源码安装三板斧

### ./configure

​	进入到之前解压缩出来路径后，执行./configure command

```
# cd postgresql-9.4.1
# ./configure --prefix=/usr/local/pgsql --with-perl --with-python
```

--prefix 指定部署的路径

### make

```
# make
```

### make install

```
# make install
```

## 授权用户软件目录权限

```
# chown -R ysys:ysys /usr/local/pgsql/
```

## 编译用户环境变量并初始化环境变量

```
# su - ysys
$ vim .bash_profile

# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH

export PGHOME=/usr/local/pgsql
export PATH=$PGHOME/bin/:$PATH
export PGDATA=$PGHOME/data
export LD_LIBRARY_PATH=$PGHOME/lib:$LD_LIBRARY_PATH

```

```
$ source .bash_profile
```



## 初始化数据库

```
$ initdb -E UNICODE -D $PGHOME/data
```

## 修改配置参数并启动

```
$ cd $PGDATA
$ vim postgresql.conf

listen_addresss=’*’
logging_collector=on

```

​	启动关闭登录数据库

```
$ pg_ctl start 
$ pg_ctl stop

$ psql -d postgres
```

