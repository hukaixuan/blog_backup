---
title: "《redis实战》笔记--数据结构及其命令"
date: 2017.04.24 10:31
tags:
 - Redis
---

![](http://upload-images.jianshu.io/upload_images/3340699-7d580a8ccce7de87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 概述

----------------------------------

 redis 可以存储键与 5 种不同数据结构类型之间的映射，分别为 STRING(字符串), LIST(列表), SET(集合), HASH(散列), ZSET(有序集合)，有些命令是对于这些数据结构是通用的，而有些只是对于特定数据结构的。

----------------------------------

#### 1. STRING

- 字符串可以存储三种类型的值：
  - 字节串(byte string)       
  - 整数
  - 浮点数

- 如果一个存储在redis字符串中的值可以被看做十进制整数或者浮点数，那么redis便允许对该值进行如下增减操作；如果用户对一个不存在的建或一个保存了空串的键执行自增或自减操作，会被视为0处理；如果这个值不能解释成一个整数或者浮点数，执行以下增减操作会报错（Python调用之前先`import redis`, `conn = redis.Redis()`）：

命令  | redis用例和描述  |  Python redis库调用
------|-----------------|---------------------
incr  |  `incr key` 将键存储的值加1  | `conn.incr('key')`
decr | `decr key` 将键存储的值减1 | `conn.decr('key')`
incrby | `incrby key amount` 将键存储的值加整数amount | `conn.incr('key', amount)`
decrby | `decrby key amount` 将键存储的值减整数amount | `conn.decr('key', amount)`
incrbyfloat | `incrbyfloat key amount` 将键存储的值加上浮点数 amount | `conn.incrbyfloat('key', amount)`
没有decrbyfloat|

- 处理子串和二进制位的命令：

命令  | redis用例和描述  | Python redis库调用
------|-----------------|--------------------
append| `append key value `将值value追加到给定的键 key当前存储的值的末尾| `conn.append('key', 'value')`
getrange(2.6之前为substr)| `getrange key start end` 获取一个由偏移量start至end范围内所有字符组成的子串，**包括start和end在内**（索引以0开始）| `conn.getrange('key', start, end)`
setrange| `setrange key offset value` 将从偏移offset开始的子串设置为给定值 | `conn.setrange('key', offset, 'new-value')`
位操作|getbit、setbit、bitcount、bitop|

----------------------------------

#### 2. LIST

- redis 的列表允许用户从序列的两端push或者pop元素, 获取列表元素，以及执行各种常见的列表操作。还可以用来存储任务信息、最近浏览过的文章或者常用联系人信息。

- 常用列表命令如下：

命令 | redis用例和概述 | python redis 库调用
-----|-----------------|--------------------
rpush|`rpush key vlaue [value ...]` 将一个或多个值push进列表的右端| `conn.rpush('key', 'value')`
lpush|`lpush key value [value ...]`...左端|`conn.lpush('key', 'value')`
rpop| `rpop key ` 移除并返回列表最右端的元素|`conn.rpop('key')`
lpop| `lpop key` 移除并返回列表最左端的元素|`conn.lpop('key')`
lindex| `lindex key offset` 返回列表中偏移量为offset的元素|`conn.index('key', offset)`
lrange|`lrange key start stop` 返回列表中从start偏移量到end偏移量范围内的所有元素，**包括start和end**|`conn.lrange('key', start, stop)`
ltrim | `ltrim key start stop` 对列表进行修剪，只保留从start到end(**包括start和end**)之间的元素| `conn.ltrim('key', start, stop)`

- 阻塞式pop及列表之间移动元素：

命令 | redis用例和描述 | Python redis 库调用
-----|-----------------|---------------------
blpop| `blpop key[key...] timeout` 从第一个非空列表中弹出位于最左端的元素，或者在timeout秒之内阻塞并等待可弹出的元素出现 | `conn.blpop(keys, timeout)`
brpop| `brpop key[key...] timeout` 从第一个非空列表中弹出位于最右端的元素，或者在timeout秒之内阻塞并等待可弹出的元素出现 |`conn.brpop(keys, timeout)`
rpoplpush| `rpoplpush source-key dest-key` 从source-key列表中pop出位于最右端的元素，然后将这个元素push进dest-key列表的最左端，并向用户返回这个元素|`rpoplpush(source-key, dest-key)`
brpoplpush| `brpoplpush source-key dest-key timeout` 从source-key列表中弹出位于最右端的元素，然后将这个元素推入dest-key列表的最左端，并向用户返回这个元素：如果souce-key为空，那么在timeout秒之内阻塞并等待可弹出的元素出现|`brpoplpush(source-key, dest-key, timeout)`
方便理解阻塞式(block), 演示如下


![阻塞式演示，左为redis-cli，右为python命令行](http://upload-images.jianshu.io/upload_images/3340699-556d0ea1db06dcaa.gif?imageMogr2/auto-orient/strip)

---------------------------------------

#### 3. SET

- Redis 的集合操作以无序的方式来存储多个**各不相同**的元素，用户可以快速地对集合必行添加元素操作，移除元素操作以及检查一个元素是否存在于集合里。

- 常用集合命令

命令 | redis用例和描述 | Python redis库调用
-----|-----------------|--------------------
sadd|`sadd key item [item...]` 添加 |`conn.sadd(key, *items)`
srem|`srem key itme [...]` 删除|`conn.srem(key, *items)`
sismember|`sismember` 检查元素item是否存在于集合key里|`conn.sismember(key, item)`
scard|`scard key` 返回集合包含的元素的数量|`conn.scard(key)`
semebers|`smembers key` 返回集合中所包含的所有元素|`conn.smembers(key)`
srandmember|`srandmember key [count]`从集合里面随机地返回一个或多个元素，当count为正时，命令返回的随机元素不会重复，为负数的时候可能会重复|`conn.srandmember(key, count=None)`
spop|`spop key` **随机**地移除集合中的一个元素，并返回被移除的元素|`conn.spop(key)`
smove|`smove source-key dest-key item`如果集合source-key包含元素item，那么从集合source-key中移除元素item，并将元素item添加到集合dest-key中；如果item被成功移除，那么命令返回1，否则返回0|`conn.smove(source-key dest-key item)`

- 用于组合和处理多个集合的redis命令

命令 | redis用例和描述 | Python redis库调用
-----|-----------------|--------------------
sdiff |`sdiff key [keys...]`返回那些存在于第一个集合、但不存在与其他集合中的元素（**差**）|`conn.sdiff(key, *keys)`
sdiffstore | `sdiffstore dest-key key [keys...]`将那些存在于第一个集合但并不存在与其他集合中的元素存储到dest-key键里面 | `conn.sdiffstore(dest-key, key, *keys)`
sinter | `sinter key [keys...]`返回那些同时存在于所有集合中的元素（**交**） | `conn.sinter(key, *keys)`
sinterstore | `sinterstore dest-key key [keys...]`交&存储 | `conn.sinterstore(dest-key, key, *keys)`
sunion | `sunion key [keys...]` **并**| `conn.sunion(key, *keys)`
sunionstore | `sunionstore dest-key key [keys...]` 并&存储| `conn.sunionstore(dest-key, key, *keys)`

------------------------------------------


#### 4. HASH

- Redis的散列可以让用户将多个键值对存储到一个Redis键里面。从功能上来说，Redis为散列值提供了一些与字符串值相同的特性，是的散列非常适用于将一些相关数据存储在一起。因此可以吧这种数据聚集看做是关系数据库中行，或者文档数据库中的文档。

- 用于添加和删除键值对的散列操作

命令 | redis用例和描述 | Python redis 库调用
-----|-----------------|---------------------
hmget| `hmget key-name key [keys...]` 从散列里面获取一个或者多个键的值 |`hmget(key-name, key [keys...])`
hmset | `hmset key-name key value [keys values ...] `为散列里的一个或多个键设置值| `hmset(key-name, dict(key=value))`
hdel | `hdel key-name key [keys ...]`删除散列里面的一个或多个键值对，返回删除的数量 | `conn.hdel(key-name, key, [keys...])`
hlen | `hlen key-name` 返回散列包含的键值对数量| `conn.hlen(key-name)`

- Redis 散列的更高级特性

命令 | redis用例和描述 | Python redis库调用
-----|-----------------|--------------------
hexists | `hexists key-name key`检查给定键是否存在于散列中 | `conn.hexists('key-name', 'key')`
hkeys | `hkeys key-name` 获取散列的所有键 | `conn.hkeys('key-name')`
hvals | `hvals key-name` 获取散列包含的所有值 | `conn.hvals('key-name')`
hgetall | `hgetall key-name` 获取散列包含的所有键值对 | `conn.hgetall('key-name')`
hincrby | `hincrby key-name key increment` 将键key保存的值加上整数increment, 对散列中一个不存在的键执行自增操作时会将键的值当做0来处理 | `conn.hincrby('key-name', 'key', increment)`
hincrbyfloat | `hincrbyfloat key-name key increment` 将键key保存的值加上浮点数increment，若 不存在当做0来处理 | `conn.hincrbyfloat('key-name', 'key', increment)`

-`hgetall`vs `hkeys&hget` :  如果散列包含的值非常大，那么用户可以先使用`hkeys` 取出散列包含的所有键， 然后再使用 `hget`一个接一个地取出键的值，从而**避免因为一次获取多个大体积的值而导致服务器阻塞**


----------------------------------------
#### 5. ZSET

- 与散列存储键值对类似，有序集合ZSET存储着成员(member)与其分值(score)之间的映射，并提供了分值处理命令，以及根据分值大小有序地获取(fetch) 或扫描(scan) 成员和分值的命令。

命令 | redis用例和描述 | python redis 库调用
-----|-----------------|---------------------
zadd|`zadd key-name score member [score member ...]` 将带有给点分值的成员添加到有序集合里面|`conn.zadd('key-name', 'member1', 'score1', 'member2', 'score2')` **与redis标准的先输入score后输入member的做法相反**
zrem|`zrem key-name member  [member...]` 从有序集合里面移除给定的成员，并返回被移除成员的数量|`conn.zrem('key-name', 'member')`
zcard|`zcard key-name` 返回有序集合包含的成员数量|`conn.zcard('key-name')`
zincrby|`zincrby key-name increment member` 将member成员的分值加上increment|`conn.zincrby('key-name', 'member', increment)`
zcount|`zcount key-name min max` 返回分值介于min 和 max 之间的成员数量|`conn.zcount('key-name', min, max)` 获取成员的排名(以0开始)
zrank|`zrank key-name member` 返回成员member 在有序集合中的排名|`conn.zrank('key-name', 'member')`
zscore|`zscore key-name member` 返回成员member的分值|`conn.zscore('key-name', 'member')`
zrange|`zrange key-name start stop [withscores]` 返回有序集合中排名介于start和stop之间的成员，如果给定了可选的withscore选项，那么命令会将成员的分值也一并返回|`conn.zrange('key-name', start, stop, withscores=True)`
zrangebyscore|`zrangebyscore key min max [withscores] [limit offset count]` 获取有序集合中分值介于min和max 之间的所有成员|

- 有序集合的范围型数据获取、删除命令，并集交集命令

描述      | 命令
---------|---------------------
逆序命令(从大到小)| **zrevrank、zrevrange、zrevrangebyscore**
删除操作| **zremrangebyrank、zremrangebyscore**
交并运算|1. **zinterstore:**`zinterstore dest-key key-count key [key ...] [weights weight [weight ...]] [aggregate sum、min、max]`<br>2.**zunionstore:**`zunionstore dest-key key-count key [key ...] [weights weight [weight ...]] [aggregate sum、min、max]`

- 交集和并集运算默认使用聚合函数sum，可以通过 aggregate= sum/min/max 来指定三种不同的聚合函数，通过可选的weights参数来决定个子集合的权重。
