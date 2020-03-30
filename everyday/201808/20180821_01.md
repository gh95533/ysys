[TOC]

# pg 判断 字符 各种类型



​	同事给我打电话说了一下解决问题

![_](../img_src/000/2018-08-21_211145.png)

​	根据这些字段的值来判断如下一下条件

​	1、空值校验

​	2、长度校验(标准长度)

​	3、存在_数字

​	4、存在_特殊字符

​	5、存在_空格

​	6、非_数字

​	7、非_字母

​	8、非_中文

​	9、非_字母数字

​	10、RQSJ 判断是否符合date类型

​	11、身份证校验



​	这里面最难反而是身份证校验



## 各种分析判断	

​	空值校验

```
length(trim(column))<>0
```

​	长度校验()

```
length(column)=?
```

​	判断是否存在数字

```
tutorial=# select '过会1223'~'[0-9]';
 ?column? 
----------
 t
(1 row)

tutorial=# select '过会ABC'~'[0-9]';
 ?column? 
----------
 f
(1 row)
```

​	判断是否为特殊字符(不知道什么是特殊字符)

```
特殊字符指的是什么？找了网上资料使用如下字符：!@#$%^&*()-+=,.?

tutorial=# select '1234abc,dfa' ~ '[!@#$%^&*()-+=,.?]';
 ?column? 
----------
 t
(1 row)

tutorial=# select '1234abcdfa' ~ '[!@#$%^&*()-+=,.?]';
 ?column? 
----------
 f
(1 row)

```



​	判断是否空格

```
tutorial=# select '过会 1223'~ E'\\s+';
 ?column? 
----------
 t
(1 row)

tutorial=# select '过会1223'~ E'\\s+';
 ?column? 
----------
 f
(1 row)
```

​	



​	判断非数字

```
tutorial=# select '123过会123' ~ '^[0-9]*$';
 ?column? 
----------
 f
(1 row)

tutorial=# select '12123' ~ '^[0-9]*$';
 ?column? 
----------
 t
(1 row)

tutorial=# select '121,23' ~ '^[0-9]*$';
 ?column? 
----------
 f
(1 row)

```

​	判断非字母

```
tutorial=# select '123过会123' ~ '^[A-Za-z]*$';
 ?column? 
----------
 f
(1 row)

tutorial=# select '123cd123' ~ '^[A-Za-z]*$';
 ?column? 
----------
 f
(1 row)

tutorial=# select 'abcd123' ~ '^[A-Za-z]*$';
 ?column? 
----------
 f
(1 row)

tutorial=# select 'abcd' ~ '^[A-Za-z]*$';
 ?column? 
----------
 t
(1 row)

```



​	判断非中文

```
tutorial=# select '121过会23' ~ '^[\u4e00-\u9fa5]{0,}$';
 ?column? 
----------
 f
(1 row)

tutorial=# select '过会' ~ '^[\u4e00-\u9fa5]{0,}$';
 ?column? 
----------
 t
(1 row)

tutorial=# select '过会123' ~ '^[\u4e00-\u9fa5]{0,}$';
 ?column? 
----------
 f
(1 row)
```



​	判断非字母数字

```
tutorial=# select 'abcd' ~ '^[A-Za-z0-9]+$';
 ?column? 
----------
 t
(1 row)

tutorial=# select '1abc2d1' ~ '^[A-Za-z0-9]+$';
 ?column? 
----------
 t
(1 row)

tutorial=# select '1abc,2d1' ~ '^[A-Za-z0-9]+$';
 ?column? 
----------
 f
(1 row)

```



​	判断是否存在字母

```
tutorial=# select '过会1223adA'~'[A-Za-z]';
 ?column? 
----------
 t
(1 row)

tutorial=# select '过会1223'~'[A-Za-z]';
 ?column? 
----------
 f
(1 row)
```



 	初步判断身份证信息

```
select '370782199802061111' ~ '^((\d{18})|([0-9x]{18})|([0-9X]{18}))$';
 ?column? 
----------
 t
(1 row)
```

​	



	## 创建函数



```

create or replace function f_get_str(zd text,len int) returns varchar
as
$$
declare
v_zd text;
v_length int4;
v_len  int4;


v_boolean boolean;
v_fh varchar(1000);
begin

v_zd :=zd;

--zd 字段，len 长度

--空值校验
select length(trim(v_zd)) into v_length;
if v_length = 0 then
v_fh :='0';--0代表空值
else 
v_fh :='1';--1代表不是空值
end if;


--长度校验 

v_len :=len;

if v_length=v_len then
v_fh :=v_fh||'1';--1代表长度符合校验
else 
v_fh :=v_fh||'0';--0代表长度不符合校验
end if;

--存在数字
v_boolean :=false;
select v_zd ~ '[0-9]' into v_boolean ;

if v_boolean then

v_fh :=v_fh||'1';--1代表存在数字

else

v_fh :=v_fh||'0';--0代表不存在数字

end if;

--存在特殊字符
v_boolean :=false;

select v_zd ~ '[!@#$%^&*()-+=,.?]' into v_boolean ;



if v_boolean then

v_fh :=v_fh||'1';--1代表存在存在特殊字符

else

v_fh :=v_fh||'0';--0代表不存在特殊字符

end if;

--判断是否空格

v_boolean :=false;

select v_zd ~ E'\\s+' into v_boolean ;

if v_boolean then

v_fh :=v_fh||'1';--1代表存在空格

else

v_fh :=v_fh||'0';--0代表不存在空格

end if;


--判断非数字

v_boolean :=false;

select v_zd ~ '^[0-9]*$' into v_boolean ;

if v_boolean then

v_fh :=v_fh||'1';--1代表字段有只有数字

else

v_fh :=v_fh||'0';--0代表字段有非数字的值

end if;

--判断非中文

v_boolean :=false;

select v_zd ~ '^[\u4e00-\u9fa5]{0,}$' into v_boolean ;

if v_boolean then

v_fh :=v_fh||'1';--1代表字段有只有中文

else

v_fh :=v_fh||'0';--0代表字段有非中文的值

end if;


--判断非字母数字

v_boolean :=false;

select v_zd ~ '^[A-Za-z0-9]+$' into v_boolean ;

if v_boolean then

v_fh :=v_fh||'1';--1代表字段有只有字母数字

else

v_fh :=v_fh||'0';--0代表字段不是只有字母数字

end if;

--判断身份证号信息
--~ '^((\d{18})|([0-9x]{18})|([0-9X]{18}))$'


select v_zd ~ '^((\d{18})|([0-9x]{18})|([0-9X]{18}))$' into v_boolean ;

if v_boolean then

v_fh :=v_fh||'1';--1代表字段符合身份证信息

else

v_fh :=v_fh||'0';--0代表字段不符合身份证信息

end if;

return v_fh;
end;
$$ language plpgsql;
```







## 链接地址

https://blog.csdn.net/NextAction/article/details/80520606

https://blog.csdn.net/wugewuge/article/details/7704996

