[TOC]

# LINUX 关闭 一批服务器

​	其实是在安装gp数据库时想到的当我要将一大批服务器关闭时，一台一台的关闭实在是太累了，都要ssh ip 然后执行 shutdown -h now，现在在网上找到一个脚本，觉得可行做一下测试。



## 操作步骤

### 生成公钥

 ssh-keygen -t rsa

```
# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
90:38:e3:da:a5:9c:e9:50:e1:b5:d1:69:8b:ec:49:8c root@oracle11
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|     . o .       |
|    = = +        |
|   o O * .       |
|    E B S        |
|   = B .         |
|  o * o          |
|   o             |
|    .            |
+-----------------+

```

当需要输入密码，默认回车键就可以了，ssh-keygen -t rsa生成的文件内容在路径<user_home>/.ssh/ 会有id_rsa和id_rsa.pub 两个文件，其中id_rsa.pub为公钥

### 给每台可信任的服务器发送公钥

ssh-copy-id  -i .ssh/id_rsa.pub  root@ip

```
# ssh-copy-id  -i .ssh/id_rsa.pub  root@192.168.1.49
The authenticity of host '192.168.1.49 (192.168.1.49)' can't be established.
RSA key fingerprint is 9c:2a:e5:78:14:6a:57:72:c1:ab:e2:3d:60:7d:44:c6.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.49' (RSA) to the list of known hosts.
root@192.168.1.49's password: 
Now try logging into the machine, with "ssh 'root@192.168.1.49'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.


```

当前路径在root的家目录下，如果是一般的用户也需要在用户的家目录下拷贝到远程的家目录下

### 创建ip文件，执行命令

cat hostlist 

cat doCommand.sh

chmod 777 doCommand.sh 

```
[root@oracle11 ~]# cat hostlist 


192.168.1.45
192.168.1.49

```

```
[root@oracle11 ~]# cat doCommand.sh 
#!/bin/sh

doCommand()
{
    hosts=`sed -n '/^[^#]/p' hostlist`
    for host in $hosts
        do
            echo ""
            echo HOST $host
            ssh $host "$@"
        done
    return 0
}

    if [ $# -lt 1 ]
    then
            echo "$0 cmd"
            exit
    fi
    doCommand "$@"
    echo "return from doCommand"
[root@oracle11 ~]# 

```

hostlist和doCommand.sh都要在一个目录下，doCommand.sh要给予可执行权限

```
# chmod 777 doCommand.sh 

```



### 执行关闭命令

./doCommand.sh  "shutdown -h now"

```
# ./doCommand.sh  "shutdown -h now"


HOST 192.168.1.45

HOST 192.168.1.49
return from doCommand
[root@oracle11 ~]# 

```



​	当然也可以执行其他命令如修改时间或者统一性的东西，不过在主机上可能没有办法赋予进来，所以主机就需要自己手动关闭了







## 链接地址

[linux 免密码登陆](linux_免密码登陆.md)

https://www.linuxprobe.com/public-private-key.html

https://blog.csdn.net/zoombinde/article/details/51902208