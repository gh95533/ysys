[TOC]

# pg gprof 目录

​	

​	同事问我gprof目录里什么？是做什么用的？是否可以删除？



## gprof怎么产生的



​	在编译软件时产生的

```
# ./configure --help
`configure' configures PostgreSQL 9.4.1 to adapt to many kinds of systems.

Usage: ./configure [OPTION]... [VAR=VALUE]...

To assign environment variables (e.g., CC, CFLAGS...), specify them as
VAR=VALUE.  See below for descriptions of some of the useful variables.

Defaults for the options are specified in brackets.

Configuration:
  -h, --help              display this help and exit
      --help=short        display options specific to this package
      --help=recursive    display the short help of all the included packages
  -V, --version           display version information and exit
  -q, --quiet, --silent   do not print `checking ...' messages
      --cache-file=FILE   cache test results in FILE [disabled]
  -C, --config-cache      alias for `--cache-file=config.cache'
  -n, --no-create         do not create output files
      --srcdir=DIR        find the sources in DIR [configure dir or `..']

Installation directories:
  --prefix=PREFIX         install architecture-independent files in PREFIX
                          [/usr/local/pgsql]
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                          [PREFIX]

By default, `make install' will install all the files in
`/usr/local/pgsql/bin', `/usr/local/pgsql/lib' etc.  You can specify
an installation prefix other than `/usr/local/pgsql' using `--prefix',
for instance `--prefix=$HOME'.

For better control, use the options below.

Fine tuning of the installation directories:
  --bindir=DIR            user executables [EPREFIX/bin]
  --sbindir=DIR           system admin executables [EPREFIX/sbin]
  --libexecdir=DIR        program executables [EPREFIX/libexec]
  --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
  --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
  --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
  --libdir=DIR            object code libraries [EPREFIX/lib]
  --includedir=DIR        C header files [PREFIX/include]
  --oldincludedir=DIR     C header files for non-gcc [/usr/include]
  --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
  --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
  --infodir=DIR           info documentation [DATAROOTDIR/info]
  --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
  --mandir=DIR            man documentation [DATAROOTDIR/man]
  --docdir=DIR            documentation root [DATAROOTDIR/doc/postgresql]
  --htmldir=DIR           html documentation [DOCDIR]
  --dvidir=DIR            dvi documentation [DOCDIR]
  --pdfdir=DIR            pdf documentation [DOCDIR]
  --psdir=DIR             ps documentation [DOCDIR]

System types:
  --build=BUILD     configure for building on BUILD [guessed]
  --host=HOST       cross-compile to build programs to run on HOST [BUILD]

Optional Features:
  --disable-option-checking  ignore unrecognized --enable/--with options
  --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)
  --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]
  --disable-integer-datetimes
                          disable 64-bit integer date/time support
  --enable-nls[=LANGUAGES]
                          enable Native Language Support
  --disable-rpath         do not embed shared library search path in
                          executables
  --disable-spinlocks     do not use spinlocks
  --enable-debug          build with debugging symbols (-g)
  --enable-profiling      build with profiling enabled
  --enable-coverage       build with coverage testing instrumentation
  --enable-dtrace         build with DTrace support
  --enable-tap-tests      enable TAP tests (requires Perl and IPC::Run)
  --enable-depend         turn on automatic dependency tracking
  --enable-cassert        enable assertion checks (for debugging)
  --disable-thread-safety disable thread-safety in client libraries
  --disable-largefile     omit support for large files
  --disable-float4-byval  disable float4 passed by value
  --disable-float8-byval  disable float8 passed by value

Optional Packages:
  --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]
  --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)
  --with-extra-version=STRING
                          append STRING to version
  --with-template=NAME    override operating system template
  --with-includes=DIRS    look for additional header files in DIRS
  --with-libraries=DIRS   look for additional libraries in DIRS
  --with-libs=DIRS        alternative spelling of --with-libraries
  --with-pgport=PORTNUM   set default port number [5432]
  --with-blocksize=BLOCKSIZE
                          set table block size in kB [8]
  --with-segsize=SEGSIZE  set table segment size in GB [1]
  --with-wal-blocksize=BLOCKSIZE
                          set WAL block size in kB [8]
  --with-wal-segsize=SEGSIZE
                          set WAL segment size in MB [16]
  --with-CC=CMD           set compiler (deprecated)
  --with-tcl              build Tcl modules (PL/Tcl)
  --with-tclconfig=DIR    tclConfig.sh is in DIR
  --with-perl             build Perl modules (PL/Perl)
  --with-python           build Python modules (PL/Python)
  --with-gssapi           build with GSSAPI support
  --with-krb-srvnam=NAME  default service principal name in Kerberos (GSSAPI)
                          [postgres]
  --with-pam              build with PAM support
  --with-ldap             build with LDAP support
  --with-bonjour          build with Bonjour support
  --with-openssl          build with OpenSSL support
  --with-selinux          build with SELinux support
  --without-readline      do not use GNU Readline nor BSD Libedit for editing
  --with-libedit-preferred
                          prefer BSD Libedit over GNU Readline
  --with-uuid=LIB         build contrib/uuid-ossp using LIB (bsd,e2fs,ossp)
  --with-ossp-uuid        obsolete spelling of --with-uuid=ossp
  --with-libxml           build with XML support
  --with-libxslt          use XSLT support when building contrib/xml2
  --with-system-tzdata=DIR
                          use system time zone data in DIR
  --without-zlib          do not use Zlib
  --with-gnu-ld           assume the C compiler uses GNU ld [default=no]

Some influential environment variables:
  CC          C compiler command
  CFLAGS      C compiler flags
  LDFLAGS     linker flags, e.g. -L<lib dir> if you have libraries in a
              nonstandard directory <lib dir>
  LIBS        libraries to pass to the linker, e.g. -l<library>
  CPPFLAGS    (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if
              you have headers in a nonstandard directory <include dir>
  CPP         C preprocessor
  LDFLAGS_EX  extra linker flags for linking executables only
  LDFLAGS_SL  extra linker flags for linking shared libraries only
  DOCBOOKSTYLE
              location of DocBook stylesheets

Use these variables to override the choices made by `configure' or to help
it to find libraries and programs with nonstandard names/locations.

Report bugs to <pgsql-bugs@postgresql.org>.


```

​	找到参数 `--enable-profiling      build with profiling enabled`当编译pg数据软件时，带上这个参数就会出现gprof目录



​	在初始化数据库时出现

```
[ysys@centos65 ~]$ initdb -E UNICODE -D $PGHOME/data
The files belonging to this database system will be owned by user "ysys".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /usr/local/pgsql/data ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
creating template1 database in /usr/local/pgsql/data/base/1 ... ok
initializing pg_authid ... ok
initializing dependencies ... ok
creating system views ... ok
loading system objects' descriptions ... ok
creating collations ... ok
creating conversions ... ok
creating dictionaries ... ok
setting privileges on built-in objects ... ok
creating information schema ... ok
loading PL/pgSQL server-side language ... ok
vacuuming database template1 ... ok
copying template1 to template0 ... ok
copying template1 to postgres ... ok
syncing data to disk ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    postgres -D /usr/local/pgsql/data
or
    pg_ctl -D /usr/local/pgsql/data -l logfile start

[ysys@centos65 ~]$ cd $PGDATA

[ysys@centos65 data]$ ls -lt
total 112
drwx------ 19 ysys ysys  4096 Aug 10 11:50 gprof
drwx------  5 ysys ysys  4096 Aug 10 11:50 base
drwx------  2 ysys ysys  4096 Aug 10 11:50 global
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_notify
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_clog
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_subtrans
drwx------  3 ysys ysys  4096 Aug 10 11:50 pg_xlog
-rw-------  1 ysys ysys  4456 Aug 10 11:50 pg_hba.conf
-rw-------  1 ysys ysys  1636 Aug 10 11:50 pg_ident.conf
-rw-------  1 ysys ysys    88 Aug 10 11:50 postgresql.auto.conf
-rw-------  1 ysys ysys 21261 Aug 10 11:50 postgresql.conf
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_dynshmem
drwx------  4 ysys ysys  4096 Aug 10 11:50 pg_logical
drwx------  4 ysys ysys  4096 Aug 10 11:50 pg_multixact
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_replslot
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_serial
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_snapshots
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_stat
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_stat_tmp
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_tblspc
drwx------  2 ysys ysys  4096 Aug 10 11:50 pg_twophase
-rw-------  1 ysys ysys     4 Aug 10 11:50 PG_VERSION

```

​	查看`$PGDATA`目录发现目录名为`gprof`



## gprof目录文件分布

```
[ysys@centos65 gprof]$ ls -lR
.:
total 68
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10662
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10664
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10665
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10667
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10669
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10671
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10673
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10676
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10678
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10680
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10682
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10684
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10686
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10688
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10690
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10692
drwx------ 2 ysys ysys 4096 Aug 10 11:50 10694

./10662:
total 1680
-rw------- 1 ysys ysys 1717882 Aug 10 11:50 gmon.out

./10664:
total 1680
-rw------- 1 ysys ysys 1718008 Aug 10 11:50 gmon.out

./10665:
total 1720
-rw------- 1 ysys ysys 1761058 Aug 10 11:50 gmon.out

./10667:
total 1720
-rw------- 1 ysys ysys 1759147 Aug 10 11:50 gmon.out

./10669:
total 1756
-rw------- 1 ysys ysys 1794994 Aug 10 11:50 gmon.out

./10671:
total 1796
-rw------- 1 ysys ysys 1835251 Aug 10 11:50 gmon.out

./10673:
total 1808
-rw------- 1 ysys ysys 1850539 Aug 10 11:50 gmon.out

./10676:
total 1788
-rw------- 1 ysys ysys 1829602 Aug 10 11:50 gmon.out

./10678:
total 1736
-rw------- 1 ysys ysys 1776955 Aug 10 11:50 gmon.out

./10680:
total 1736
-rw------- 1 ysys ysys 1774141 Aug 10 11:50 gmon.out

./10682:
total 1752
-rw------- 1 ysys ysys 1790542 Aug 10 11:50 gmon.out

./10684:
total 1868
-rw------- 1 ysys ysys 1910053 Aug 10 11:50 gmon.out

./10686:
total 1744
-rw------- 1 ysys ysys 1785565 Aug 10 11:50 gmon.out

./10688:
total 1732
-rw------- 1 ysys ysys 1772293 Aug 10 11:50 gmon.out

./10690:
total 1776
-rw------- 1 ysys ysys 1815154 Aug 10 11:50 gmon.out

./10692:
total 1776
-rw------- 1 ysys ysys 1817044 Aug 10 11:50 gmon.out

./10694:
total 1720
-rw------- 1 ysys ysys 1759714 Aug 10 11:50 gmon.out

```

​	命令ls -lR可以递归打印出来当前文件夹下的所有文件夹及文件



## gprof目录

​	在目录中发现只有gmon.out的文件，那么上层它的上层目录一堆数字又是什么？

​	在一些文档中写的是pid,gprof会为每个相关的pid生成一个目录和gmon.out文件

​	在这里设计一个小实验，测试一下是不是gprof会为每个postgresql相关进程创建一个目录和文件

​	

​	操作步骤：

​	1、每隔1秒记录ysys用户(postgresql安装的用户)的进程情况

​	2、重新初始化initdb

​	3、将initdb所产生的进程和gprof目录名相对照

​	4、pg启动和关闭



编译文件检测ysys进程

```
# vim psu.sh 
# /bin/bash
ps aux| grep ysys | grep -v grep  >> /tmp/ysys.log
```

通过crontab -e设置每隔一秒检查进程（笨办法）

```
# crontab -e
* * *  *  *     /root/psu.sh
* * *  *  *     sleep 1; /root/psu.sh
* * *  *  *     sleep 2; /root/psu.sh
* * *  *  *     sleep 3; /root/psu.sh
* * *  *  *     sleep 4; /root/psu.sh
* * *  *  *     sleep 5; /root/psu.sh
* * *  *  *     sleep 6; /root/psu.sh
* * *  *  *     sleep 7; /root/psu.sh
* * *  *  *     sleep 8; /root/psu.sh
* * *  *  *     sleep 9; /root/psu.sh
* * *  *  *     sleep 10; /root/psu.sh
* * *  *  *     sleep 11; /root/psu.sh
* * *  *  *     sleep 12; /root/psu.sh
* * *  *  *     sleep 13; /root/psu.sh
* * *  *  *     sleep 14; /root/psu.sh
* * *  *  *     sleep 15; /root/psu.sh
* * *  *  *     sleep 16; /root/psu.sh
* * *  *  *     sleep 17; /root/psu.sh
* * *  *  *     sleep 18; /root/psu.sh
* * *  *  *     sleep 19; /root/psu.sh
* * *  *  *     sleep 20; /root/psu.sh
* * *  *  *     sleep 21; /root/psu.sh
* * *  *  *     sleep 22; /root/psu.sh
* * *  *  *     sleep 23; /root/psu.sh
* * *  *  *     sleep 24; /root/psu.sh
* * *  *  *     sleep 25; /root/psu.sh
* * *  *  *     sleep 26; /root/psu.sh
* * *  *  *     sleep 27; /root/psu.sh
* * *  *  *     sleep 28; /root/psu.sh
* * *  *  *     sleep 29; /root/psu.sh
* * *  *  *     sleep 30; /root/psu.sh
* * *  *  *     sleep 31; /root/psu.sh
* * *  *  *     sleep 32; /root/psu.sh
* * *  *  *     sleep 33; /root/psu.sh
* * *  *  *     sleep 34; /root/psu.sh
* * *  *  *     sleep 35; /root/psu.sh
* * *  *  *     sleep 36; /root/psu.sh
* * *  *  *     sleep 37; /root/psu.sh
* * *  *  *     sleep 38; /root/psu.sh
* * *  *  *     sleep 39; /root/psu.sh
* * *  *  *     sleep 40; /root/psu.sh
* * *  *  *     sleep 41; /root/psu.sh
* * *  *  *     sleep 42; /root/psu.sh
* * *  *  *     sleep 43; /root/psu.sh
* * *  *  *     sleep 44; /root/psu.sh
* * *  *  *     sleep 45; /root/psu.sh
* * *  *  *     sleep 46; /root/psu.sh
* * *  *  *     sleep 47; /root/psu.sh
* * *  *  *     sleep 48; /root/psu.sh
* * *  *  *     sleep 49; /root/psu.sh
* * *  *  *     sleep 50; /root/psu.sh
* * *  *  *     sleep 51; /root/psu.sh
* * *  *  *     sleep 52; /root/psu.sh
* * *  *  *     sleep 53; /root/psu.sh
* * *  *  *     sleep 54; /root/psu.sh
* * *  *  *     sleep 55; /root/psu.sh
* * *  *  *     sleep 56; /root/psu.sh
* * *  *  *     sleep 57; /root/psu.sh
* * *  *  *     sleep 58; /root/psu.sh
* * *  *  *     sleep 59; /root/psu.sh
```

重新初始化initdb

```
$ initdb -E UNICODE -D $PGHOME/data
The files belonging to this database system will be owned by user "ysys".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /usr/local/pgsql/data ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
creating template1 database in /usr/local/pgsql/data/base/1 ... ok
initializing pg_authid ... ok
initializing dependencies ... ok
creating system views ... ok
loading system objects' descriptions ... ok
creating collations ... ok
creating conversions ... ok
creating dictionaries ... ok
setting privileges on built-in objects ... ok
creating information schema ... ok
loading PL/pgSQL server-side language ... ok
vacuuming database template1 ... ok
copying template1 to template0 ... ok
copying template1 to postgres ... ok
syncing data to disk ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    postgres -D /usr/local/pgsql/data
or
    pg_ctl -D /usr/local/pgsql/data -l logfile start


```

检查gprof目录

```
$ ls -lt /usr/local/pgsql/data/gprof/

drwx------ 2 ysys ysys 4096 Aug 11 03:30 6527
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6525
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6519
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6517
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6515
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6513
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6511
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6509
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6507
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6505
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6498
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6496
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6494
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6492
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6486
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6485
drwx------ 2 ysys ysys 4096 Aug 11 03:30 6483

```

检查/tmp/ysys.log中日志信息

```
ysys      6480  7.0  0.2 108632  2436 pts/0    S+   03:30   0:00 initdb -E UNICODE -D /usr/local/pgsql/data
ysys      6486  0.0  1.9 266892 20128 pts/0    R+   03:30   0:00 /usr/local/pgsql/bin/postgres --boot -x1 -F
ysys      6497  0.0  0.1 106100  1184 pts/0    S+   03:30   0:00 sh -c "/usr/local/pgsql/bin/postgres" --single -F -O -c search_path=pg_catalog -c exit_on_error=true template1 >/dev/null
ysys      6498  0.0  2.4 271652 25068 pts/0    R+   03:30   0:00 /usr/local/pgsql/bin/postgres --single -F -O -c search_path=pg_catalog -c exit_on_error=true template1
ysys      6518  0.0  0.1 106100  1184 pts/0    S+   03:30   0:00 sh -c "/usr/local/pgsql/bin/postgres" --single -F -O -c search_path=pg_catalog -c exit_on_error=true template1 >/dev/null
ysys      6519 37.0  2.6 271020 27144 pts/0    R+   03:30   0:00 /usr/local/pgsql/bin/postgres --single -F -O -c search_path=pg_catalog -c exit_on_error=true template1
```

两个信息中命中有6486,6498，6519，一种可能是发生的太快，每隔1秒也记录不到进程信息，另外一种可能是gprof并没有记录到所有信息



开启并关闭数据库

```
[ysys@centos65 gprof]$ ls -lt
total 104
-rw------- 1 ysys ysys 8941 Aug 11 03:51 gmon.out
drwx------ 2 ysys ysys 4096 Aug 11 03:51 7790
drwx------ 2 ysys ysys 4096 Aug 11 03:51 7792
drwx------ 2 ysys ysys 4096 Aug 11 03:51 7794
drwx------ 2 ysys ysys 4096 Aug 11 03:51 7793
drwx------ 2 ysys ysys 4096 Aug 11 03:51 7795
drwx------ 2 ysys ysys 4096 Aug 11 03:33 7791

```

再次检查ysys.log

```
ysys      7790  0.0  1.8 268476 18708 pts/0    S    03:33   0:00 /usr/local/pgsql/bin/postgres
ysys      7792  0.0  0.7 268476  8064 ?        Ss   03:33   0:00 postgres: checkpointer process   
ysys      7793  0.0  0.7 268476  7776 ?        Ss   03:33   0:00 postgres: writer process     
ysys      7794  0.0  0.6 268476  6728 ?        Ss   03:33   0:00 postgres: wal writer process   
ysys      7795  0.0  0.7 268872  7408 ?        Ss   03:33   0:00 postgres: autovacuum launcher process   
ysys      7796  0.0  0.6 123620  6592 ?        Ss   03:33   0:00 postgres: stats collector process   

```

本次出现的进程号和目录只是相差1个，就是7791，那么这个7791又是什么？那么能否找到这个进程？



这个应该连续的进程号在本次实验中就是找不到，好吧，这个问题暂时先留着。



初步测试发现目录一层就是实际的pid



## gmon.out 内容



随机找到一个gmon.out

```
[ysys@centos65 gprof]$ gprof /usr/local/pgsql/bin/postgres gmon.out|less

Flat profile:

Each sample counts as 0.01 seconds.
 no time accumulated

  %   cumulative   self              self     total           
 time   seconds   seconds    calls  Ts/call  Ts/call  name    

 %         the percentage of the total running time of the
time       program used by this function.

cumulative a running sum of the number of seconds accounted
 seconds   for by this function and those listed above it.

 self      the number of seconds accounted for by this
seconds    function alone.  This is the major sort for this
           listing.

calls      the number of times this function was invoked, if
           this function is profiled, else blank.
 
 self      the average number of milliseconds spent in this
ms/call    function per call, if this function is profiled,
           else blank.

 total     the average number of milliseconds spent in this
ms/call    function and its descendents per call, if this 
           function is profiled, else blank.

name       the name of the function.  This is the minor sort
           for this listing. The index shows the location of
           the function in the gprof listing. If the index is
           in parenthesis it shows where it would appear in
           the gprof listing if it were to be printed.
^L
                     Call graph (explanation follows)

```



`gprof是GNU profiler工具。可以显示程序运行的“flat profile”，包括每个函数的调用次数，每个函数消耗的处理器时间。也可以显示“调用图”，包括函数的调用关系，每个函数调用花费了多少时间。还可以显示“注释的源代码”，是程序源代码的一个复本，标记有程序中每行代码的执行次数 `



## gprof 目录下文件是否可以删除

​	

​	gprof目录下的文件是postgresql安装启动关闭时所在进程的分析信息，理论上对于数据库来说，删除并不会影响数据库的一致性。



​	测试:在插入数据的同时，删除gprof下目录，重启数据库

```
tutorial=# insert into test1 select generate_series(1,1000),md5('guohui');
INSERT 0 1000

$ rm -rf gprof/*

$ pg_ctl stop -m fast
$ pg_ctl start -m fast

tutorial=# select * from test1;
  id  |               info               
------+----------------------------------
    1 | cfb599bba2e35793a620de1ecec0d09a
    2 | cfb599bba2e35793a620de1ecec0d09a
    3 | cfb599bba2e35793a620de1ecec0d09a
    4 | cfb599bba2e35793a620de1ecec0d09a
    5 | cfb599bba2e35793a620de1ecec0d09a
    6 | cfb599bba2e35793a620de1ecec0d09a
    7 | cfb599bba2e35793a620de1ecec0d09a
    8 | cfb599bba2e35793a620de1ecec0d09a

```



​	只要gprof目录只有gmon.out文件就可以被删除，不过删除文件前请再三确认删除命令和删除的文件。



​	注意：gprof目录最好不要删除，操作过程中可能会有一些报错，但是gprof目录下的子目录可以删除。









## 链接地址



https://www.postgresql.org/message-id/5CE0F779-5F83-4797-BD86-E310A2057C07%40gmail.com

https://github.com/digoal/blog/blob/master/201601/20160125_01.md

https://dba.stackexchange.com/questions/137122/profiling-postgres-with-gprof-no-data-gprof-folder-and-no-time-accumulated

http://www.postgresql-archive.org/Using-Gprof-with-Postgresql-td2074440.html

https://www.safaribooksonline.com/library/view/postgresql-96-high/9781784392970/5f0536d8-1b4e-4d50-9e8d-1d4d004bd2d0.xhtml

https://grokbase.com/p/postgresql/pgsql-performance/0998zgvg2q/using-gprof-with-postgresql

http://ipufirakefypif.tk/007e9f8b.html

https://blog.csdn.net/stanjiang2010/article/details/5655143

http://blog.sina.com.cn/s/blog_4a471ff601013vud.html

