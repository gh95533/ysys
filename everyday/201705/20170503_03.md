[TOC]

# linux sh get ftp  filename



linux get ftp filename

由于“想哭”病毒在windows上面造成的危害，现在将ftp服务器迁移到linux，与之相关的数据处理作业也要放到linux服务器，现在使用kettle作为etl工具，那么只需要将ftp下的文件信息下载到某个本地文件中，就可以直接的使用kettle的ftp下载控件来处理文件

get ftp filename

[[root@centos65](mailto:root@centos65) opt]# cat getftp.sh 

\#!/bin/bash

\# get file from ftp

ftp -niv  << EOF

open 192.168.1.201

user shengting 1234567812

ascii

prompt off

dir /opt/vsftp/shengting  /home/good.txt

close

bye

EOF

这样就可以获得文件名和文件的时间，初步可以判定文件是否为新的产生或者被更新过,请注意如果下载中文件名为带中文的，可能会出现乱码的情况