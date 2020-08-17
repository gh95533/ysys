[TOC]

# linux 高级权限管理-ACL

​	传统的UGO权限模型无法应对复杂的权限设置需求，如对于一个文件只能设置一个组，并且对该组进行权限控制，但是如果该文件有多个组会对其进行访问，并且都要进行权限限制时，传统的UGO模型就无法满足需求了。

## ACL

​	ACL(Access Control List)是一种高级权限机制，允许对一个文件或文件夹进行灵活的，复杂的权限设置

​	ACL需要在挂载文件的时候打开ACL功能：

```
mount -o acl /dev/sda5 /mnt
```

​	ACL允许针对不同用户，不同组对一个目标文件/文件夹进行权限设置，不受UGO模型限制

## ACL 操作

​	查看一个文件,文件夹的ACL设置（默认根路径自带getfacl)

```
getfacl filename
```

​	针对一个用户对文件进行ACL设置

```
getfacl -m u:user:rwx filename
```

​	针对一个组对文件进行ACL设置

```
setfacl -m g:group:rwx filename
```

​	删除一个ACL设置

```
setfacl -x u:username filename
```





## 链接地址

https://www.cnblogs.com/Jimmy1988/p/7249844.html