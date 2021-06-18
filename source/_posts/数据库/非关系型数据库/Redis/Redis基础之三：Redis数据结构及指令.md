---
title: Redis基础之三：Redis数据结构及指令
date: 2020-11-10 15:48:11
tags:
  - 缓存
categories:
  - 数据库
  - 非关系型数据库
  - Redis
---

# 概述

Redis适用key-value方式存储，key统一为String类型，以下所述数据类型均为value的类型

数据类型分为：字符串类型、散列类型、列表类型、集合类型、有序集合类型。

![数据类型架构](Redis基础之三：Redis数据结构及指令/redis-data-structure-types.jpeg)



**五大数据结构对比**

| 类型                 | 简介                                                   | 特性                                                         | 场景                                                         |
| :------------------- | :----------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| String(字符串)       | 二进制安全，最基本类型                                 | 可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M | ---                                                          |
| Hash(字典)           | 键值对集合,即编程语言中的Map类型                       | 适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去) | 存储、读取、修改用户属性                                     |
| List(列表)           | 链表(双向链表)                                         | 增删快,提供了操作某一段元素的API                             | 1、最新消息排行等功能(比如朋友圈的时间线) <br />2、消息队列  |
| Set(集合)            | 哈希表实现,元素不重复                                  | 1、添加、删除,查找的复杂度都是O(1) <br />2、为集合提供了求交集、并集、差集等操作 | 1、共同好友 <br />2、利用唯一性,统计访问网站的所有独立ip <br />3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐 |
| Sorted Set(有序集合) | 将Set中的元素增加一个权重参数score,元素按score有序排列 | 数据插入集合时,已经进行天然排序                              | 1、排行榜<br />2、带权重的消息队列                           |



# Key指令

## 查看Key
`EXISTS key [key1...]`
		判断Key是否存在，返回存在的键个数，可跟多个Key值
`TYPE key`
		返回Key对应的Value类型，只能跟一个Key。如果对应的Key不存在，返回为none
`KEYS pattern`

## 删除/重命名Key
`DEL key [key1...]`
	删除Key，返回删除键的个数，可跟多个Key值。key中的内容也一并删除
`RENAME key newkey`
	将 key 改名为 newkey ，当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。

## 失效时间
`EXPIRE key seconds` / `PEXPIRE key milliseconds`
	为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。返回1成功，0设置失败或不存在Key
	只有key被DEL、 SET 、 GETSET 命令删除或覆写时才可移除生存时间，ALTER修改命令不行
	可对已经带有生存时间的 key 执行 EXPIRE 命令来取代旧的生存时间
`PERSIST key`
	取消Key的失效时间，取消成功则返回1，失败则返回0（Key本身就是永久的或者Key不存在）
`TTL key`  /  `PTTL key`
	查看Key的有效时间(秒)，只能跟一个Key。Key不存在返回-2；Key没有使用EXPIRE设置时间返回-1

# String

一个键最大能存储512M

## 指令

### 增
`SET Key Value`
		设置Key-Value
		如果 key 已经持有其他值， SET 就覆写旧值，无视类型
`MSET Key Value [Key1 value1]`
		一次性设置多个，key-value要成对出现

### 删

`DEL key`

### 改
`APPEND key`
	向字符串尾部追加字符串，追加的Key不存在，这个指令相当于SET指令。返回字符串长度
	如果 key 已经持有其他值， SET 就覆写旧值，无视类型

`INCR key`
		字符串形式的整数+1 ，如果Key不存在，进行了set操作

`DECR key`
		字符串形式的整数值-1，如果Key不存在，进行了set操作

`INCRBY key increment`
		key中的value增指定值

`DECRBY key decrement`
		key中的value减指定值

`INCRBYFLOAT key increment`
		增加浮点数，如果Key不存在返回-1

### 查
`GET Key`
		获取Key对应的Value

`MGET key [key1 key2]`
		一次性获取多个

`STRLEN Key`
	获取字符串长度，不存在的Key会返回0

## 适用场景

1. 简单的 KV 缓存
2. 参赛者又获得了十票
3. redis对于KV的操作效率很高，可以直接用作计数器。例如，统计在线人数等等，另外string类型是二进制存储安全的，所以也可以使用它来存储图片，甚至是视频等。

# Hash

键值对集合,即编程语言中的Map类型

## 指令

### 增
`HSET key field value`
	设置field-value
	**会覆盖**哈希表中已存在的域
	插入新的Key-Value返回的是1，如果是修改旧的Value，返回是0
    如果 key 不存在，一个空哈希表被创建并执行 `HMSET` 操作

`HMSET key field value [field1 value1]`
	设置多个field-value，一定要成对

`HSETNX key field value`
	在Map的Key不存在时执行HSET操作
	当Map的Key存在时，**不会覆盖**哈希表中已存在的域
	可以看做`HSET NOT EXISTS`的缩写

### 删
`HDEL key field [field1]`
    可删除多个field-value

### 改
`HSET key field value`
    已经存在于哈希表中，旧值将被覆盖

`HINCRBY key field num`
    域 field 增加num，会在对应的Key不存在时执行SET操作

### 查
`HGET Key field`
	获取Key中field对应的Value
	
`HMGET key [key1 key2]`
	一次性获取多个
	
`HGETALL key`
	获取key中所有field-value
	单行为Key，双行为Value
	这个指令是将Map遍历然后转换成List类型呈现
	
`HKEYS key`
	获取key中所有field
	
`HVALS key`
	返回 key 中所有域的值
	
`HLEN key`
	获取MAP的field-value对数

`HEXISTS key field`
	判断key中是否存在指定field

## 适用场景

存放键值对，一般可以用来存某个对象的基本属性信息，例如，用户信息，商品信息等，另外，由于hash的大小在小于配置的大小的时候使用的是ziplist结构，比较节约内存，所以针对大量的数据存储可以考虑使用hash来分段存储来达到压缩数据量，节约内存的目的，例如，对于大批量的商品对应的图片地址名称。比如：商品编码固定是10位，可以选取前7位做为hash的key,后三位作为field，图片地址作为value。这样每个hash表都不超过999个，只要把redis.conf中的hash-max-ziplist-entries改为1024，即可。

# List

链表(双向链表)

## 指令

### 增
`LPUSH key value [value1 value2]`
	将一个或多个值 value 插入到列表 key 的表头(最左边)

`RPUSH key value [value1 value2]`
	将一个或多个值 value 插入到列表 key 的表尾(最右边)

`LINSERT key BEFORE|AFTER target value`
	在target值前/后插入value值

`RPOPLPUSH source des`
	使用RPOP弹出source中的一个元素，然后使用LPUSH添加到des中

### 删
`LPOP key`
	移除并返回列表的头元素，key不存在返回nil

`RPOP key`
	移除并返回列表的尾元素，key不存在返回nil

`LREM key count value`
	返回值为删除元素的个数
	count=0删除全部，count>0删除表尾(右)count个，count<0删除表头(左)count个

### 改
`LSET key index newValue`
	设置索引值，调整value位置

`LTRIM key start end`
	保留指定片段，范围外的片段丢弃

### 查
`LRANGE key start stop`
	List的索引是从0开始的，支持负索引，-1代表最右边第一个元素，范围是包含关系[]

`LINDEX key index`
		获取索引位置的value




## 适用场景

1. 存储一些列表型的数据结构，类似粉丝列表、文章的评论列表之类的东西
2. 基于 list 实现分页查询，类似微博那种下拉不断分页的东西
3. 实现简单的消息队列

# Set

哈希表实现,元素不重复

## 指令

### 增

`SADD key value1 value2 value3....`
	不会插入重复元素

### 删

`SREM key value1 value2 value3...`
	删除某个值或者多个值

`SPOP key num`
	随机获取一个或者多个(num)集合中的元素
	获取的元素会从集合中移除

### 改

`SMOVE key1 key2 value`
	将value元素从key1集合移动到 key2集合
	原子性操作

### 查

`SMEMBERS key`
	获取全部值，以插入顺序无重复

`SISMEMBER key value`
	查看值是否存在

`SCARD key`
	获取集合中元素的个数

`SRANDMEMBER key num`
	获取的元素不会移除
	随机获取一个或者多个(num)集合中的元素

### 集合运算

#### 求差集

`SDIFF set1 set2 set3`
	返回差集

`SDIFFSTORE set1 set2 set3`
	将运算结果存放在第一个Set中，且第一个集合不会参与运算，并且如果第一个集合有元素会被清空

#### 求交集

`SINTER set1 set2 set3`
	返回交集

`SINTERSTORE set1 set2 set3`
	将运算结果存放在第一个Set中，且第一个集合不会参与运算，并且如果第一个集合有元素会被清空

#### 求并集

`SUNION set1 set2 set3`
	返回并集

`SUNIONSTORE set1 set2 set3`
	将运算结果存放在第一个Set中，且第一个集合不会参与运算，并且如果第一个集合有元素会被清空

## 适用场景

1. 基于 Redis 进行全局的 set 去重
2. 利用集合操作，查找共同好友
3. 适合用来给人、事物贴标签

# SortedSet

将Set中的元素增加一个权重参数score,元素按score有序排列

## 指令

### 增
`ZADD key score member [[score member] [score member] ...]`
	将一个或多个 member 元素及其 score 值加入到有序集 key 当中

### 删
`ZREM key member [member...]`
	删除有序集合中一个或多个成员
	
`ZREMRANGEBYLEX key min max`
	移除有序集合中给定的字典区间的所有成员
	
`ZREMRANGEBYRANK key start stop`
	移除有序集合中给定的排名区间的所有成员
	
`ZREMRANGEBYSCORE key min max`
	移除有序集合中给定的分数区间的所有成员

### 查
`ZCARD key`
​	获取有序集合中成员的个数

`​ZCOUNT key min max`
​	计算在有序集合中指定区间分数的成员数

`​ZRANK/ZREVRANK key member`
​	返回有序集 key 中成员 member 的排名。其中有序集成员按 score 值递增(从小到大/从大到小)顺序排列

`​ZSCORE key member`
​	返回有序集中，成员的分数值

`​ZRANGE key start stop`
​	通过索引区间返回有序集合指定区间内的成员

`​ZREVRANGE key start stop [WITHSCORES]`
​	返回有序集中指定区间内的成员，通过索引，分数从高到低

`​ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`
​	通过分数返回有序集合指定区间内的成员，min 和 max 可以是 -inf 和 +inf

`​ZREVRANGEBYSCORE key max min [WITHSCORES]`
​	返回有序集中指定分数区间内的成员，分数从高到低排序

`ZINCRBY key increment member`
	返回有序集合中指定成员的索引

### 集合运算

#### 求并集

`ZUNIONSTORE destination numkeys key [key ...]`
	计算给定的一个或多个有序集的并集，并存储在新的 key 中

#### 求交集

`ZINTERSTORE destination numkeys key [key...]`
	计算给定的一个或多个有序集的交集，其中给定 key 的数量必须以 numkeys 参数指定，并将该交集(结果集)储存到 destination

## 适用场景

适合做需要得到排名的场景需求，比如某场线上文章比赛，举办方通过谁的文章点赞更多，谁就能获得更高的排名

