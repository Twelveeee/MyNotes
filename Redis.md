# Redis

Redis 官网：https://redis.io/

中文网：http://www.redis.cn/

源码地址：https://github.com/redis/redis

Redis 在线测试：http://try.redis.io/

Redis 命令参考：http://doc.redisfans.com/

## 简介

REmote DIctionary Server(Redis) 是完全开源的，遵守 BSD 协议，是一个高性能的 key-value 数据库。

Redis 与其他 key - value 缓存产品有以下三个特点

- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

## 数据结构

- String: 字符串
- Hash: 散列
- List: 列表
- Set: 集合
- Sorted Set: 有序集合

## 命令

### Keys 命令

任何二进制序列都可以作为Redis的Key使用（例如普通的字符串或一张JPEG图片）
Redis允许的最大Key长度是512MB（对Value的长度限制也是512MB）

| 命令               | 描述                                    |
| ------------------ | --------------------------------------- |
| DEL key            | 该命令用于在 key 存在时删除 key。       |
| DUMP key           | 序列化给定 key ，并返回被序列化的值。   |
| EXISTS key         | 检查给定 key 是否存在。                 |
| EXPIRE key seconds | 为给定 key 设置过期时间，以秒计。       |
| KEYS pattern       | 查找所有符合给定模式( pattern)的 key 。 |
| PERSIST key        | 移除 key 的过期时间，key 将持久保持。   |
| ...                | ...                                     |

### String（字符串）

```sql
-- 语法
COMMAND KEY_NAME

SET redis "key-value数据库"
GET redis
"key-value数据库"
```

一个键最大能存储512mb

| 命令                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| SET key value            | 设置指定 key 的值。O(1)                                      |
| GET key                  | 获取指定 key 的值。O(1)                                      |
| GETSET key value         | 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。O(1) |
| MSET key1 [key2..] value | 为多个key设置value。O(N)                                     |
| MGET key1 [key2..]       | 获取所有(一个或多个)给定 key 的值。O(N)                      |
| SETNX key value          | 只有在 key 不存在时设置 key 的值。O(1)                       |
| INCR key                 | 将key对应的value值自增1，并返回自增后的值。<br />只对可以转换为整型的String数据起作用。O(1) |
| INCRBY key value         | 将key对应的value值自增指定的整型数值，并返回自增后的值。O(1) |
| DECR/DECRBY              | 同INCR/INCRBY，自增改为自减。                                |
| ...                      | ...                                                          |

INCR/DECR系列命令要求操作的value类型为String，并可以转换为64位带符号的整型数字，否则会返回错误。
也就是说，进行INCR/DECR系列命令的value，必须在[-2^63 ~ 2^63 - 1]范围内。
Redis采用单线程模型，天然是线程安全的，这使得INCR/DECR命令可以非常便利的实现高并发场景下的精确控制。

### Hash（哈希）

Hash即哈希表，Redis的Hash和传统的哈希表一样，是一种field-value型的数据结构，可以理解成将HashMap搬入Redis。
Hash非常适合用于表现对象类型的数据，用Hash中的field对应对象的field即可。
Hash的优点包括：
可以实现二元查找，如"查找ID为1000的用户的年龄"
比起将整个对象序列化后作为String存储的方法，Hash能够有效地减少网络传输的消耗
当使用Hash维护一个集合时，提供了比List效率高得多的随机访问命令

```sql
HMSET runoobkey name "redis tutorial" description "redis basic commands for caching" likes 20 visitors 23000

HGETALL runoobkey
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
```

| 命令                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| HSET key field value     | 将key对应的Hash中的field设置为value。<br />如果该Hash不存在，会自动创建一个。O(1) |
| HGET key field           | 获取存储在哈希表中指定字段的值。                             |
| HMSET/HMGET              | 同HSET和HGET，可以批量操作同一个key下的多个field。O(N)       |
| HSETNX key field value   | 同HSET，但如field已经存在，HSETNX不会进行任何操作。O(1)      |
| HEXISTS key field        | 判断指定Hash中field是否存在，存在返回1，不存在返回0。O(1)    |
| HDEL key field1 [field2] | 删除一个或多个哈希表字段                                     |
| HINCRBY  key field       | 同INCRBY命令，对指定Hash中的一个field进行INCRBY，O(1)        |
| HLEN key                 | 获取哈希表中字段的数量                                       |
| HGETALL key              | 获取在哈希表中指定 key 的所有字段和值 O(N)                   |
| HKEYS key                | 获取所有哈希表中的字段  O(N)                                 |
| HVALS key                | 获取所有哈希表中的值  O(N)                                   |

`HGETALL\HKEYS\HVALS`会对Hash进行完整遍历，Hash中的field数量与命令的耗时线性相关，对于尺寸不可预知的Hash，应严格避免使用上面三个命令，而改为使用HSCAN命令进行游标式的遍历，

### 列表(List)

Redis的List是链表型的数据结构，可以使用`LPUSH/RPUSH/LPOP/RPOP`等命令在List的两端执行插入元素和弹出元素的操作。
List也支持在特定index上插入和读取元素的功能，但其时间复杂度较高（O(N)），应小心使用。

一个列表最多可以包含 2^(32) -  个元素 (4294967295, 每个列表超过40亿个元素)。

```sql
redis 127.0.0.1:6379> LPUSH runoobkey redis
(integer) 1
redis 127.0.0.1:6379> LPUSH runoobkey mongodb
(integer) 2
redis 127.0.0.1:6379> LPUSH runoobkey mysql
(integer) 3
redis 127.0.0.1:6379> LRANGE runoobkey 0 10

1) "mysql"
2) "mongodb"
3) "redis"
```

| 命令                                  | 描述                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| LPUSH key value1 [value2]             | 将一个或多个值插入到列表头部                                 |
| LPOP key                              | 移出并获取列表的第一个元素                                   |
| RPUSH/RPOP                            | 同上                                                         |
| LPUSHX/RPUSHX                         | 与LPUSH/RPUSH类似，区别在于，LPUSHX/RPUSHX操作的key如果不存在，则不会进行任何操作 |
| LLEN key                              | 返回指定List的长度，时间复杂度O(1)                           |
| LRANGE                                | 返回指定List中指定范围的元素（双端包含，即LRANGE key 0 10会返回11个元素），时间复杂度O(N)。应尽可能控制一次获取的元素数量，一次获取过大范围的List元素会导致延迟，同时对长度不可预知的List，避免使用LRANGE key 0 -1这样的完整遍历操作。 |
| LINDEX key index                      | 返回指定List指定index上的元素，如果index越界，返回nil。index数值是回环的，即-1代表List最后一个位置，-2代表List倒数第二个位置。时间复杂度O(N) |
| LSET key index value                  | 将指定List指定index上的元素设置为value，如果index越界则返回错误，时间复杂度O(N)，如果操作的是头/尾部的元素，则时间复杂度为O(1) |
| LINSERT key BEFORE\|AFTER pivot value | 向指定List中指定元素之前/之后插入一个新元素，并返回操作后的List长度。如果指定的元素不存在，返回-1。如果指定key不存在，不会进行任何操作，时间复杂度O(N) |
| BLPOP key1 [key2 ] timeout            | 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOP key1 [key2 ] timeout            | 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |

由于Redis的List是链表结构的，`LINDEX\LSET\LINSERT`算法效率较低，需要对List进行遍历，命令的耗时无法预估，在List长度大的情况下耗时会明显增加，应谨慎使用。
`BLPOP/BRPOP`能够实现类似于BlockingQueue的能力，即在List为空时，阻塞该连接，直到List中有对象可以出队时再返回。

### 集合(Set)

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。
集合对象的编码可以是 intset 或者 hashtable。

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。
集合中最大的成员数为 2^(32) - 1 (4294967295, 每个集合可存储40多亿个成员)。

```sql
redis 127.0.0.1:6379> SADD runoobkey redis
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mongodb
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 1
redis 127.0.0.1:6379> SADD runoobkey mysql
(integer) 0
redis 127.0.0.1:6379> SMEMBERS runoobkey

1) "mysql"
2) "mongodb"
3) "redis"
```

| 命令                       | 描述                                 |
| -------------------------- | ------------------------------------ |
| SADD key member1 [member2] | 向集合添加一个或多个成员             |
| SCARD key                  | 获取集合的成员数                     |
| SDIFF key1 [key2]          | 返回第一个集合与其他集合之间的差异。 |
| SMEMBERS key               | 返回集合中的所有成员                 |
| SPOP key                   | 移除并返回集合中的一个随机元素       |
| SREM key member1 [member2] | 移除集合中一个或多个成员             |
| ...                        | ...                                  |

### 有序集合(sorted set)

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 2^(32) - 1 (4294967295, 每个集合可存储40多亿个成员)。

```sql
redis 127.0.0.1:6379> ZADD runoobkey 1 redis
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 2 mongodb
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 1
redis 127.0.0.1:6379> ZADD runoobkey 3 mysql
(integer) 0
redis 127.0.0.1:6379> ZADD runoobkey 4 mysql
(integer) 0
redis 127.0.0.1:6379> ZRANGE runoobkey 0 10 WITHSCORES

1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "4"
```

| 命令                                     | 描述                                                   |
| ---------------------------------------- | ------------------------------------------------------ |
| ZADD key score1 member1 [score2 member2] | 向有序集合添加一个或多个成员，或者更新已存在成员的分数 |
| ZCARD key                                | 获取有序集合的成员数                                   |
| ZCOUNT key min max                       | 计算在有序集合中指定区间分数的成员数                   |
| ZINCRBY key increment member             | 有序集合中对指定成员的分数加上增量 increment           |
| ZLEXCOUNT key min max                    | 在有序集合中计算指定字典区间内成员数量                 |
| ...                                      | ...                                                    |



## 持久化

Redis提供了将数据定期自动持久化至硬盘的能力，包括RDB和AOF两种方案，两种方案分别有其长处和短板，可以配合起来同时运行，确保数据的稳定性。

### RBD

采用RDB持久方式，Redis会定期保存数据快照至一个rbd文件中，并在启动时自动加载rdb文件，恢复之前保存的数据。

**优点**
对性能影响最小。如前文所述，Redis在保存RDB快照时会fork出子进程进行，几乎不影响Redis处理客户端请求的效率。
每次快照会生成一个完整的数据快照文件，所以可以辅以其他手段保存多个时间点的快照（例如把每天0点的快照备份至其他存储媒介中），作为非常可靠的灾难恢复手段。
使用RDB文件进行数据恢复比使用AOF要快很多。

**缺点**
快照是定期生成的，所以在Redis crash时或多或少会丢失一部分数据。
如果数据集非常大且CPU不够强（比如单核CPU），Redis在fork子进程时可能会消耗相对较长的时间（长至1秒），影响这期间的客户端请求。

### AOF

采用AOF持久方式时，Redis会把每一个写请求都记录在一个日志文件里。在Redis重启时，会把AOF文件中记录的所有写操作顺序执行一遍，确保数据恢复到最新。

**优点**
最安全，在启用appendfsync always时，任何已写入的数据都不会丢失，使用在启用appendfsync everysec也至多只会丢失1秒的数据。
AOF文件在发生断电等问题时也不会损坏，即使出现了某条日志只写入了一半的情况，也可以使用redis-check-aof工具轻松修复。
AOF文件易读，可修改，在进行了某些错误的数据清除操作后，只要AOF文件没有rewrite，就可以把AOF文件备份出来，把错误的命令删除，然后恢复数据。

**缺点**
AOF文件通常比RDB文件更大
性能消耗比RDB高
数据恢复速度比RDB慢



## 数据淘汰

什么时候淘汰？
要写入的时候，内存不够了。
定期淘汰。

Redis提供了5种数据淘汰策略：

`volatile-lru`：使用LRU算法进行数据淘汰（淘汰上次使用时间最早的，且使用次数最少的key），只淘汰设定了有效期的key
`allkeys-lru`：使用LRU算法进行数据淘汰，所有的key都可以被淘汰
`volatile-random`：随机淘汰数据，只淘汰设定了有效期的key
`allkeys-random`：随机淘汰数据，所有的key都可以被淘汰
`volatile-ttl`：淘汰剩余有效期最短的key

## 集群

主从复制模式能实现读写分离，但是不能自动故障转移；
哨兵模式基于主从复制模式，能实现自动故障转移，达到高可用，但与主从复制模式一样，不能在线扩容，容量受限于单机的配置；
Cluster模式通过无中心化架构，实现分布式存储，可进行线性扩展，也能高可用，但对于像批量操作、事务操作等的支持性不够好。

### 主从复制

**基本原理**

主从复制模式中包含一个主数据库实例（master）与一个或多个从数据库实例（slave）
客户端可对主数据库进行读写操作，
客户端对从数据库进行读操作，
主数据库写入的数据会实时自动同步给从数据库。

**工作机制**
slave启动后，向master发送SYNC命令，master接收到SYNC命令后保存快照，并使用缓冲区记录保存快照这段时间内执行的写命令（写成log）
master将保存的快照文件发送给slave，并继续记录执行的写命令
slave接收到快照文件后，加载快照文件，载入数据
master快照发送完后开始向slave发送缓冲区的写命令，slave接收命令并执行，完成复制初始化
此后master每次执行一个写命令都会同步发送给slave，保持master与slave之间数据的一致性

**优点**
master能自动将数据同步到slave，可以进行读写分离，分担master的读压力
master、slave之间的同步是以非阻塞的方式进行的，同步期间，客户端仍然可以提交查询或更新请求

**缺点**
不具备自动容错与恢复功能，master或slave的宕机都可能导致客户端请求失败，需要等待机器重启或手动切换客户端IP才能恢复
master宕机，如果宕机前数据没有同步完，则切换IP后会存在数据不一致的问题
难以支持在线扩容，Redis的容量受限于单机配置

### Sentinel（哨兵）模式

**基本原理**
哨兵模式基于主从复制模式，只是引入了哨兵来监控与自动处理故障。
哨兵顾名思义，就是来为Redis集群站哨的，一旦发现问题能做出相应的应对处理。其功能包括：

监控master、slave是否正常运行
当master出现故障时，能自动将一个slave转换为master（大哥挂了，选一个小弟上位）
多个哨兵可以监控同一个Redis，哨兵之间也会自动监控

**工作机制**
哨兵与master数据库建立连接，定期向master和slave、和其他的sentinel发送信息。
sentinel监控的时候发送内容为哨兵的ip端口、运行id、配置版本、master名字、master的ip端口还有master的配置版本，其他哨兵可以通过该信息判断是否是新的哨兵，判断master的版本是否为最新，判断数据库节点有没有停止服务。
如果发现master挂了，哨兵向共同监控这个master的哨兵发送信息，确认master真的挂了，从剩下的slave中推选出新的master，原本的master变成slave。

**优点**
哨兵模式基于主从复制模式，所以主从复制模式有的优点，哨兵模式也有
哨兵模式下，master挂掉可以自动进行切换，系统可用性更高

**缺点**
同样也继承了主从模式难以在线扩容的缺点，Redis的容量受限于单机配置
需要额外的资源来启动sentinel进程，实现相对复杂一点，同时slave节点作为备份节点不提供服务

### Cluster模式

Cluster模式实现了Redis的分布式存储，即每台节点存储不同的内容，来解决在线扩容的问题。

**基本原理**

Cluster采用无中心结构
所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽
节点的fail是通过集群中超过半数的节点检测失效时才生效
客户端与redis节点直连,不需要中间代理层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可

**工作机制**
在Redis的每个节点上，都有一个插槽（slot），取值范围为0-16383
当我们存取key的时候，Redis会根据CRC16的算法得出一个结果，然后把结果对16384求余数，这样每个key都会对应一个编号在0-16383之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作
为了保证高可用，Cluster模式也引入主从复制模式，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点
当其它主节点ping一个主节点A时，如果半数以上的主节点与A通信超时，那么认为主节点A宕机了。如果主节点A和它的从节点都宕机了，那么该集群就无法再提供服务了

Cluster模式集群节点最小配置6个节点(3主3从，因为需要半数以上)，其中主节点提供读写操作，从节点作为备用节点，不提供请求，只作为故障转移使用。

**优点**
无中心架构，数据按照slot分布在多个节点。
集群中的每个节点都是平等的关系，每个节点都保存各自的数据和整个集群的状态。每个节点都和其他所有节点连接，而且这些连接保持活跃，这样就保证了我们只需要连接集群中的任意一个节点，就可以获取到其他节点的数据。
可线性扩展到1000多个节点，节点可动态添加或删除
能够实现自动故障转移，节点之间通过gossip协议交换状态信息，用投票机制完成slave到master的角色转换

**缺点**
客户端实现复杂，驱动要求实现Smart Client，缓存slots mapping信息并及时更新，提高了开发难度。目前仅JedisCluster相对成熟，异常处理还不完善，比如常见的“max redirect exception”
节点会因为某些原因发生阻塞（阻塞时间大于 cluster-node-timeout）被判断下线，这种failover是没有必要的
数据通过异步复制，不保证数据的强一致性
slave充当“冷备”，不能缓解读压力
批量操作限制，目前只支持具有相同slot值的key执行批量操作，对mset、mget、sunion等操作支持不友好
key事务操作支持有线，只支持多key在同一节点的事务操作，多key分布不同节点时无法使用事务功能
不支持多数据库空间，单机redis可以支持16个db，集群模式下只能使用一个，即db 0 Redis Cluster模式不建议使用pipeline和multi-keys操作，减少max redirect产生的场景。

## 应用场景

### 缓存

在日常对数据库的访问中，读操作的次数远超写操作，比例大概在 **1:9** 到 **3:7**，所以需要读的可能性是比写的可能大得多的。当我们使用SQL语句去数据库进行读写操作时，数据库就会**去磁盘把对应的数据索引取回来**，这是一个相对较慢的过程。

如果我们把数据放在 Redis 中，也就是直接放在内存之中，让服务端**直接去读取内存中的数据**，那么这样速度明显就会快上不少，并且会极大减小数据库的压力，但是使用内存进行数据存储开销也是比较大的，限于成本的原因，一般我们**只是使用 Redis 存储一些常用和主要的数据**，比如用户登录的信息等。

一般而言在使用 Redis 进行存储的时候，我们需要从以下几个方面来考虑：

- **业务数据常用吗？命中率如何？**如果命中率很低，就没有必要写入缓存；
- **该业务数据是读操作多，还是写操作多？**如果写操作多，频繁需要写入数据库，也没有必要使用缓存；
- **业务数据大小如何？**如果要存储几百兆字节的文件，会给缓存带来很大的压力，这样也没有必要；

在考虑了这些问题之后，如果觉得有必要使用缓存，那么就使用它！使用 Redis 作为缓存的读取逻辑如下，

```php
if(ReadRedis){
  ReadSuess;
}else{
  ReadFail;
  ReadDataFromDB;
  WriteDataToRedis;
}
```

1. 当**第一次读取数据的时候**，读取 Redis 的数据就会失败，此时就会触发程序读取数据库，把数据读取出来，并且写入 Redis 中；
2. 当**第二次以及以后需要读取数据时**，就会直接读取 Redis，读到数据后就结束了流程，这样速度就大大提高了。

写操作流程：
先写入数据库，再写入Redis，所以写操作次数大于读操作的时候，就没必要使用Redis

**会话缓存**(session cache) : 把session存在 redis里，比如购物车信息。

**全页缓存**(FPC): 直接把页面缓存到redis中，

### 消息队列

Redis提供List和Set操作，使得Redis能作为一个比较好的消息队列平台来使用，但是如果涉及到专业的，对消息的可靠性要求非常高的（比如订单信息）就需要用专业的消息队列。

### 排行榜/计数器

redis提供集合的操作做用户分数的递增之类的，原子性的递增。

### 秒杀

秒杀其实经常会出现的问题包括：

- 并发太高导致程序阻塞。
- 库存无法有效控制，出现超卖的情况。

两个解决方案：

- 数据尽量缓存,阻断用户和数据库的直接交互。
- 通过锁来控制避免超卖现象。

redis具体操作：

1. 提前预热数据，放入Redis
2. 商品列表放入Redis List
3. 商品的详情数据 Redis hash保存，设置过期时间
4. 商品的库存数据Redis sorted set保存
5. 用户的地址信息Redis set保存
6. 订单产生扣库存通过Redis制造分布式锁，库存同步扣除
7. 订单产生后发货的数据，产生Redis list，通过消息队列处理
8. 秒杀结束后，再把Redis数据和数据库进行同步

## 注意事项

考虑网络，限制住redis的不是单线程cpu，是网络。
5条set和一个mset的时间复杂度一样，但是前者需要发起多次网络请求。此时便可以使用Redis提供的pipelining功能来实现在一次交互中执行多条命令。

redis的事务不支持回滚。

单线程的redis，当执行某个命令耗时比较长的时候，会拖慢其他命令。
redis提供了Slow Log功能，可以自动记录耗时较长的命令。

当同一秒内有大量key过期时，也会引发Redis的延迟。在使用时应尽量将key的失效时间错开。

