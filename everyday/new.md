







```
20200301
20200302
20200303
20200304
20200305
20200306
20200307
20200308
20200309
20200310
20200311
20200312
20200313
20200314
20200315
20200316
20200317
20200318
20200319
20200320
20200321
20200322
20200323
20200324
20200325
20200326
20200327
20200328
20200329
20200330
20200331
```

##  第一版

```
#coding=utf-8
file_path = 'D:\\github\\ysys\\mirror\\202003_date.txt'
with open(file_path) as file_object:
	for line in file_object:
		print(str(line.rstrip()))
		value = str(line.rstrip())
		file=open("D:\\github\\ysys\\everyday\\2003\\"+value+"_01.md","w")
		file.close()

```



## 第二版

​	要求创建月份目录

​	要求创建日期文件

​	存在月份目录或者日期文件则跳过

```

```

