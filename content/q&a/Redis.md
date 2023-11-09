+++
title = 'Redis.md'
date = 2023-11-09T15:56:46+08:00
draft = true
+++

# Redis

## Redis 数据类型

### 五种基本类型

- String 字符串

  ```shell
  set、get、del
  ```

- Hash 散列

  ```shell
  hset/hget x key value 设置、获取一个值
  hmset/hmget x key1 value1 key2 value2 同时设置、获取多个值
  hgetall x 获取某个 hash 的所有属性和值
  hdel x  删除某个 hash 的一个或多个字段
  ```

- List 列表

- Set 集合

  ```shell
  sadd、spop、scard、smembers
  ```

- Sorted Set 有序集合

  ```
  zadd
  ```

  

### 三种特殊类型

- HyperLogLog 基数统计，计算一个集合内不重复元素的个数

- GEO 地理位置信息

- Bitmap 位图

  ```shell
  setbit x key val
  getbit x key
  ```

  

## JRedis



## Redis 配置

> include 引入其他配置

> 网络

```shell
bind 127.0.0.1 # 绑定 ip
protected-mode yes # 保护模式
port 6379 # 端口设置
```

> 通用

```shell
deamonize yes # 以守护进程的方式运行，默认是 no，需要自己开启为 yes
pidfile /var/run/redis_6379.pid # 如果以后台方式运行，需要指定一个 pid 文件

# 日志级别
# debug 调试信息
# verbose 调试信息，比 debug 重要一些
# notice 类似 info 信息
# warning 警告信息
loglevel notice
logfile "" # 日志的文件位置
database 16 # 数据库的数量，默认是16
always-show-logo yes #是否总是显示 logo
```

> 快照

持久化，在规定时间内，执行了多少次操作，则会持久化到文件 .rdb、.aof。

```shell
save 900 1 # 如果 900s 内，至少有 1 个 key 进行了修改，则进行持久化操作
save 300 10
save 60 10000

stop-writes-on-gbsave-error yes # 如果持久化出错，是否继续工作

rdbcompression yes # 是否压缩 rdb 文件，会消耗一些 cpu 资源
rdbchecksum yes # 保存 rdb 文件时，是否进行错误校验
dir ./ # rdb 文件保存目录
dbfilename dump.rdb # 文件明
```

> REPLICATION 复制

> SECURITY 安全

```shell
requirepass xxxxx # 需要密码

# 控制台设置密码
127.0.0.1:6379> config set requirepass "123456"
OK
# 控制台使用密码登录
127.0.0.1:6379> auth 123456
OK
# 控制台获取密码
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
```

> 限制 CLIENTS

```shell
maxclients 10000 # 最大客户端连接数量
maxmemory <bytes> # 最大内存容量
maxmemory-policy noeviction # 内存到达上线后的处理策略
    1.volatile-lru 只对设置了过期时间的 key 进行 lru
    2.allkeys-lru 删除 lru 算法的 leu
    3.volatile-random 随机删除即将过期的 key
    4.allkeys-random 随机删除
    5.volatile-ttl 删除即将过期的
    6.noeviction 永不过期，返回错误
```

> APPEND ONLY 模式，AOF

```shell
appendonly no # 默认不开启 aof 模式，默认使用 rdb 方式，大部分情况下 rdb 完全够用
appendfilename "appendonly.aof" # aof 持久化的文件名

# appendfsync always # 每次修改都同步，消耗性能
# appendfsync no     # 不同步，操作系统会自己同步，速度最快
appendfsync everysec # 每秒执行一次同步，可能会丢失这一秒的数据

no-appendfsync-on-rewrite no # 重写 aof 文件时是否阻塞 appendfsync

auto-aof-rewrite-percentage 100 # aof 文件重写的增长比例，每次重写后 aof 重写的大小限制会根据这个比例增加
auto-aof-rewrite-min-size 64mb # aof 文件触发重写的最小限制，如果最开始 aof 文件超过这个大小，Reids 会 fork 一个新的进程来重写 aof 文件
```

## Redis 事务

```shell
watch xxx
multi
do something ...
exec
```



## Redis 持久化

Redis 的持久化分为 rdb 和 aof 两种，默认使用 rdb，aof 默认不开启，如果两种同时开启，则启动 Redis 时会优先使用 aof 文件来恢复数据，因为 aof 数据更完整。

### RDB

> RDB 是什么

RDB 是 Redis 用来进行持久化的一种方式，是把当前内存中的数据集快照写入磁盘，也就是 Snapshot 快照（数据库中所有键值对数据）。恢复时是将快照文件直接读到内存里。

Redis 会单独创建一个子进程来进行持久化，

> 触发机制

- 自动触发
  - save 的规则满足的情况下会自动触发 rdb 规则。
  - 退出 redis，也会产生 rdb 文件。
- 手动触发
  - 执行 save、bgsave 命令会生成 rdb 文件。
  - 执行 flushall 命令也会触发 rdb 规则，但生成的 rdb 文件是空的。

> 优缺点

- 优点
  - 适合大规模的数据恢复。
  - 对数据的完整性要求不高。
- 缺点
  - 需要一定的时间间隔进行操作，如果 redis 意外宕机，则会丢失最后一段时间修改的数据。
  - 创建子进程的时候会占用一定的系统资源，如果数据集过大，有可能引起系统性能问题。

### AOF（Append Only File）

> AOF 是什么

以日志的形式记录每个写操作，将 Redis 执行过的所有写指令记录下来，只许追加文件，但不可以修改文件，Redis 启动时会读取该文件重新构建数据。

AOF 默认是不开启的，修改配置中的 appendonly 项为 yes，重启 redis 即可生效。

如果 aof 文件有错误，则 Redis 是无法启动的，需要修复 aof 文件，修复 aof 文件需要用到 redis-check-aof 命令：

```shell
redis-check-aof --fix appendonly.aof
```

> 优缺点

- 优点

  aof 可以带来更高的数据安全性，它提供了三种策略：

  - 每次修改都同步，文件的完整性会更好
  - 每秒同步一次，可能会丢失一秒钟的数据
  - 从不同步，效率最高

- 缺点

  - 相对数据文件来说，aof 文件远大于 rdb，修复速度也会比 rdb 慢
  - aof 运行效率也要比 rdb 慢，所以 redis 默认的配置就是 rdb

## Redis 发布订阅

## Redis 主从复制

### 概念

是指将一台 Redis 服务器的数据，复制到其他 Redis 服务器。前者称为主节点（master/leader），后者称为从节点（slave/follower）；数据的复制时单项的，只能从主节点到从节点，Master 以写为主，Slave 以读为主。

 ### 主从复制的作用主要包括：

1. 数据冗余
2. 故障恢复
3. 负载均衡
4. 高可用基石

 ### 配置

单机多进程：修改端口、log 名、pid、存储文件名，不要和其他的冲突

> 通过配置，永久生效

```shell
replicaof [masterip] [masterport] # 作为某台主机的从机
```

> 通过命令，暂时生效

```shell
slaveof [host] [port] # 在从机中使用命令，指定作为某台主机的从机
slaveof no one # 如果主机断开，可以使用 slaveof no one 解除主机
```

>复制原理

slave 连接到 master 后会发送一个一个 sync 同步命令

master 接到命令后，启动后台存盘进程，同时收集所有修改数据集的命令，在后台进程执行完毕后，master 将整个数据文件同步到 slave，完成一次完全同步

全量复制：slave 服务器在接收到数据库文件后，将其存盘并加载到内存中

增量复制：master 继续将新的修改命令一次传给 slave，完成同步

> 哨兵

```shell
sentinel monitor myredis ip port 1 # 配置，sentinel monitor 被监控名 ip port 1

redis-sentinel sentinel.conf # 启动哨兵命令
```



## 缓存穿透和雪崩

### 缓存穿透

缓存穿透就是一种请求数据缓存存在大量不命中的情况，例如查询数据时，redis 和持久层数据库中都没有查询到，不停的发送查询请求，就会出现缓存穿透，缓存就失去了意义

> 解决方案

1. 布隆过滤器
2. 返回空缓存

### 缓存击穿

缓存击穿是指缓存中没有但数据库中有数据，由于并发用户特别多，同时都缓存没有读到数据，又同时都去数据库库请求数据，引起数据库压力瞬间增大，造成过大压力的一种情况

> 解决方案

1. 设置热点信息永不过期
2. 加互斥锁

### 缓存雪崩

缓存雪崩类似于缓存击穿，不同的是指缓存中的数据大批量过期或redis 突然宕机而导致用户去缓存无法获取到数据，而大批量到数据库请求数据而造成数据库压力过大的一种情况

> 解决方案

1. redis 高可用，集群
2. 限流降级