[TOC]

# man psql:no manual entry for psql



## 执行命令 man psql 查看相关信息说明

```
[osdba@mysql45 ~]$ man psql
No manual entry for psql
```

​	执行命令，出现没有psql选项猜测可能没有出现man的依赖包

```
[root@mysql45 yum.repos.d]# yum -y install man-pages*
```

​	重新执行命令 man psql

```
[osdba@mysql45 ~]$ man psql
No manual entry for psql
```

​	执行一个常见的命令find

```
[root@mysql45 /]# man find
FIND(1)                                                                FIND(1)

NAME
       find - search for files in a directory hierarchy

SYNOPSIS
       find [-H] [-L] [-P] [-D debugopts] [-Olevel] [path...] [expression]

DESCRIPTION
       This  manual  page  documents the GNU version of find.  GNU find searches the directory
       tree rooted at each given file name by evaluating the given  expression  from  left  to
       right,  according to the rules of precedence (see section OPERATORS), until the outcome
       is known (the left hand side is false for and operations, true for or), at which  point
       find moves on to the next file name.

       If you are using find in an environment where security is important (for example if you
       are using it to search directories that are writable by other users), you  should  read
       the  "Security  Considerations" chapter of the findutils documentation, which is called
       Finding Files and comes with findutils.   That document also includes a lot more detail
       and discussion than this manual page, so you may find it a more useful source of infor-
       mation.
...
```

​	不知道是否需要配置一些参数或者需要将文件迁移到某些地方？







