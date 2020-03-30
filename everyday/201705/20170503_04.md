[TOC]

# WINDOW bat 获取ftp文件信息



window get ftp filename

近期整理kettle抽取ftp服务器文件时，整理一下如何从ftp上将文件名等信息写到本地文件，通过kettle后续处理，将没有下载或者更新的文件下载到本地，处理数据

1、将ftp文件信息写入到本地文件中

bat文件内容

@echo off

set h=xxx

set u=xxxx

set p=xxxx123

echo open %h%>ftp.txt

echo %u%>>ftp.txt

echo %p%>>ftp.txt

echo dir>>ftp.txt

echo bye>>ftp.txt

ftp -s:ftp.txt>e:\ftpdir.txt

echo open %h%>ftp.txt

echo %u%>>ftp.txt

echo %p%>>ftp.txt

for /f "tokens=4" %%i in ('findstr "<DIR>" ftpdir.txt') do (

echo cd %%~i>>ftp.txt

echo dir>>ftp.txt

echo cd ..>>ftp.txt)

echo bye>>e:\ftp.txt

ftp -s:ftp.txt>e:\ftpfile.txt

遇到某个ftp登陆为匿名登陆，而且本次情况只需要下载当前日期的文件,kettleftp控件无法匿名登陆到ftp,因此使用bat脚本直接将文本文件下载到本地上

bat模板

@echo off

set yw=%date:~0,4%-%date:~5,2%-%date:~8,2%

echo cd /载/载 >>goo.txt 

echo get  %yw%_数据.zip >>goo.txt 

echo bye >>goo.txt 

ftp -A -s:goo.txt  ip_address

ftp -s:goo.txt

del goo.txt