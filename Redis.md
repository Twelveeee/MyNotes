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

### String（字符串）

```sql
-- 语法
COMMAND KEY_NAME

SET redis "key-value数据库"
GET redis
"key-value数据库"
```

一个键最大能存储512mb

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

