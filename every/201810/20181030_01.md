[TOC]

# gpload delimiter character choose

**文档整理**

ysys

**日期**

2018-10-30

**标签**

greenplum,gpload,delimiter character



## 背景

​	最近一段时间需要将数据从Oracle同步到Gp数据库中，最好的同步手段是通过gpload加载csv文件到Gp数据库中，但是发现单个字符没有办法总是会将某些数据拦腰斩断，导致数据无法入库，后面发现gpload只能使用一个分割符分割，那么如何选择分隔符，选择什么分隔符比较好，不会造成太大数据遗留，选择什么分割符，可能让数据完美入库。



## 单个字符



​	之前单个字符经常导致出现‘extra data ...’,就选用了两个字符，后面执行时报错

```
A gpload control file processing warning occurred. A delimiter must be single ASCII charactor, you can also use unprintable characters(for example: '\x1c' / E'\x1c' or '\u001c' / E'\u001c'

A gpload control file processing error occurred. Invalid delimiter, gpload quit immediately
```



```
Optional. Specifies a single ASCII character that separates columns within each row (line) of data. The default is a tab character in TEXT mode, a comma in CSV mode. You can also specify a non- printable ASCII character or a non-printable unicode character, for example: "\x1B" or "\u001B". The escape string syntax, E'character-code', is also supported for non-printable characters. The ASCII or unicode character must be enclosed in single quotes. For example: E'\x1B' or E'\u001B'.
```

​	网上的文档解释可以使用`E'\x1B'`使用，那么是否可以根据数据考虑一下可能性，我觉得可能的基础可以使用特殊的字符也是可以替代。

​		







## 链接地址

http://ascii.911cha.com/

https://www.cnblogs.com/zzsdream/p/5802254.html

https://kimnote.com/tools/ascii/

https://github.com/Qihoo360/gpstall/blob/master/script/gpload.py

https://stackoverflow.com/questions/28568747/using-ascii-31-field-separator-character-as-postgresql-copy-delimiter

https://gpdb.docs.pivotal.io/500/utility_guide/admin_utilities/gpload.html

