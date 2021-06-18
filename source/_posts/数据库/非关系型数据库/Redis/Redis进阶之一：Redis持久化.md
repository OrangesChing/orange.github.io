---
title: Redis进阶之一：Redis持久化
date: 2020-08-19 15:48:11
tags:
  - 缓存
  - 面试知识点
categories:
  - 数据库
  - 非关系型数据库
  - Redis
---

RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘。默认的持久化方式为RDB，这种方式就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb。

Reddis的所有的配置都是在redis.conf文件中，其中包含RDB和AOF两种持久化机制的各种配置。

# Redis DataBase(RDB)

RDB就是把数据以**快照**的形式保存在磁盘上，即像对数据拍照一样，把某一时刻的数据保存到磁盘上。

对于RDB来说需讨论的关键问题是：什么时候拍照，拍照的时候Redis是否正常工作

## 触发机制

RDB提供了三种机制：`save`、`bgsave`、`自动触发`

### 手动触发

#### save命令（已废弃）

在主线程中保存快照。会**阻塞当前Redis服务器**，直到保存完毕

**优点**是不会消耗额外的内存空间

**缺点**是对于内存大的实例会造成长时间的阻塞，线上环境不建议使用

具体流程如下：

![img](Redis进阶之一：Redis持久化/e7cd7b899e510fb3aa8c05042b22c093d0430ca7.jpeg)

#### bgsave命令

Redis进程执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。

堵塞只发生在fork阶段，一般时间很短。

BGSAVE命令是针对SAVE堵塞问题做的优化。因此Redis内部所有的设计RDB的操作都采用BGSAVE的方式，而save命令已经废弃。

执行该命令时，Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。具体流程如下：

![img](Redis进阶之一：Redis持久化/023b5bb5c9ea15cefb035bc8431132f53b87b21e.jpeg)

### 自动触发

自动触发是由配置文件配置完成

#### save n m

自动触发最常见的情况是在配置文件中通过`save m n`，指定当`m`秒内发生`n`次变化时，会自动触发`bgsave`

`redis.conf`配置文件中可配置

默认配置如下：

```conf
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behaviour will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""
# 不需要持久化，可以注释掉所有的 save 行来停用功能
save 900 1      # 900秒内如果至少有1个key的值变化
save 300 10     # 300 秒内如果至少有10个key的值变化
save 60 10000   # 60 秒内如果至少有 10000 个 key 的值变化
```

其他配置

- stop-writes-on-bgsave-error：默认值为yes。启用了RDB且最后一次后台保存数据失败时，Redis是否停止接收数据。这会让用户意识到数据没有正确持久化到磁盘上，否则没有人会注意到灾难发生了

- rdbcompression：默认值是yes。对于存储到磁盘中的快照，可以设置是否进行压缩存储。

- rdbchecksum：默认值是yes。在存储快照后，是否使用使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。

- dbfilename：设置快照的文件名，默认是 dump.rdb

- dir：设置快照文件的存放路径

```
# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# For default that's set to 'yes' as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
rdbcompression yes  # 是否使用使用CRC64算法来进行数据校验

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
rdbchecksum yes

# The filename where to dump the DB
dbfilename dump.rdb   # 设置快照的文件名

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./
```



#### 其他

除了save m n以外，还有一些其他情况会触发bgsave：

1. 如果从节点执行`全量复制操作`，主节点自动执行`BGSAVE`生成`RDB`文件并发送给从节点。
2. 执行`debug reload`命令重新加载`Redis`时，也会自动触发`save`操作。
3. 默认情况下执行`shutdown`命令时，如果没有开启`AOF持久化功能`则自动执行`BGSAVE`。



## RDB的优劣

**优势**

- RDB是一次全量备份，存储的是内存数据的二进制序列化形式，存储上非常紧凑，非常适合用于进行备份和灾难恢复

- 生成RDB文件的时候，redis主进程会`fork()`一个子进程来处理所有保存工作，主进程不需要进行任何磁盘`IO`操作

- RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快

**劣势**

父进程修改内存子进程不会反应出来。进行快照持久化时，会开启一个子进程专门负责快照持久化，子进程会拥有父进程的内存数据，但在快照持久化期间父进程修改的数据不会反应在子进程种被保存，可能丢失数据

# Append Only File(AOF)

全量备份总是耗时的，Redis还提供一种更加高效的方式AOF

## 持久化步骤

1. **记录写操作：**将每一个收到的写命令都通过`write`函数追加到文件中，就像日志记录一样

   ![img](Redis进阶之一：Redis持久化/32fa828ba61ea8d3c2502e396b1b3848251f58b0.jpeg)

2. **AOF重写：**持久化文件会变的越来越大。为了压缩AOF的持久化文件。redis提供了**`bgrewriteaof`**命令。将内存中的数据以命令的方式保存到临时文件中，同时会`fork`出一条**新进程**来将文件重写，重写aof文件的操作，并没有读取旧的AOF文件，而是**将整个内存中的数据库内容用命令的方式重写了一个新的AOF文件**，这点和快照有点类似。

![img](Redis进阶之一：Redis持久化/09fa513d269759ee28454d2c4cea4b106c22dfd3.jpeg)



## 触发机制

多久将内存的数据写入文件中

- 每修改同步always：同步持久化 每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好

- 每秒同步everysec：异步操作，每秒记录 如果一秒内宕机，有数据丢失

- 不同no：从不同步

|      | always     | exyerysec                 | mo     |
| ---- | ---------- | ------------------------- | ------ |
| 优点 | 不丢失数据 | 每秒一次fsync，丢一秒数据 | 不可管 |
| 缺点 | IO开销较大 | 丢一秒数据                | 不可控 |

对应配置文件中的配置项如下：

```
# appendfsync always				#一直记录，每次有变更就记录
# appendfsync everysec				#每秒记录一次
# appendfsync no				    #每隔一段时间记录一次，根据系统里面算法决定，不安全
```

# RDB和AOF对比及选择

|            | RDB    | AOF          |
| ---------- | ------ | ------------ |
| 启动优先级 | 低     | 高           |
| 体积       | 小     | 大           |
| 恢复速度   | 快     | 慢           |
| 数据安全性 | 丢数据 | 根据策略决定 |
| 轻重       | 重     | 轻           |