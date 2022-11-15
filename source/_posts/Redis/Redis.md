---
title: Redis

date: 2020-02-29

categories: 
- Redis

---

Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。与传统数据库不同的是，redis的数据是存在内存中的，所以读写速度非常快，因此被广泛用于缓存方向。另外，redis也常用来做分布式锁。

<!--more-->

### 数据类型

redis是key-value数据库，它的可以全部是string类型的，而value则有五种类型

- String
- Hash
- List
- Set
- zset (Sorted Set)

### 设置过期时间

Redis中有个设置时间过期的功能，即对存储在redis数据库中的值可以设置一个过期时间。作为一个缓存数据库，这是非常实用的。如我们一般项目中的 token 或者一些登录信息，尤其是短信验证码都是有时间限制的，按照传统的数据库处理方式，一般都是自己判断过期，这样无疑会严重影响项目性能。

我们 set key 的时候，都可以给一个 expire time，就是过期时间，通过过期时间我们可以指定这个 key 可以存活的时间。

如果假设你设置了一批 key 只能存活1个小时，那么接下来1小时后，redis是怎么对这批key进行删除的？

**定期删除+惰性删除。**

通过名字大概就能猜出这两个删除方式的意思了。

- **定期删除**：redis默认是每隔100ms就**随机抽取**一些设置了过期时间的key，检查其是否过期，如果过期就删除。选择随机删除的原因是如果redis中存在大量key每隔100ms就要遍历所有的key太耗费性能了
- **惰性删除** ：定期删除可能会导致很多过期 key 到了时间并没有被删除掉。所以就有了惰性删除。假如你的过期 key，靠定期删除没有被删除掉，还停留在内存里，当你使用到这个key是redis发现是已经过期的了就会把它删除

### 内存淘汰机制

**redis 提供 6种数据淘汰策略：**

如果定期删除漏掉了很多key，但是我们又一直没有去使用这些key，也不会进行惰性删除，导致大量内存被占用时，redis就会使用自己的内存淘汰机制。

1. **volatile-lru**：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
2. **volatile-ttl**：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
3. **volatile-random**：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
4. **allkeys-lru**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（最常用）
5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
6. **no-eviction**：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。

4.0版本后增加以下两种：

7. **volatile-lfu**：从已设置过期时间的数据集(server.db[i].expires)中挑选最不经常使用的数据淘汰
8. **allkeys-lfu**：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的key

在redis配置文件中有一行： # maxmemory-policy volatile-lru  采用哪种内存淘汰策略对应修改即可


### 持久化

redis做持久化是为了机器重启后能恢复数据或者为了防止故障先进行备份，redis持久化主要有两种方式

**RDB (snapshotting）快照持久化**

redis可以通过创建快照来获取内存中的数据副本，对快照备份后可以复制到其它服务器（redis主从结构），提高可用性，也可以保留在原地以便机器故障后重启使用。

RDB是redis的默认持久化方式，redis.conf配置文件中的默认配置：

```
save 900 1           #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 300 10          #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
```

**AOF（append-only file）持久化**

默认情况下redis没有开启AOF持久化，可在配置文件中修改开启

```
appendonly yes
```

开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件，因此AOF的实时性更好。

在Redis的配置文件中存在三种不同的 AOF 持久化方式，它们分别是：

```conf
appendfsync always    #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
appendfsync everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘
appendfsync no        #让操作系统决定何时进行同步
```

为了兼顾性能和数据，可以考虑使用 appendfsync everysec 配置，这样每秒写入一次，对性能影响较小，即便出了故障也只损失一秒的数据。

### redis遇到的问题

- 缓存雪崩
- 缓存穿透
- 缓存击穿

**缓存雪崩**

缓存雪崩是指缓存中数据大批量到过期时间，请求全都落到数据库上以致压力过大。

解决方案：

1. 给缓存设置时间时再加一个随机数，防止同一时间缓存大量过期
2. 果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
3. hystrix限流降级


**缓存穿透**

存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为-1的数据。

解决方案

1. 布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉。
2. 一个查询返回的数据为空（不管是数 据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但是缓存时间很短，一般不超过五分钟。


**缓存击穿**

 缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。  和**缓存雪崩**不同的是，缓存雪崩是大量数据缓存到期，而缓存击穿是单一数据缓存到期但是访问量较多造成的压力，比如社区热点信息。
 
 解决方案
 
 1. 定时任务主动更新缓存
 2. 加锁机制，使得只有一个请求能落到数据库，其它请求还是从缓存中获取