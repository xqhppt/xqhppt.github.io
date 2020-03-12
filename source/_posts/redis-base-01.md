---
title: Redis数据类型的使用场景
date: 2020-03-12 16:56:00
author: xqh
tags:
- Redis
- 缓存
categories: Redis
---

# String
### get/set mget/mset

说明：设置/批量设置键值信息，value可以是任何形式的字符串

用法：

``set key value``

``get key``

场景：

常规缓存使用方式

### incr/decr

说明：将key中的数值增/减1，如果key不存在会先初始化为0再加1，该操作是原子性的

用法：

``incr key``

``get key``

场景：

结合``expire``用做计数器和限数器

如某篇文章点击量通过``incr article:id``来计数，用``get article:id``来获取

### setbit/getbit/bitcount

说明：设置指定偏移量上的位；获取指定偏移量上的位；指定key中，位值为1的数量

用法：

``setbit key offset value``

``getbit key offset``

``bitcount key``

场景：

活跃数统计，开关等

如日活跃人数，通过``setbit dayactive:yyyyMMdd userid 1``，通过``bitcount dayactive:yyyyMMdd``来获取

### setnx/getset

说明：当key不存在时候将key的值设为value，存在不做操作；将key的值设为value，并返回key的旧值

用法：

``setnx a 11``

``getset a 22``

场景：

分布式锁实现


# Hash

### hset/hget/hkeys/hgetall

说明：当key不存在时候将key的值设为value，存在不做操作；将key的值设为value，并返回key的旧值

用法：

``hset key field value``

``hget key field``

``hkeys key``

``hgetall key``

场景：

存储对象信息，用于优化普通键值存储

如原来存储用户的年龄、姓名、性别等信息需要``set user:userid {\"name\":\"lily\",\"age\":18,\"sex\":1}``这样把信息转出json后再存，
当数据量大的时候十分占用内存；

用hash类型可以很好解决这个问题

通过``hmset user:userid name lily age 18 sex 1``来存储

通过``hgetall user:userid``获取全部内容

通过``hget user:userid name``获取某个属性


# List

### lpush/rpop/brpop/lrange/llen

说明：list是redis比较重要的一个数据结构，支持插入列表、弹出队列、阻塞弹出队列、返回列表元素、获取列表长度等操作

用法：

``lpush ls 1 2 3 4 5``

``lrange ls 0 -1``

场景：

列表、消息队列等

如列表反向cache，通过列表存储数据库存储信息，然后在从数据库中获取详细信息

如生产者往队列中插入消息``lpush queue:topic:name msg``，消费者通过``rpop ls``和``brpop ls 10``来消费消息


# Set

### sadd/srem/sdiff/sinter/smembers/sunion 

说明：实现了集合相关的添加、删除、差集、交集、并集等操作

用法：

``sadd s1 1 2 3 5 6``

``sinter s1 s2``

场景：

好友、关注、被关注、共同好友等

如关注我的人，通过``sadd user:myid 10``来关注我，通过``smembers user:myid``来获取关注我的人


# SortedSet

### zadd/zrem/zcard/zrange/zrank/zrangebyscore 

说明：可以添加元素的权重和值到key中，并按权重来获取序列

用法：

``zadd ls 10 a1 6 a2 17 a3``

``zrange ls 0 -1``

场景：

排行榜、有序队列

如排名前10的玩家，通过``zadd user:leaderboard score userid``来设置玩家分值，通过``zrevrange user:leaderboard 0 10``来获取
