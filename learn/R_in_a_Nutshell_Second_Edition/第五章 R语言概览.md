[TOC]

# R05 R语言概览

**document support**

ysys

**date**

2019-10-29

**label**

R,learn



## 概要

​	学习一门语言就必须学习句法和语法，本章将给出一个R语言的概览，帮助理解并且编写R代码。



## 表达式

​	R代码是由一系列表达式组成的。R中的表达式的例子包括赋值语句，条件语句和算术表达式。

```
> x <- 1
> if (1 > 2) "yes" else "no"
[1] "no"
> 127%%10
[1] 7
```

​	表达式由对象和函数组成的，你可以用换行符或者分号来分割表达式。比如，下面就是一些列有分号分割的表达式：

```
> "this expression will be printed";7+3;exp(0+1i*pi)
[1] "this expression will be printed"
[1] 10
[1] -1+0i
```



## 对象

​	所有的R代码都用于操作对象(object),对象就是计算机世界里的'个体'。R中的对象包括数值型向量，字符型向量，列表，函数等。

```
> c(1,2,3,4,5)
[1] 1 2 3 4 5
> "this is an object too"
[1] "this is an object too"
> list(c(1,2,3,4,5),"this is an object too","this wohoel thing is a list")
[[1]]
[1] 1 2 3 4 5

[[2]]
[1] "this is an object too"

[[3]]
[1] "this wohoel thing is a list"

> function(x,y){x+y}
function(x,y){x+y}
```



## 符号

​	R中变量的名字成为符号(symbol).将一个对象赋给变量名时，其实是将该对象赋给当前环境里的一个符号。

```
> x <- 1
```



## 函数

​	函数是R中一个特殊的对象，它接受一些输入对象,返回一个输出对象。

```
> animals <- c("cow","chicken","pig","tuba")
> animals[4] <- "duck"
> animals
[1] "cow"     "chicken" "pig"     "duck"   
```



## 在赋值语句中，对象会被赋值

​	在赋值操作中，大部分对象是不可变的。

```
> u <- list(1)
> v <- u
> u[[1]]<-"hat"
> u
[[1]]
[1] "hat"

> v
[[1]]
[1] 1
```

​	这一点适用于R中大多数原始对象





## R中一切皆为对象



## 特殊值



### NA

not available 缺失值



### Inf和-Inf



### NaN



### NULL



### 强制转换



## R解释器



## 观察R是如何工作的

​	R解释器代码

```
> if (x > 1) "orage" else "apple"
[1] "apple"
```

​	利用quote()函数演示上述表达式的解析过程

```
> typeof(quote(if (x > 1) "orage" else "apple"))
[1] "language"
```

​	不幸的是，针对语言对象的print函数返回的信息量极其有限

```
> quote(if (x > 1) "orage" else "apple")
if (x > 1) "orage" else "apple"
```

​	将语言对象转换为列表

```
> as(quote(if (x > 1) "orage" else "apple"),"list")
[[1]]
`if`

[[2]]
x > 1

[[3]]
[1] "orage"

[[4]]
[1] "apple"

```

​	同样可以将typeof函数应用到列表中的所有元素上。

```
> lapply(as(quote(if (x > 1) "orage" else "apple"),"list"),typeof)
[[1]]
[1] "symbol"

[[2]]
[1] "language"

[[3]]
[1] "character"

[[4]]
[1] "character"

```

​	







