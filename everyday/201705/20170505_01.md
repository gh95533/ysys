[TOC]

# install greenplum loader in linux



install greenplum loader in linux

kettle的表输出，插入更新控件对于greenplum的支持力度较小，在生产库中直接无法使用，建议使用kettle调用gp的load软件来处理数据；这样就需要在kettle服务器上装一套greenplum loader软件；或者将kettle服务器直接部署在gp的主节点上；建议采用第一种方案；避免对greenplum整个环境的影响。

网上查找资料只有几份关于greenplum loader install的文档，它的部分软件部署是源代码（二进制方式这个说法是错误，之前的文档都是以二级制来命名这个方式，请注意是错误的，应该是源代码形式），所以安装过程可能有这种或者那种的报错，要做好准备。

操作系统：CentOS release 6.5 (Final)

软件：PyYAML-3.10.tar.gz；greenplum-loaders-4.3.8.1-build-1-RHEL5-x86_64.zip；yaml-0.1.4.tar.gz

在结合多个地方部署过程中遇到最大的情况时，没有安装xterm,后续的图形化界面无法打开，因此，在看本文档时，记住第一件事就是安装xterm.

yum -y xterm*

查看操作系统

cat /etc/redhat-release or cat /etc/issue

```
[root@etl ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@etl ~]# cat /etc/issue
CentOS release 6.5 (Final)
Kernel \r on an \m
```



查看python版本

python

python版本要求2.4.4以上

将软件都上传到opt文件

解压pyyaml文件

tar -zxvf PyYAML-3.10.tar.gz

配置python

python setup.py install

解压yaml软件

tar -zxvf yaml-0.1.4.tar.gz

检查编译环境并生成makefile

./configure

生成可执行文件并安装(之前文档注释直接是二进制安装，是有歧义的)

make && make install 

解压软件

unzip greenplum-loaders-4.3.8.1-build-1-RHEL5-x86_64.zip

安装greenplum-loader

./greenplum-loaders-4.3.8.1-build-1-RHEL5-x86_64.bin 

查看greenplum_loaders_path.sh 下的$GPHOME_LOADERS$ 路径是否正确

cat greenplum_loaders_path.sh 

一般如果不是修改文件路径就不会有问题

到root家目录下修改bash_profile[date:2017-08-09 修改]

vim .bash_profile

pgdatabase是创建的数据库

pghost是安装数据库的主机名

pguser:用户名

使环境生效

source .bash_profile

source greenplum_loaders_path.sh 

到目前为止，安装已经完成了，但是我们需要验证

启动gpfdist

nohup /usr/local/greenplum-loaders-4.3.8.1-build-1/bin/gpfdist -d /home/admin/ -p 8888 > /tmp/gpfdist.log 2>&1 &

创建模板文件（*.yml）

vim abc.yml 

执行脚本

gpload -f abc.yml 

整个过程分为三个小软件安装以及相对应的依赖包，测试环境是否搭建成功，这个步骤做完之后，就可以使用kettle调用greenplum loader控件来传入数据了

=========================================================================

错误

1.1、python setup.py install 执行报错

solation：

仔细查看setup.py的路径，应该到解压包路径

2.1、./configure 执行报错

资料：<http://blog.csdn.net/duguduchong/article/details/8699774>

在config.log中检查gcc,没有找到，现在需要安装gcc的依赖包

yum -y install gcc*

3.1 nohup /usr/local/greenplum-loaders-4.3.8.1-build-1/bin/gpfdist -d /home/admin/ -p 8888 > /tmp/gpfdist.log 2>&1 & 执行报错

查看日志

资料：<https://discuss.pivotal.io/hc/zh-cn/community/posts/206424348-Error-Message-gpfdist-error-while-loading-shared-libraries-libyaml-0-so-1>

source greenplum_loaders_path.sh 

4.1 gpload -f abc.yml  执行报错

去主机节点上修改pg_hba.conf文件

vim pg_hba.conf

gpstop -u

4.2 gpload -f abc.yml  执行报错

文件中报错信息： ERROR:  permission denied: no privilege to create a readable gpfdist(s) external table

修改参数文件postgresql.conf中的两个参数

gp_external_enable_exec = on   # enable external tables with EXECUTE.

gp_external_grant_privileges = on  #enable create http/gpfdist for non su's

5.1安装完这些之后，在使用yum源可能就会报错，暂时无法解决

主要是因为使用了gpload的python依赖包，导致无法正常使用，因此建议先安装xterm包，可以使用xstart远程连接。

文章链接地址：

<http://blog.csdn.net/feng12345zi/article/details/52046430>

<http://blog.csdn.net/duguduchong/article/details/8699774>

<https://discuss.pivotal.io/hc/zh-cn/community/posts/206424348-Error-Message-gpfdist-error-while-loading-shared-libraries-libyaml-0-so-1>