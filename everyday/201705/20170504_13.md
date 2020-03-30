[TOC]

# kettle more ftp Single-Level Directory new files download

​      

首先本篇文章使用的场景是，单个目录下每天可能生成一个或者多个文件，需要下载

要求如下：

​     1、ftp需要使用用户名密码登录

​     kettle的ftp控件需要使用用户名密码登录(匿名登录方式本人测试时不行)

​     2、尽量要求目录名及文件名都是英文，或者首字母简拼方式的文件

​     3、文件类型要求是txt等类型，不要是excel这种文件

​     4、文件的间隔符要求是有且能够区别，如使用“，”，“；”等

​     

上面的要求3，4其实并不是本次需要的，但是如果用户和你商量需要对方提供或者被提供时，尽量提出该些要求

 假设如下

​      1、已经获取了ftp上的文件列表并且解析入库

​      2、最新的文件列表在数据库已经标注出来

环境如下

1、数据库oracle11g

2、kettle5.2(windows2008)

操作如下

​      1、在创建一个ftp配置表，模本如下

​      

​      create table t_ftp_config

​      （bzw varchar2(100),--后期区分各个ftp来源

​          ftpip   varchar2(100),--ftpip地址

​          ftpuser varchar2(100),--ftp用户

​          remotedir varchar2(100),--ftp远程目录

​          ftpport varchar2(100),--ftp端口值

​          downloaddir varchar2(100)--下载到本地路径);

        

​       2、提前需要拷贝的sql语句

    将需要下载的文件与之关联，一般可以通过路径名来作为关联条件

​           select ftpip,ftpuser,ftppass,remotedir,ftpport,doanloaddir,filename from t_ftp_config t , ftp文件解析表 where downloaddir=path and bzw=${BZW}

              

   ftp文件解析表就是ftp需要下载的文件名，路径名；BZW就是标志位，BZW的作用就是区分不同的作业，需要我们来传参

​            这是本次唯一需要配置的sql语句，只需要配置到表输入出就可以了

            

​       3、开启kettle配置

​             该方案中有三个job,三个trans，如果可以更优化的的话，请联系告知我(gh_ysys@126.com)

    

[CENTER_FTP_COPY_ROWS_TO_RESULT.ktr](../img_src/43d7de9be1ad4e08bdec6bcce963547d/CENTER_FTP_COPY_ROWS_TO_RESULT.ktr)

[CENTER_FTP_RESULT_VARIABLE_SYSTEM_DOWNLOAD.kjb](../img_src/51670e6733eb4753b214c8fce3875583/CENTER_FTP_RESULT_VARIABLE_SYSTEM_DOWNLOAD.kjb)

[CENTER_FTP_SET_VARIABLE_NO_REMOTEDIR.ktr](../img_src/64a4c51118b04dca90a9df1bf6bf61d0/CENTER_FTP_SET_VARIABLE_NO_REMOTEDIR.ktr)

[CENTER_FTP_STSTEN_DOWNLOAD.kjb](../img_src/44db8931efeb4aeabc24c8027c1012b0/CENTER_FTP_STSTEN_DOWNLOAD.kjb)

[CENTER_FTP_VARIABLE_SYSTEM_DOWNLOAD.kjb](../img_src/bf2136ef9e7f405db0f62cacf6f8d2b9/CENTER_FTP_VARIABLE_SYSTEM_DOWNLOAD.kjb)

[CENTER_SYSTEM_INFO.ktr](../img_src/d75e2e673719405389547266f819a22c/CENTER_SYSTEM_INFO.ktr)

        

                

​      4、拷贝完之后，重新创建一个job,右击设置中选择参数，添加parameter=BZW,DEFAULT VALUE='BZW的值'，就可以调度所有的ftp下载，调用的job是center_ftp_result_variable_system_download.kjb

注意事项：

1、如果ftp中文件路径不是根路径，需要添加remotedir，如果是根路径，那么就不需要添加remotedir,如果在整体方案中使用变量remotedir，如果是根路径的话，就是报错，建议拷贝之前的作业重新生成一个带有remotedir,如果有remotedir，就走带有remotedir的方案；没有remotedir，就走没有带有remotedir的方案