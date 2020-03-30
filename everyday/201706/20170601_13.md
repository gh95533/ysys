[TOC]

# linux user  group



linux user group

**linux都需要有一个用户的身份去运行，用户限制使用者获进程可以使用，不可以使用那些资源**

## 组用来方便组织管理用户

每个用户都拥有一个UserID,操作系统实际使用的用户ID，而非用户名

每个用户属于一个主组，属于一个或多个附属组

每个组拥有一个GroupID

每个进程以一个用户身份运行，并受该用户可访问资源的限制

每个可登录用户拥有一个指定的shell

## users

root用户（id为0的用户为root用户），系统用户（1~499），普通用户（500以上）

系统用户是为某些服务创建，但是这些用户只是作为这些进程而服务的，不需要登陆的shell

系统中的文件都有一个所属用户和用户组

使用passwd修改密码

相关文件

/etc/passwd：保存用户信息

/etc/shadow: 保存用户密码(只有root用户可以查看)

/etc/group :保存组信息

### 查看用户登陆

```
[root@mysql45 ~]# whoami
root
[root@mysql45 ~]# who
root     pts/0        2018-08-05 02:50 (192.168.1.1)
[root@mysql45 ~]# w
 02:55:42 up 6 min,  1 user,  load average: 0.00, 0.04, 0.02
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    192.168.1.1      02:50    0.00s  0.02s  0.00s w
```

约定俗成的习惯，命令越长显示的信息越短，命令越短显示的信息越长



### 创建用户

[root@centos65 ~]# useradd gh_study_acc

这个命令执行下列操作

1.在/etc/passwd中添加用户信息

2.如果使用passwd 创建密码，密码信息将会保存在/etc/shadow

3.为用户建立一个新的家目录/home/gh_study_acc

4.将/etc/skel下的文件拷贝到用户家目录中

5.建立一个与用户名相同的组，新建用户默认属于这个同名组

命令useradd支持参数

-d 家目录

-s 登陆shell

-u userid

-g 主组

-G 附属族（最多31个组，用“，”分割）

也可以修改/etc/passwd的方式实现，但是不建议使用

### 修改用户

命令usermod用来修改用户

usermod 参数  username

命令usermod支持参数

-l 新用户名

-u 新userid

-d 用户家目录位置

-g 用户所属主组

-G 用户所属附属组

-L 锁定用户使其不能登陆

-U 解除锁定

usermod -u 777 gh_study_a1;

usermod -l gh_study_a2 gh_study_a1;

### 删除用户

userdel  username(家目录不删除)

userdel -r username（删除家目录）

## group

创建，修改，删除组

groupadd groupname

修改组

groupmod -n newname oldname  修改组名

groupmod -g newGid groupname  修改组ID

删除组

groupdel  groupname