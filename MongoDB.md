# MongoDB是啥

# MongoDB基础知识

## 基本概念

- 文档型数据库
- 逻辑上按照DB/collection/document组织
- BD:数据库
- collection：类似于关系数据库中的table
- doc：类似于关系数据库中的row

SQL和MongoDB概念区别

| SQL         | MongoDB     | 解释                          |
| ----------- | ----------- | ----------------------------- |
| database    | database    | 数据库                        |
| table       | collection  | 表/集合                       |
| row         | document    | 行/文件                       |
| column      | field       | 列/域                         |
| index       | index       | 索引                          |
| table joins |             | 表连接，mongodb不支持         |
| primary key | primary key | 主键，mongodb自动设置ID为主键 |

![image-20210106225600798](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210106225600798.png)

## 数据类型

| 数据类型           | 描述                                                       |
| ------------------ | ---------------------------------------------------------- |
| String             | 字符串，mongoDB中UTF-8才是合法的                           |
| Integer            | 整形                                                       |
| Boolean            | 布尔                                                       |
| Double             | 双精度浮点数                                               |
| Min/Max keys       | 将一个值与BSON（二进制的JSON）元素最低值和最高值对比       |
| Array              | 数组                                                       |
| Timestamp          | 时间戳                                                     |
| Object             | 用于内嵌文档                                               |
| Null               | 用于创建空值                                               |
| Symbol             | 符号，基本等同于字符串，但是用于一般采用特殊符号类型的语言 |
| Date               | 日期，UNIX时间格式                                         |
| Object ID          | 对象ID，用于创建文档的ID                                   |
| Binary Data        | 二进制数据，用于存二进制数据                               |
| Code               | 代码类型，用于在文档中存储JavaScript代码                   |
| Regular expression | 正则表达式类型                                             |

**Object ID**
类似于唯一主键，可以很快的去生成和排序，包含12bytes，含义是:
前面四个字节是时间戳，接着是三个机器标识码，接着俩个是进程ID组成的PID，最后三个字节是随机数。

## 数据库基本操作

1. 显示数据库
   `show dbs `
   `show databases `

2. 切换到xxx数据库（第一次使用就是创建xxx数据库）
   `use xxx`

3. 创建集合
   `db.createCollection("t_student")`

4. 显示集合列表
   `show collections`

5. 插入doc数据
   `db.t_student.insert({"name":"asd","age":18})`

6. 显示数据集合
   `db.t_student.find()`

   

   **实例**

   插入文档

   ```
   db.book.insert(
   {
   	title:"my first blog post",
       published:new Date(),
       tags:["NoSQL","MongoDB"],
       type:"Work",
       author:"James",
       viewCount:25,
       commentCount:2
   })
   
   ---console---
   WriteResult({"nInserted":1})
   ```

   查询操作

   ```
   db.book.find({"_id":ObjectID("0123456789ab")})
   db.book.find({author:"James"})
   
   ---console---
   {
   	_id:ObjectID("0123456789ab"),
   	title:"my first blog post",
       published:ISODate("2020-01-6T11:11:11.553Z"),
       tags:[
       	"NoSQL",
       	"MongoDB"
       ],
       type:"Work",
       author:"James",
       viewCount:25.0,
       commentCount:2.0
   }
   ```

   删除文档

   ```
   db.book.remove({"_id":ObjectID("0123456789ab")})
   
   ---console---
   WriteResult({"nRemoved":1})
   ```

   更新文档

   ```
   db.book.update(
   	{"_id":ObjectID("0123456789ab")},
   	{"$set":
   		{
   		"viewCount":3
   		}
   	}
   )
   
   ---console---
   WriteResult({"nMatched:1","nUpdate":0,"nModified":1})
   ```

   删除字段

   ```
   db.book.update(
   	{"_id":ObjectID("0123456789ab")},
   	{"$unset":
   		{
   		"viewCount":""
   		}
   	}
   )
   ```

   

   ## MongoDB应用场景

   **适用场景**

- 灵活多样化的数据存储（半结构化、无结构化）

- 快速开发、迭代

- 高性能易扩展

- 弱一致性

  **不适用的场景**

- 需要大量join表操作
- 复杂的查询
- 强一致性的事务



# 索引

**数据库索引就是一种加快海量数据查询的关键技术**

数据库索引的作用和拼音目录是一样的，就是最快速的锁定目标数据所在的位置范围。

## MongoDB的索引类型和使用

### 单字段索引 (Single Field Index)

顾名思义：即单个字段上创建的索引。

单字段索引，其能加速对该字段的各种查询请求，是最常见的索引形式，DDS默认创建的_id索引就是这种类型。

考虑如下的例子:

` db.person.createIndex( {age: 1} )` 

`{age: 1} `代表升序索引，也可以通过`{age: -1}`来创建降序索引，对于单字段索引，升序/降序效果是一样的，因为DDS可以再任一方向上遍历索引。

### 复合索引 (Compound Index)

复合索引是单字段索引的升级版本，它是针对多个字段联合创建索引。

复合索引的顺序很重要，索引会先按第一个字段排序，第一个字段相同的文档按第二个字段排序，依次类推。

考虑如下的例子:

` db.person.createIndex( {age: 1, name: -1} ) `

索引会先以age对人群进行排序，然后对于age相同的人，以name逆序排序。

### 多键索引 (Multikey Index)

当索引的字段为数组时，创建出的索引称为多键索引。

多键索引会为数组的每个元素建立一条索引。DDS检查到需要创建索引的字段为数组，会自动创建多键索引，不需要显式指定。

考虑如下数据：

` {"_id" : "1"), "name" : "jack", "age" : 19, habbit: [“football, runnning”]} } `

` { "_id" : "2"), "name" : "rose", "age" : 20, habbit: [“movie, football”]} } `

对habbit字段创建索引

` db.person.createIndex( {habbit: 1} ) // 自动创建多key索引`

### 地理索引 (Geospatial Index)

地理索引(Geospatial Indexes)：DDS的地理空间索引可以帮助我们在包含地理空间形状和点集的结合上高效地执行空间查询。

考虑如下数据，记录所有餐馆的位置：

```json
{location: {type: "Point", coordinates: [-73.856077, 40.848447]}, name: "KFC"}
{location: {type: "Point", coordinates: [-73.856077, 40.848447]}, name: "McDonald"}
```

如果需要查询离家最近的餐馆，则可以对coordinates创建地理索引

` db.restaurants.createIndex({ location: "2dsphere" })`

### 文本索引 (Text Index)

DDS支持在字符串内容上执行文本检索的查询操作。为了执行文本检索，DDS使用 text index 和 $text 操作符。文本索引能解决快速文本查找的需求。

考虑如下数据，记录所有餐馆的位置：

```json
{ _id: 1, name: "Java Hut", description: "Coffee and cakes" }
{ _id: 2, name: "Burger Buns", description: "Gourmet hamburgers" }
{ _id: 3, name: "Coffee Shop", description: "Just coffee" }
```

如果需要查询name包含java 和 coffee的记录，可以对name字段创建文本索引

` db.stores.createIndex( { name: "text"} )`

•然后使用查询

` db.stores.find( { $text: { $search: "java coffee" } } )`

## 索引的属性

DDS除了支持多种不同类型的索引，还能对索引定制一些特殊的属性。

**唯一索引(unique index)**：保证索引对应的字段不会出现相同的值,
例： `db.persons.createIndex ({ name: 1 },{ unique: true } )`

**TTL索引**：可以针对某个时间字段，指定文档的过期时间,例：
`db.persons.createIndex( { "lastModifiedDate": 1 }, { expireAfterSeconds: 3600 } )`

**部分索引** **(partial index):** 只针对符合某个特定条件的文档建立索引,
例：`db.persons.createIndex({name:1},{partialFilterExpression:{age: {$gt:25}}})`

**稀疏索引(sparse index):** 就是只包含有索引字段的文档的条目，跳过索引键不存在的文档,
例：`db.persons.createIndex( { “lovers": 1 }, { sparse: true } )`

## 索引操作

```sql
--加索引
db.products.createIndex( { "item": 1, "stock": 1 } )

--查索引
db.products.getIndexes()
db.products.getIndex( { "item": 1, "stock": 1 } )

--删索引
db.products.dropIndex( { "item": 1, "stock": 1 } )

--加索引
db.products.createIndex( { "item": 1} )

--观察索引变化
db.products.getIndexes()

--这里应该显示使用了item索引
db.products.find({ "item": "Banana"}).explain();


```



# 认证和权限管理

## 安全简介

![image-20210109161748611](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109161748611.png)

**基本概念**
**资源**：一个资源可以是一个数据库、集合、或者一个集群
**动作**：是指对资源的一个执行行为，比如读取表、读取数据库，其中读取便是一个动作。
**权限**：权限指的是对某类或某一些资源执行某些动作的允许
**角色**：系统中的角色，通常是代表了一种权力等级的象征，比如论坛中的管理员、版主、游客等等，就是角色；系统定义中，角色往往代表一组权限的集合。
**用户**：可登录系统的实体，一个用户通常可被赋予多个角色
**认证**：服务端用来验证客户端的身份，只有经过合法认证的用户才能访问数据库。

## 用户认证

![image-20210109161931650](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109161931650.png)

**认证方法**：
``db.auth(“username”，“pwd”);`
**认证机制**：
SCRAM
x.509
**认证数据库**：
用户登录的时候都要指定认证登录的数据库

## 用户和角色管理

![image-20210109162038416](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109162038416.png)

一个用户想要访问数据库，需要赋予相关的Role,角色就是对一些特定资源可以操作的命令集合。

创建一个用户，并给用户读数据库reporting的权限

```json
db.createUser(
{
	user: "reportsUser",
	pwd: "12345678",
	roles: [
		{ role: "read", db: "reporting" },
		]
}
)
```

创建一个角色，赋予该角色执行killop和killcursors的权限。

```json
db.createRole(
	{
	role: "manageOpRole",
	privileges: [
		{resource:{cluster: true },actions: [ "killop", "inprog" ] },
		{resource: { db: "", collection: "" }, actions: [ "killCursors" ] }
	],
	roles: []
	}
)
```



## 常用角色

**数据库访问：**

| 角色名称  | 拥有权限                 |
| --------- | ------------------------ |
| read      | 允许读取指定数据库的角色 |
| readWrite | 允许读写指定数据库的角色 |

**数据库管理**：

| 角色名称  | 拥有权限                                                     |
| --------- | ------------------------------------------------------------ |
| dbOwner   | 数据库拥有者(最高)，集合了dbAdmin/userAdmin/readWrite角色权限 |
| dbAdmin   | 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile |
| userAdmin | 允许管理当前数据库的用户，如创建用户、为用户授权             |

**数据库通用角色**：

| 角色名称             | 拥有权限                 |
| -------------------- | ------------------------ |
| readAnyDatabase      | 允许读取所有数据库       |
| readWriteAnyDatabase | 允许读写所有数据库       |
| userAdminAnyDatabase | 允许管理所有数据库的用户 |
| dbAdminAnyDatabase   | 允许管理所有数据库       |

# 副本集、集群部署

## MongoDB副本集

### MongoDB副本集架构说明

MongoDB副本集采用一主多备模式

![image-20210109163710996](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109163710996.png)

**主节点Primary**：提供读写能力
**备节点Secondary**：数据副本，可以选举为Primary，提高可用性
**其他节点角色**
Hidden：优先级为0
Arbiter：仅选举能力

```JSON
rs.status() //查看副本集信息
```

### 副本集实例创建

### 副本集实例详情及使用



## MongoDB集群

### MongoDB集群架构说明

MongoDB集群包含MongoS、Config、Shard节点
![image-20210109165005053](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109165005053.png)

**mongos**：查询路由
**config**：集群元数据（副本集构成）
**shard**：数据分片（副本集构成）
不同shard数据不同
分片数据组成数据全集

### 华为云DDS集群实例创建

### 华为云DDS集群实例详情及使用

# Oplog介绍和使用

## **Oplog是什么**

Oplog就是一张Capped表，位于local库下。记录了用户的操作，用于主备间的同步。

**Capped表的概念**

MongoDB中有一种特殊类型的集合，值得特别留意，那就是固定集合（capped collection）
**特点如下**：
大小有限制
随着写入，老数据会不断淘汰。类似于环形队列
**适用场景**：
非常适合记录日志的场景，或者，关注实时数据的场景。

## **Oplog的机制和作用**

Oplog的定位和作用：Oplog就是一张Capped表，位于local库下。记录了用户的操作，用于主备间的同步。

![image-20210109172615078](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109172615078.png)

## Oplog的使用

```json
//主备延时的查询
db.printSlaveReplicationInfo()

//记录操作的审计 和 追踪近期数据的变化
db.oplog.rs.find().sort({$natural:-1}).limit(1).pretty()

//查询oplog表
use local
db.oplog.rs.find().sort({$natural:-1})   --- 倒序查询
```

## **ChangeStream**

Change stream允许应用实时获取mongodb数据的变更功能，可以用于ETL、跨平台数据同步、通知服务等。
ChangeStream本质上，就是在数据库内部通过读取汇总oplog，用更优雅地方式展现出来。

```json
use mydb
var cs = db.mycol.watch()
cs.next()
```

## 注意事项

**注意点：**

- 备延时过大（未脱节）
  主：[10-20]，备：[2-12]
- 主备脱节
  主：[15-20]，备：[2-12]

**解决方法：**
1，合理调整业务负载
2，实时跟踪监控，及时调整
3，根据实际业务，调整oplog的大小
命令：`db.adminCommand({replSetResizeOplog: 1, size: 16000})`
链接：https://docs.mongodb.com/manual/tutorial/change-oplog-size/

# 高可用性机制

## DDS MongoDB 副本集架构

华为云DDS 经典副本集架构采用PSA（Primary+Secondary+Arbiter）经典架构，支持数据冗余和读写分离的特性。

![image-20210109175031281](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109175031281.png)

**主节点（Primary）**
•接收所有的写请求，然后把修改同步到所有Secondary。
•一个Replica Set只能有一个Primary节点，当Primary挂掉后，其他Secondary或者Arbiter节点会重新选举出来一个主节点。
•默认读请求也是发到Primary节点处理的，可以通过修改客户端连接配置以支持读取Secondary节点。

**副本节点（Secondary）**与主节点保持同样的数据集。当主节点挂掉的时候，参与选主。

**仲裁者（Arbiter）**
•不保有数据，不参与选主，只进行选主投票。
•使用Arbiter可以减轻数据存储的硬件需求，Arbiter几乎没什么大的硬件资源需求，但重要的一点是，在生产环境下它和其他数据节点不要部署在同一台机器上。

## DDS MongoDB 集群架构

华为云DDS 经典集架构采用MongoS+ConfigServer+Shard部署方式。

![image-20210109175242639](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109175242639.png)

**MongoS节点**
•用户访问的路由节点，用户应用直接访问此节点，此节点不做存储，只做指令路由。
•MongoS节点冗余保障用户访问集群数据的可靠性。

**ConfigServer节点**
•集群的大脑，保存着集群和分片的元数据，也是副本集结构，各个分片包含哪些数据信息。
•MongoS拷贝ConfigServer上的分片信息到本地，然后将客户端请求分发到不同的Shard节点。
**Shard**
就是分片，每个分片都是一个副本集，通过副本集结构确保每一个分片数据的可靠性。



## MongoDB 高可用原理

副本集节点架构中，需要解决两方面的问题一致性和容错性的问题，MongoDB从3.2版本开始引入Raft算法解决这类问题（该版本之前，MongoDB采用Bully算法解决选主问题），故Raft算法是MongoDB实现高可用性的核心算法。

在Raft中，节点有三种角色：
**Leader**：负责接收客户端的请求，将日志复制到其他节点并告知其他节点何时应用这些日志是安全的。
**Candidate**：用于选举Leader的一种角色。
**Follower**：负责响应来自Leader或者Candidate的请求。

通过三种角色的关系转换，解决选举角色的问题，如下：

![image-20210109175425923](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109175425923.png)

## Oplog / ChangeStream容灾备份能力演进

### Oplog

Oplog是local库下的一个固定集合（Capped Collection）即：`local.oplog.rs`
记录主从节点复制的信息。副本集节点架构下，数据的复制同步就是通过Oplog完成。
Oplog格式如下：

```json
•ts: 8字节的时间戳，由4字节unixtimestamp + 4字节自增计数表示。在选举(如master宕机时)新primary时，会选择ts最大的那个secondary作为新primary
•op：1字节的操作类型
	•"i"：insert
	•"u"：update
	•"d"：delete
	•"c"：dbcmd
	•"db"：声明当前数据库(其中ns 被设置成为=>数据库名称+'.')
	•"n": no op,即空操作，其会定期执行以确保时效性
•ns：操作所在的namespace
•o：操作所对应的document，即当前操作的内容（比如更新操作时要更新的的字段和值）
•o2: 在执行更新操作时的where条件，仅限于update时才有该属性
```

### ChangeStream

从MongoDB 3.6版本开始，引入了一个重要特性：**ChangeStream**。
为解决集群场景下，Oplog数据的先后顺序问题，MongoDB引入了逻辑时钟，并在此基础上增加ChangeStream。
相比自动Oplog，ChangeStream 有以下优点：
•如果只有单个节点持久化，那么oplog对应的操作是可能被回滚的，而change stream有Durability特性
•在集群场景下，change stream支持跨shards的数据同步,而Oplog不行，在跨Shards场景下会导致数据DML先后顺序错乱；

# 事务处理

## 事务四要素(ACID)

**数据库事务**
数据库事务（简称：事务）是数据库管理系统执行过程中的一个逻辑单位，由一个有限的数据库操作序列构成

**A(tomic)**
一个事务（transaction）中的所有操作，或者全部完成，或者全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。即，事务不可分割、不可约简。

**C(onsistency)**
在事务开始之前和事务结束以后，数据库的完整性没有被破坏。事务执行未违背已存在约束(触发器/主外键约束/CDC顺序)

**I(solation)**
数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括未提交读（Read uncommitted）、提交读（read committed）、可重复读（repeatable read）和串行化（Serializable）

**D(urability)**
事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

## 事务四要素在MongoDB中的体现

**A(tomic)**
单文档事务中，数据/索引/对应的Oplog的写入是原子的
多文档事务中，每条文档的数据/索引/oplog的写入是原子的

**C(onsistency)**
文档被更新/插入后，对应的索引/Oplog也一并可见，并且数据在Oplog中的顺序按照commitTimestamp顺序

**I(solation)**
多文档事务默认SnapshotIsolation的隔离级别
支持readCommitted/SnapshotIsolation两种

**D(urability)**
由writeConcern指定
{w:value, j:bool}
	w:数据复制到几个节点之后返回成功
	j: 是否等待数据库写journal(write ahead log)后返回

## MongoDB Session&Transaction

Session 是Server 端对象，封装了事务上下文, 持久化存放在config.system.sessions表中
Session 用来维持RetryableWrite和Multi-Document Transaction 上下文
RetryableWrite
	•写操作失败后可重试，由服务端判断是否已经执行成功
	•Exactly Once Delivery

![image-20210109181716411](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109181716411.png)

每个retryableWrite的txnNumber递增
数据库记录上一次执行成功的<lsid, txnNumber>
<lsId, txnNumber> 在客户端和数据库之间传递
数据库通过客户端传递的lsId, txnNumber判断是否已经执行过了

## Transaction

开启一个Session

```sql
//Start a session.
session = db.getMongo().startSession({ readPerference:{mode:"primary"}})

employeesCollection = session.getDatabase("hr").employees;
eventsCollection = session.getDatabase("reporting").events;
```

开启一个事务

```sql
//Start a transaction
session.startTransaction({ readConcern:{ level:"snapshot"},writeConcern:{w:"marjor"}})
```

执行事务操作

```sql
//Operations inside the transaction
try{
	employeesCollection.uppdateOne({employee:3},{$set:{status:"Inactive"}});
	eventsCollection.insertOne({employee:3,status:{new:"Inactive",old:"Active"}});
}catch(error){
	//abort trnsaction on error
	session.abortTransaction();
	throw error;
}
```

提交事务，关闭Session

```sql
//Commit the transaction using write concern set at transaction start
session.commitTransction();

session.endSession();
```

## MongoDB Transaction Limit

•事务默认最大时长60s (从begin 到abort)
•Mongo的事务是no-steal,no-force策略，提交前不持久化
•过大的事务会OOM

## MongoDB事务与复制(副本集多文档事务)

Abort的事务不生成Oplog
事务在Commit时批量生成OplogChain并插入Oplog表
从节点复制OplogChain并应用

OplogChain
单条Oplog最大16MB
一条Oplog放不下整个事务的操作
通过prevOpTime字段将事务完整的Oplog串起来

![image-20210109182912152](https://twelveeee-note.oss-cn-beijing.aliyuncs.com/Image/image-20210109182912152.png)

## MongoDB 2PC事务协议

基于HLC(混合逻辑时钟)
基于CLOCK-SI算法
	Prepare阶段
		商量出所有参与节点中最大的PrepareTs最为CommitTs
	Commit阶段	
		每个节点以CommitTs作为提交时间戳执行提交