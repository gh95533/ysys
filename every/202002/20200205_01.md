[TOC]

# 第7章 R对象



## 概况

​	R提供了一组面向对象编程的机制，本章重点讲述R中的内嵌对象，以及如何使用这些内嵌对象。



## 基本数据类型

​	基础向量

​	复合对象

​	特殊对象

​	R语言

​	函数

​	内置对象

​	字节码对象



## 向量

​	最简单就是使用函数c.

```
> v <- c(.295,.300,.250,.287,.215)
> v
[1] 0.295 0.300 0.250 0.287 0.215
```

​	c函数会将所有的参数类型强制转为同一类型

```
> v <- c(.295,.300,.250,.287,.215,"zilch")
> v
[1] "0.295" "0.3"   "0.25"  "0.287" "0.215" "zilch"
```

​	设定recursive=TRUE时,c函数可以从其他数据结构中递归形成向量

```
> v <- c(.295,.300,.250,.287,.215,list(.102,.200,.303),recursive=TRUE)
> v
[1] 0.295 0.300 0.250 0.287 0.215 0.102 0.200 0.303
> typeof(v)
[1] "double"
```

​	当没有设置时

```
> v <- c(.295,.300,.250,.287,.215,list(.102,.200,.303))
> v
[[1]]
[1] 0.295

[[2]]
[1] 0.3

[[3]]
[1] 0.25

[[4]]
[1] 0.287

[[5]]
[1] 0.215

[[6]]
[1] 0.102

[[7]]
[1] 0.2

[[8]]
[1] 0.303

> typeof(v)
[1] "list"
> class(v)
[1] "list"

```

​	另外一种生成向量的工具是操作符号":"

```
> 1:20
 [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19
[20] 20
```

​	更加灵活的函数是seq

```
> seq(from=5,to=25,by=5)
[1]  5 10 15 20 25
```

​	可以通过向量的长度属性来调整向量的长度

```
> w <- 1:10
> length(w) <- 5
> w
[1] 1 2 3 4 5
> length(w) <- 10
> w
 [1]  1  2  3  4  5 NA NA NA NA NA
```

​	



## 列表



```
> l <- list(1,2,3,4,5)
> l
[[1]]
[1] 1

[[2]]
[1] 2

[[3]]
[1] 3

[[4]]
[1] 4

[[5]]
[1] 5

> 
> parcel <- list(destination="New York",dimensions=c(2,6,9),price=12.95)
> parcel$price
[1] 12.95
```





## 其他对象



### 矩阵

```
> m <- matrix(data=1:12,nrow=4,ncol=3,dimnames = list(c("r1","r2","r3","r4"),c("c1","c2","c3")))
> m
   c1 c2 c3
r1  1  5  9
r2  2  6 10
r3  3  7 11
r4  4  8 12
```



### 数组

```
> a <- array(data=1:24,dim=c(3,4,2))
> a
, , 1

     [,1] [,2] [,3] [,4]
[1,]    1    4    7   10
[2,]    2    5    8   11
[3,]    3    6    9   12

, , 2

     [,1] [,2] [,3] [,4]
[1,]   13   16   19   22
[2,]   14   17   20   23
[3,]   15   18   21   24

```

### 因子

​	分析数据时经常遇到分类变量。

​	如下，brown重复多个，我只想要一个因子

```
> eye.colors <- c("brown","blue","blue","green","brown","brown","brown")
> eye.colors
[1] "brown" "blue"  "blue"  "green" "brown" "brown" "brown"
```

​	添加factor

```
> eye.colors <- factor(c("brown","blue","blue","green","brown","brown","brown"))
> levels(eye.colors)
[1] "blue"  "brown" "green"
> eye.colors
[1] brown blue  blue  green brown brown brown
Levels: blue brown green
```

​	可以按照有序因子时显示因子水平的顺序

```
> survey.results <- factor(c("Disagree","Neutral","Strongly Disagree","Neutral","Agree","Strongly Agree","Disagree","Strongly Agree","Neutral","Agree"),levels=c("Strongly Disagree","Disagree","Neutral","Agree","Strongly Agree"),ordered=TRUE)
> survey.results
 [1] Disagree          Neutral           Strongly Disagree Neutral           Agree             Strongly Agree   
 [7] Disagree          Strongly Agree    Neutral           Agree            
Levels: Strongly Disagree < Disagree < Neutral < Agree < Strongly Agree
```

​	有的懵



​	内部使用整型数据

```
> class(eye.colors)
[1] "factor"
> eye.colors.integer.vector <- unclass(eye.colors)
> eye.colors.integer.vector
[1] 2 1 1 3 2 2 2
attr(,"levels")
[1] "blue"  "brown" "green"
> class(eye.colors.integer.vector)
[1] "integer"
> 
> class(eye.colors.integer.vector) <- "factor"
> eye.colors.integer.vector
[1] brown blue  blue  green brown brown brown
Levels: blue brown green
> class(eye.colors.integer.vector)
[1] "factor"
```



### 数据框

​	数据框要求每一行的长度一致

```
> data.frame(a=c(1,2,3,4,5),b=c(1,2,3,4))
Error in data.frame(a = c(1, 2, 3, 4, 5), b = c(1, 2, 3, 4)) : 
  参数值意味着不同的行数: 5, 4
```

​	示例

```
> top.bacom.searching.cities <- data.frame(city=c("BeiJing","ShangHai","QingDao"),rank = c(1,2,3))
> top.bacom.searching.cities
      city rank
1  BeiJing    1
2 ShangHai    2
3  QingDao    3
> typeof(top.bacom.searching.cities)
[1] "list"
> class(top.bacom.searching.cities)
[1] "data.frame"
```





### 公式

​	经常需要表述变量之间的关系。绘制两个变量之间的关系图来展示变量之间的关系

```
> sample.formula <- as.formula(y~x1+x2+x3)
> class(sample.formula)
[1] "formula"
> typeof(sample.formula)
[1] "language"
```

​	其他请参考书本



### 时间序列

​	略



### Shingle 对象

​	Shingle 对象是因子对象的连续性泛化



### 日期和时间对象

```
> data.I.started.writing <- as.Date("2020/3/12","%Y/%m/%d")
> data.I.started.writing
[1] "2020-03-12"
> today <- Sys.Date()
> today
[1] "2020-03-12"
> today - data.I.started.writing
Time difference of 0 days
```





###  连接对象

​	略



## 属性



常见属性

| 属性      | 描述                       |
| --------- | -------------------------- |
| class     | 对象的类                   |
| comment   | 对象的注解                 |
| dim       | 对象的维度                 |
| dimnames  | 与对象的每个维度相关的名字 |
| names     | 返回对象的名字属性         |
| row.names | 对象的列名                 |
| tsp       | 对象的起始点               |
| level     | 因子型变量的水平           |



```
> m <- matrix(data=1:12,nrow=4,ncol=3,dimnames = list(c("r1","r2","r3","r4"),c("c1","c2","c3")))
> m
   c1 c2 c3
r1  1  5  9
r2  2  6 10
r3  3  7 11
r4  4  8 12
> attributes(m)
$dim
[1] 4 3

$dimnames
$dimnames[[1]]
[1] "r1" "r2" "r3" "r4"

$dimnames[[2]]
[1] "c1" "c2" "c3"
```

```
> colnames(m)
[1] "c1" "c2" "c3"
> rownames(m)
[1] "r1" "r2" "r3" "r4"
```

​	可以通过简单改变属性将矩阵转换为其他类的对象

```
> dim(m) <- NULL
> m
 [1]  1  2  3  4  5  6  7  8  9 10 11 12
> class(m)
[1] "integer"
> typeof(m)
[1] "integer"
```

​	

​	分别定义一个二维数组和一个相同对象的向量

```
> a <- array(data=1:12,dim=c(3,4))
> a
     [,1] [,2] [,3] [,4]
[1,]    1    4    7   10
[2,]    2    5    8   11
[3,]    3    6    9   12
> b <- 1:12
```

​	使用`==`比较一下两个结果

```
> a == b
     [,1] [,2] [,3] [,4]
[1,] TRUE TRUE TRUE TRUE
[2,] TRUE TRUE TRUE TRUE
[3,] TRUE TRUE TRUE TRUE
```

​	可以使用all.equal函数比较两个对象的数据和维度以甄别两个对象是否近乎相同

```
> all.equal(a,b)
[1] "Attributes: < Modes: list, NULL >"                    "Attributes: < Lengths: 1, 0 >"                       
[3] "Attributes: < names for target but not for current >" "Attributes: < current is not list-like >"            
[5] "target is matrix, current is numeric"   
```

​	检查两个对象是否一致,而不关心不一致的原因

```
> identical(a,b)
[1] FALSE
```

​	对b赋予一个维度属性

```
> dim(b) <- c(3,4)
> identical(a,b)
[1] TRUE
```



### 类

​	typeof 函数，class函数

​	





## 全部脚本

```
v <- c(.295,.300,.250,.287,.215)
v


v <- c(.295,.300,.250,.287,.215,"zilch")
v

v <- c(.295,.300,.250,.287,.215,list(.102,.200,.303),recursive=TRUE)
v
typeof(v)

v <- c(.295,.300,.250,.287,.215,list(.102,.200,.303))
v
typeof(v)
class(v)


1:20
seq(from=5,to=25,by=5)


w <- 1:10
length(w) <- 5
w
length(w) <- 10
w



l <- list(1,2,3,4,5)
l

parcel <- list(destination="New York",dimensions=c(2,6,9),price=12.95)
parcel$price


m <- matrix(data=1:12,nrow=4,ncol=3,dimnames = list(c("r1","r2","r3","r4"),c("c1","c2","c3")))
m


a <- array(data=1:24,dim=c(3,4,2))
a


eye.colors <- c("brown","blue","blue","green","brown","brown","brown")
eye.colors
eye.colors <- factor(c("brown","blue","blue","green","brown","brown","brown"))
levels(eye.colors)
eye.colors


survey.results <- factor(c("Disagree","Neutral","Strongly Disagree","Neutral","Agree","Strongly Agree","Disagree","Strongly Agree","Neutral","Agree"),levels=c("Strongly Disagree","Disagree","Neutral","Agree","Strongly Agree"),ordered=TRUE)
survey.results


class(eye.colors)
eye.colors.integer.vector <- unclass(eye.colors)
eye.colors.integer.vector
class(eye.colors.integer.vector)

class(eye.colors.integer.vector) <- "factor"
eye.colors.integer.vector
class(eye.colors.integer.vector)


data.frame(a=c(1,2,3,4,5),b=c(1,2,3,4))

top.bacom.searching.cities <- data.frame(city=c("BeiJing","ShangHai","QingDao"),rank = c(1,2,3))
top.bacom.searching.cities
typeof(top.bacom.searching.cities)
class(top.bacom.searching.cities)


sample.formula <- as.formula(y~x1+x2+x3)
class(sample.formula)
typeof(sample.formula)





data.I.started.writing <- as.Date("2020/3/12","%Y/%m/%d")
data.I.started.writing
today <- Sys.Date()
today
today - data.I.started.writing


m <- matrix(data=1:12,nrow=4,ncol=3,dimnames = list(c("r1","r2","r3","r4"),c("c1","c2","c3")))
m
attributes(m)
dim(m)
dimnames(m)
colnames(m)
rownames(m)
dim(m) <- NULL
m
class(m)
typeof(m)



a <- array(data=1:12,dim=c(3,4))
a
b <- 1:12
a == b

all.equal(a,b)
identical(a,b)

dim(b) <- c(3,4)
identical(a,b)






```



​	



