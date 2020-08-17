[TOC]

# linux 免密码登陆

​	ssh-keygen:创建公钥和密钥,会生成id_rsa和id_rsa.pub两个文件

​	ssh-copy-id:把本地的公钥复制到远程主机的authorized_keys文件(不会覆盖文件，是追加到文件末尾)，并且会设置远程主机用户目录的.ssh和.ssh/authorized_keys权限

测试 使用osdba用户

[osdba@pg41 ~]$ ssh-keygen -t rsa

[osdba@pg41 ~]$ ssh-copy-id -i  .ssh/id_rsa.pub [osdba@192.168.1.42](mailto:osdba@192.168.1.42)

[osdba@pg42 ~]$ ssh-keygen -t rsa

[osdba@pg42 ~]$ ssh-copy-id  -i .ssh/id_rsa.pub osdba@192.168.1.41

它们之间就可以互相登陆了