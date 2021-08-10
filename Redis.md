# Redis

Redis 官网：https://redis.io/

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

| 命令                   | 描述                                                       |
| ---------------------- | ---------------------------------------------------------- |
| SET key value          | 设置指定 key 的值                                          |
| GET key                | 获取指定 key 的值。                                        |
| GETRANGE key start end | 返回 key 中字符串值的子字符                                |
| GETSET key value       | 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。 |
| MGET key1 [key2..]     | 获取所有(一个或多个)给定 key 的值。                        |
| SETNX key value        | 只有在 key 不存在时设置 key 的值。                         |
| ...                    | ...                                                        |

### Hash（哈希）

Redis hash 是一个键值(key=>value)对集合。
Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

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

| 命令                     | 描述                                    |
| ------------------------ | --------------------------------------- |
| HDEL key field1 [field2] | 删除一个或多个哈希表字段                |
| HEXISTS key field        | 查看哈希表 key 中，指定的字段是否存在。 |
| HGET key field           | 获取存储在哈希表中指定字段的值。        |
| HGETALL key              | 获取在哈希表中指定 key 的所有字段和值   |
| HKEYS key                | 获取所有哈希表中的字段                  |
| HLEN key                 | 获取哈希表中字段的数量                  |
| ...                      | ...                                     |

### 列表(List)

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

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
| BLPOP key1 [key2 ] timeout            | 移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| BRPOP key1 [key2 ] timeout            | 移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。 |
| LINDEX key index                      | 通过索引获取列表中的元素                                     |
| LINSERT key BEFORE\|AFTER pivot value | 在列表的元素前或者后插入元素                                 |
| LPOP key                              | 移出并获取列表的第一个元素                                   |
| LPUSH key value1 [value2]             | 将一个或多个值插入到列表头部                                 |
| ...                                   | ...                                                          |

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
