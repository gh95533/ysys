[TOC]

# pg uuid

## 背景

​	今天下午在插入数据时需要对某个字段插入uuid数据，暂时通过25位'0'和7为从‘1000000’开始的序列暂时解决该问题，那么如果必须使用uuid，如何操作uuid

## UUID 类型

​	数据类型uuid存储由RFC 4122,ISO/IEC 9834-8:2005以及相关标准定义的通用唯一标识符(UUID),(某些系统将这种数据类型引用为全局唯一标识符GUID)。这种标识符是一个128位的量，它由一个精心选择的算法产生，该算法能保证在已知空间中任何其他使用相同算法的人能够产生同一个标识符的可能行非常宵，因此，对于分布式系统，这些标识符相比序列生成器而言提供了一种很好的唯一性保障，序列生成器只能在一个数据库中保证唯一

​	一个UUID被写成一个小写十六进制的序列，该序列被连字符分隔成多个组：首先是一个8位组，接下来是三个4位组，最后是一个12位组。总共的32位表示了128个二进制位。一个标准形式的UUID类似于:

```
a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11
```

​	postgresql接受另外一种输入方式:使用大写位，标准格式被花括号包围，忽略某些或者全部连字符，在任意4位组后面增加一个连字符。

```
A0EEBC99-9C0B-4EF8-BB6D-6BB9BD380A11
{a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11}
a0eebc999c0b4ef8bb6d6bb9bd380a11
a0ee-bc99-9c0b-4ef8-bb6d-6bb9-bd38-0a11
{a0eebc99-9c0b4ef8-bb6d6bb9-bd380a11}
```

​	输出总是采用标准形式

​	postgresql为UUID提供了存储和比较函数，但是核心数据库不包含任何用于产生UUID的函数，因为没有一个单一算法能够很好的适应每一个应用。uuid-ossp模块提供了实现一些标准算法的函数，pgcrypto模块也为随机UUID提供了一个生成函数。此外，UUID可以由客户端应用产生，或者由通过服务器函数调用的其他库生成。



## uuid-ossp

​	uuid-ossp模块提供函数使用几种标准算法之一产生通用唯一标识符(UUID)。还提供产生某些特殊UUID常量的函数算法，分别用版本好1,3,4,5（没有版本2的算法)。这些算法中的每一个版本都适合于不同的应用集合



### uuid-ossp 函数

​	下表展示了可用来产生UUID的函数。相关标准ITU-T Rec.X.667,ISO/IEC9834-8:2005 以及RFC 4122指定了四种用于产生UUID的

用于UUID产生的函数



#### 用于UUID产生的函数

| 函数                                          | 描述                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| `uuid_generate_v1()`                          | 这个函数产生一个版本 1 的 UUID。这涉及到计算机的 MAC 地址和一个时间戳。注意这种 UUID 会泄露产生该标识符的计算机标识以及产生的时间，因此它不适合某些对安全性很敏感的应用。 |
| `uuid_generate_v1mc()`                        | 这个函数产生一个版本 1 的 UUID，但是使用一个随机广播 MAC 地址而不是该计算机真实的 MAC 地址。 |
| `uuid_generate_v3(namespace uuid, name text)` | 这个函数使用指定的输入名称在给定的名字空间中产生一个版本 3 的 UUID。该名字空间应该是由`uuid_ns_*()`函数（如[表 F-34](http://www.postgres.cn/docs/9.6/uuid-ossp.html#UUID-OSSP-CONSTANTS)所示）产生的特殊常量之一（理论上它可以是任意 UUID）。名称是选择的名字空间中的一个标识符。例如：`SELECT uuid_generate_v3(uuid_ns_url(), 'http://www.postgresql.org');`名称参数将使用 MD5 进行哈希，因此从产生的 UUID 中得不到明文。采用这种方法的 UUID 生成没有随机性并且不涉及依赖于环境的元素，因此是可以重现的。 |
| `uuid_generate_v4()`                          | 这个函数产生一个版本 4 的 UUID，它完全从随机数产生。         |
| `uuid_generate_v5(namespace uuid, name text)` | 这个函数产生一个版本 5 的 UUID，它和版本 3 的 UUID 相似，但是采用的是 SHA-1 作为哈希方法。版本 5 比版本 3 更好，因为 SHA-1 被认为比 MD5 更安全。 |

#### 返回UUID常量的函数



| `uuid_nil()`     | 一个"nil" UUID 常量，它不作为一个真正的 UUID 发生。          |
| ---------------- | ------------------------------------------------------------ |
| `uuid_ns_dns()`  | 为 UUID 指定 DNS 名字空间的常量。                            |
| `uuid_ns_url()`  | 为 UUID 指定 URL 名字空间的常量。                            |
| `uuid_ns_oid()`  | 为 UUID 指定 ISO 对象标识符（OID） 名字空间的常量（这属于 ASN.1 OID，它与PostgreSQL使用的 OID 无关）。 |
| `uuid_ns_x500()` | 为 UUID 指定 X.500 可识别名（DN）名字空间的常量。Constant designating the X.500 distinguished name (DN) namespace for UUIDs. |



### 编译uuid-ossp

​	在历史上这个模块依赖于OSSP UUID库，这也是这个模块名称的由来。虽然现在还能在<http://www.ossp.org/pkg/lib/uuid/> 上找到OSSP UUID库，但是它已经不再被维护并且越来越难以被一直到新的平台。uuid-ossp现在在一些平台上可以脱离OSSP库被编译。在FreeBSD,NetBSD和一些其他源自BSD的平台上，在核心的libc库中已经包含了合适的UUID创建函数。在Linux,OS X和一些其他平台上，libuuid 库中提供了合适的函数，它最初是来自与e2fsprogs项目(不过在现在Linux上它被认为是util-linux-ng的一部分)。在调用configure时，指定--with-uuid=bsd可使用BSD的函数，指定--with-uuid=e2fs会使用e2fsprog的libuuid,指定--with-uuid=ossp则会使用ossp uuid库。在一台特定的机器上可能会存在多种上述的库，因此configure不会自动选择其中一个

​	

