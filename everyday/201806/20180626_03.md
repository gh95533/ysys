
## linux 卸载 openjdk

- 首先了解openjdk和sunjdk区别联系

https://www.cnblogs.com/caibi/p/6319488.html

### centos 6.5 卸载openjdk

操作系统在安装过程中，可能会将一部分openjdk的jar包安装进去，本次使用较为精简安装，没有出现太多的openjdk

- java version

```
[root@gh53 ~]# java -version
java version "1.7.0_45"
OpenJDK Runtime Environment (rhel-2.4.3.3.el6-x86_64 u45-b15)
OpenJDK 64-Bit Server VM (build 24.45-b08, mixed mode)


```
- rpm -qa|grep jdk

```
[root@gh53 ~]# rpm -qa|grep java
java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64
java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64
tzdata-java-2013g-1.el6.noarch

```

- rpm -e --nodeps *** 或者使用命令 yum remove *openjdk*

```
[root@gh53 ~]# rpm -e --nodeps tzdata-java-2013g-1.el6.noarch
[root@gh53 ~]# rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64
[root@gh53 ~]# rpm -e --nodeps java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64

```
- rpm -qa|grep jdk

```
[root@gh53 ~]# rpm -qa|grep java
[root@gh53 ~]# 

```
有些出现javapackages-tools-3.4.1-11.el7.noarch，python-javapackages-3.4.1-11.el7.noarch并不需要删除这些，只要删除openjdk的jar包

- 下载jdk1.8jar包并上传到/software

```

[root@gh53 ~]# cd /software/
[root@gh53 software]# ls
jdk-linux-x64.tar.gz

```

- 解压安装包并将其拷贝到/usr/bin/java下

```
# tar -zxvf jdk-linux-x64.tar.gz
# cp -r jdk1.8.0_31 /usr/bin/java

```

- 修改root家目录下的.bash_profile

```
# cd ~
# vim .bash_profile


export JAVA_HOME=/usr/bin/java  
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export  PATH=${JAVA_HOME}/bin:$PATH


```
- 重新检查java版本

```
[root@gh53 ~]# java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)


```

链接地址：

https://www.cnblogs.com/Dylansuns/p/6974272.html