[TOC]

# gp 部署 yum 坏了



## 问题

​	

​	在`centos`部署完`greenplum`数据库后，需要执行一个`yum -y install xterm* `,安装一个图形化依赖包或者其他软件包时，`yum`就不管用了

```
[root@gp43master yum.repos.d]# yum update

There was a problem importing one of the Python modules

required to run yum. The error leading to this problem was:

   No module named yum

Please install a package which provides this module, or

verify that the module is installed correctly.

It's possible that the above module doesn't match the

current version of Python, which is:

2.6.2 (r262:71600, May 12 2009, 15:34:31)  

[GCC 4.1.1]

If you cannot solve this problem yourself, please go to  

the yum faq at:

  http://yum.baseurl.org/wiki/Faq

```



## 解决方案



  	检查`root`还有其他用户的`bash_profile`或者`.bashrc`环境变量，在默认的`gp`数据库安装时，都会提示需要将命令`source /usr/local/greenplum-db/greenplum_path.sh`写到环境变量中，而`greenplum`依赖的`python`版本和`yum`依赖的`python`版本冲突，导致无法运行`yum`。

​	可以在使用yum时注销`source /usr/local/greenplum-db/greenplum_path.sh`或者是创建一个新的用户，就可以使用`yum`了



## 链接地址



<https://www.cnblogs.com/freem/p/5914863.html>

<http://yum.baseurl.org/wiki/Faq>

https://zhuanlan.zhihu.com/p/34205786