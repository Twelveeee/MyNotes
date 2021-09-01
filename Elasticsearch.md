# Elasticsearch

[Elasticsearch: 权威指南 中文版](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)

[elastic官网](https://www.elastic.co/cn/)

## 简介

*Elasticsearch* 是一个实时的分布式搜索分析引擎，它能让你以前所未有的速度和规模，去探索你的数据。

它被用作全文检索、结构化搜索、分析以及这三个功能的组合：

Wikipedia 使用 Elasticsearch 提供带有高亮片段的全文搜索，还有 search-as-you-type 和 did-you-mean 的建议。
卫报 使用 Elasticsearch 将网络社交数据结合到访客日志中，为它的编辑们提供公众对于新文章的实时反馈。
Stack Overflow 将地理位置查询融入全文检索中去，并且使用 more-like-this 接口去查找相关的问题和回答。
GitHub 使用 Elasticsearch 对1300亿行代码进行查询。

## 基础

### 安装

略

### 使用demo

如果你正在使用 Java，在代码中你可以使用 Elasticsearch 内置的两个客户端：

**节点客户端（Node client）**
节点客户端作为一个非数据节点加入到本地集群中。换句话说，它本身不保存任何数据，但是它知道数据在集群中的哪个节点中，并且可以把请求转发到正确的节点。
**传输客户端（Transport client）**
轻量级的传输客户端可以将请求发送到远程集群。它本身不加入集群，但是它可以将请求转发到集群中的一个节点上。
两个 Java 客户端都是通过 9300 端口并使用 Elasticsearch 的原生 传输 协议和集群交互。集群中的节点通过端口 9300 彼此通信。如果这个端口没有打开，节点将无法形成一个集群。

如果用的是其他语言，那么Elasticsearch支持使用RESTful API 通过端口 *9200* 与 Elasticsearch 进行通信，可以用web客户端，甚至可以使用curl命令。

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：

```js
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

被 `< >` 标记的部件：

| `VERB`         | 适当的 HTTP *方法* 或 *谓词* : `GET`、 `POST`、 `PUT`、 `HEAD` 或者 `DELETE`。 |
| -------------- | ------------------------------------------------------------ |
| `PROTOCOL`     | `http` 或者 `https`（如果你在 Elasticsearch 前面有一个 `https` 代理） |
| `HOST`         | Elasticsearch 集群中任意节点的主机名，或者用 `localhost` 代表本地机器上的节点。 |
| `PORT`         | 运行 Elasticsearch HTTP 服务的端口号，默认是 `9200` 。       |
| `PATH`         | API 的终端路径（例如 `_count` 将返回集群中文档数量）。Path 可能包含多个组件，例如：`_cluster/stats` 和 `_nodes/stats/jvm` 。 |
| `QUERY_STRING` | 任意可选的查询字符串参数 (例如 `?pretty` 将格式化地输出 JSON 返回值，使其更容易阅读) |
| `BODY`         | 一个 JSON 格式的请求体 (如果请求需要的话)                    |

例子

```js
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```

Elasticsearch 返回一个 HTTP 状态码（例如：`200 OK`）和（除`HEAD`请求）一个 JSON 格式的返回值。前面的 `curl` 请求将返回一个像下面一样的 JSON 体：

```js
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```

在返回结果中没有看到 HTTP 头信息是因为我们没有要求`curl`显示它们。想要看到头信息，需要结合 `-i` 参数来使用 `curl` 命令：

```js
curl -i -XGET 'localhost:9200/'
```


Elasticsearch 使用 JavaScript Object Notation（ [*JSON*](http://en.wikipedia.org/wiki/Json)）作为文档的序列化格式。

demo

输入数据：我们可以通过一条命令完成所有这些动作：

```js
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

url：
**`megacorp`** ：索引
**`employee`**：类型
**`1`**：特定雇员的ID

检索文档：

```js
GET /megacorp/employee/1

//return 
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```



## 数据操作 

### 集群健康

运行`_cat` API

```js
curl -X GET "localhost:9200/_cat/health?v"
```

返回内容：

```js
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

`status` 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：
**`green`**:所有的主分片和副本分片都正常运行。
**`yellow`**:所有的主分片都正常运行，但不是所有的副本分片都正常运行。
**`red`**:有主分片没能正常运行。

还可以获取集群中节点列表

```js
curl -X GET "localhost:9200/_cat/nodes?v&pretty"
```

```js
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           10           5   5    4.46                        mdi      *      PB2SGZY
```

有个名为`PB2SGZY`的节点，是集群中的单个节点。

列出所有index

```js
curl -X GET "localhost:9200/_cat/indices?v&pretty"
```

```js
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
//第二行是空，说明当前没有index
```



### 文档

就是一个存储的内容，类似于document DB / mongoBD
一条信息的存储是以文档进行存储的。一次存一条文档。

`_index`：文档在哪存放
`_type`：文档表示的对象类别（弃用）
`_id`：文档唯一标识

### 创建索引

```js
PUT /customer?pretty
GET /_cat/indices?v
```

```js
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b
```

回应的结果显示，现在又一个名为`customer`的index，有5个主分片和1个副本（默认值），其中有0个文档。
health是黄色，表明没有备份或者其他什么问题。因为这个新建的index只有一个节点在运行，在另外一个节点加入es集群之前，这个集群是没有备份的，如果加入了第二个节点，副本一分配，这个节点的健康状态就会变成绿色。



### 新建文档

新建文档，使用自己的id

```js
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

如果不输入{id}，会新建一个

如果index不存在，es会自动创建一个index。

```js
GET /customer/_doc/1?pretty
```

```js
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```

当我们索引一个文档，怎么确认我们正在创建一个完全新的文档，而不是覆盖现有的呢？
请记住， _index 、 _type 和 _id 的组合可以唯一标识一个文档。

```js
PUT /website/blog/123?op_type=create
{ ... }

PUT /website/blog/123/_create
{ ... }
```

如果创建新文档的请求成功执行，Elasticsearch 会返回元数据和一个 `201 Created` 的 HTTP 响应码。

另一方面，如果具有相同的 `_index` 、 `_type` 和 `_id` 的文档已经存在，Elasticsearch 将会返回 `409 Conflict` 响应码，以及如下的错误信息：

```js
{
   "error": {
      "root_cause": [
         {
            "type": "document_already_exists_exception",
            "reason": "[blog][123]: document already exists",
            "shard": "0",
            "index": "website"
         }
      ],
      "type": "document_already_exists_exception",
      "reason": "[blog][123]: document already exists",
      "shard": "0",
      "index": "website"
   },
   "status": 409
}
```

### 删除文档

**删除索引**

```js
DELETE /customer?pretty
```

和删除文档一样

```js
DELETE /website/blog/123
```

如果找到该文档，Elasticsearch 将要返回一个 `200 ok` 的 HTTP 响应码，和一个类似以下结构的响应体。注意，字段 `_version` 值已经增加:

```js
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```

如果文档没有找到，我们将得到 `404 Not Found` 的响应码和类似这样的响应体：

```js
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```



即使文档不存在（ `Found` 是 `false` ）， `_version` 值仍然会增加。这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。

### 更新文档

在 Elasticsearch 里文档是 *不可改变* 的，不能修改它们。相反，如果想要更新现有的文档，需要 *重建索引* 或者进行替换，比如删掉一个再新建一个。

```sense
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```

在响应体中，我们能看到 Elasticsearch 已经增加了 `_version` 字段值：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 2,
  "created":   false 
}
```

`created` 标志设置成 `false` ，是因为相同的索引、类型和 ID 的文档已经存在。

在内部，Elasticsearch 已将旧文档标记为已删除，并增加一个全新的文档。 尽管你不能再对旧版本的文档进行访问，但它并不会立即消失。当继续索引更多的数据，Elasticsearch 会在后台清理这些已删除文档。

也可以使用简单的脚本执行更新，例如将年龄+5

```js
POST /customer/_doc/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```

### 简单查询文档

为了从 Elasticsearch 中检索出文档，我们仍然使用相同的 `_index` ,  `_id` ，但是 HTTP 谓词更改为 `GET` :

**参数方式请求**

```js
GET /bank/_search?q=*&sort=account_number:asc&pretty
```

`q=\*` 参数指示es匹配索引中的所有文档
`sort=account_number:asc`尝试知名对account_number升序排序，
`pretty`表示返回便于识别的json字符串

```js
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}

```

响应体：
`took`: es的执行搜索时间 单位ms
`time_out`:是否超时
`_shards`:搜索了多少个分片，搜索成功和失败的分片数。
`hits`:搜索结果
`hits.total`:搜索结果的数量
`hits.hits`:实际搜索结果数组
`hits.sort`:结果的排序键

不同于上面的参数方式请求，这是请求体方式请求，就是把查询的内容放入json body中。

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

**请求体方式请求**

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
  "from": 10,
  "size": 5
}
```

`match_all`:查询索引内部所有文档。
`sort`:排序
`from、size`:类似于mysql `limit 10,5`
如果size未指定，默认是10，



**返回一部分文档**

默认情况下， `GET` 请求会返回整个文档，这个文档正如存储在 `_source` 字段中的一样。

```js
GET /website/blog/123?_source=title,text
```

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```

如果只要source字段，不要元数据

```js
GET /website/blog/123/_source
```

```js
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```

bool查询

```js
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

`must`子语句里面的所有条件都必须满足才是匹配的数据。

相反的，还有一个`must_not`

```js
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

`must_not`内的语句必须全不满足才是匹配的数据。

`bool`里允许混合条件，也就是同时使用must和must_not、should

```js
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

`bool`里还有`filter`

用于过滤文档

```js
//eg 选取余额大于20000，小于30000的文档
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

聚合(group by)

对数据进行分组和统计信息，就跟mysql一样，近似于sql group by

```js
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

等同于

```sql
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC LIMIT 10;
```

response

```js
{
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped" : 0,
    "failed": 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets" : [ {
        "key" : "ID",
        "doc_count" : 27
      }, {
        "key" : "TX",
        "doc_count" : 27
      }, {
        "key" : "AL",
        "doc_count" : 25
      }, {
        "key" : "MD",
        "doc_count" : 25
      }, {
        "key" : "TN",
        "doc_count" : 23
      }, {
        "key" : "MA",
        "doc_count" : 21
      }, {
        "key" : "NC",
        "doc_count" : 21
      }, {
        "key" : "ND",
        "doc_count" : 21
      }, {
        "key" : "ME",
        "doc_count" : 20
      }, {
        "key" : "MO",
        "doc_count" : 20
      } ]
    }
  }
}
```

对余额进行排序

```js
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

降序排序平均余额

```js
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```



**检查文档是否存在**

```js
curl -i -XHEAD http://localhost:9200/website/blog/123
```

如果文档存在， Elasticsearch 将返回一个 `200 ok` 的状态码：

```js
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

若文档不存在， Elasticsearch 将返回一个 `404 Not Found` 的状态码：

```js
curl -i -XHEAD http://localhost:9200/website/blog/124
```

```js
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

当然，一个文档仅仅是在检查的时候不存在，并不意味着一毫秒之后它也不存在：也许同时正好另一个进程就创建了该文档。



### 批量查询文档

将多个请求合并成一个，避免单独处理每个请求花费的网络延时和开销。 如果你需要从 Elasticsearch 检索很多文档，那么使用 *multi-get* 或者 `mget` API 来将这些检索请求放在一个请求中，将比逐个文档请求更快地检索到全部文档。

`mget` API 要求有一个 `docs` 数组作为参数，每个元素包含需要检索文档的元数据， 包括 `_index` 、 `_type` 和 `_id` 。如果你想检索一个或者多个特定的字段，那么你可以通过 `_source` 参数来指定这些字段的名字：

```js
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```

该响应体也包含一个 `docs` 数组， 对于每一个在请求中指定的文档，这个数组中都包含有一个对应的响应，且顺序与请求中的顺序相同。 

```js
{
   "docs" : [
      {
         "_index" :   "website",
         "_id" :      "2",
         "_type" :    "blog",
         "found" :    true,
         "_source" : {
            "text" :  "This is a piece of cake...",
            "title" : "My first external blog entry"
         },
         "_version" : 10
      },
      {
         "_index" :   "website",
         "_id" :      "1",
         "_type" :    "pageviews",
         "found" :    true,
         "_version" : 2,
         "_source" : {
            "views" : 2
         }
      }
   ]
}
```

如果想检索的数据都在相同的 `_index` 中（甚至相同的 `_type` 中），则可以在 URL 中指定默认的 `/_index` 或者默认的 `/_index/_type` 。

你仍然可以通过单独请求覆盖这些值：

```js
GET /website/blog/_mget
{
   "docs" : [
      { "_id" : 2 },
      { "_type" : "pageviews", "_id" :   1 }
   ]
}
```

如果所有文档的 `_index` 和 `_type` 都是相同的，你可以只传一个 `ids` 数组，而不是整个 `docs` 数组：

```js
GET /website/blog/_mget
{
   "ids" : [ "2", "1" ]
}
```

即使有某个文档没有找到，上述请求的 HTTP 状态码仍然是 `200` 。事实上，即使请求 *没有* 找到任何文档，它的状态码依然是 `200` --因为 `mget` 请求本身已经成功执行。 为了确定某个文档查找是成功或者失败，你需要检查 `found` 标记。

### 批量操作

与 `mget` 可以使我们一次取回多个文档同样的方式， `bulk` API 允许在单个步骤中进行多次 `create` 、 `index` 、 `update` 或 `delete` 请求。 如果你需要索引一个数据流比如日志事件，它可以排队和索引数百或数千批次。

`bulk` 与其他的请求体格式稍有不同，如下所示：

```js
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```

这种格式类似一个有效的单行 JSON 文档 *流* ，它通过换行符(`\n`)连接到一起。注意两个要点：
每行一定要以换行符(`\n`)结尾， *包括最后一行* 。这些换行符被用作一个标记，可以有效分隔行。
这些行不能包含未转义的换行符，因为他们将会对解析造成干扰。这意味着这个 JSON *不* 能使用 pretty 参数打印。

```js
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } 
```

 `delete` 动作不能有请求体,它后面跟着的是另外一个操作。
最后一个换行符不要落下。

### 并发处理

***悲观并发控制***
这种方法被关系型数据库广泛使用，它假定每个操作都会有冲突发生，因此阻塞访问资源以防止冲突。 一个典型的例子是读取一行数据之前先将其锁住，确保只有放置锁的线程能够对这行数据进行修改。

***乐观并发控制***
Elasticsearch 中使用的这种方法是假定冲突不可能发生，并且不会阻塞正在尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。

Elasticsearch默认使用乐观并发控制，每个文档都有一个 `_version` （版本）号，当文档被修改时版本号递增。 Elasticsearch 使用这个 `_version` 号来确保变更以正确顺序得到执行。如果旧版本的文档在新版本之后到达，它可以被简单的忽略。

利用 `_version` 号来确保 应用中相互冲突的变更不会导致数据丢失。我们通过指定想要修改文档的 `version` 号来达到这个目的。 如果该版本不是当前版本号，我们的请求将会失败。

新建一个文档

```sense
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}
```

我们检索文档:

```sense
GET /website/blog/1
```

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out..."
  }
}
```

当我们尝试通过重建文档的索引来保存修改，我们指定 `version` 为我们的修改会被应用的版本：

```sense
PUT /website/blog/1?version=1 
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```

此请求成功，并且响应体告诉我们 `_version` 已经递增到 `2` ：

```sense
{
  "_index":   "website",
  "_type":    "blog",
  "_id":      "1",
  "_version": 2
  "created":  false
}
```

然而，如果我们重新运行相同的索引请求，仍然指定 `version=1` ， Elasticsearch 返回 `409 Conflict` HTTP 响应码，和一个如下所示的响应体：

```sense
{
   "error": {
      "root_cause": [
         {
            "type": "version_conflict_engine_exception",
            "reason": "[blog][1]: version conflict, current [2], provided [1]",
            "index": "website",
            "shard": "3"
         }
      ],
      "type": "version_conflict_engine_exception",
      "reason": "[blog][1]: version conflict, current [2], provided [1]",
      "index": "website",
      "shard": "3"
   },
   "status": 409
}
```

这告诉我们在 Elasticsearch 中这个文档的当前 `_version` 号是 `2` ，但我们指定的更新版本号为 `1` 。

我们现在怎么做取决于我们的应用需求。我们可以告诉用户说其他人已经修改了文档，并且在再次保存之前检查这些修改内容。 或者，在之前的商品 `stock_count` 场景，我们可以获取到最新的文档并尝试重新应用这些修改。

所有文档的更新或删除 API，都可以接受 `version` 参数，这允许你在代码中使用乐观的并发控制，这是一种明智的做法。





## 映射

***精确值*** 
如它们听起来那样精确。例如日期或者用户 ID，但字符串也可以表示精确值，例如用户名或邮箱地址。对于精确值来讲，`Foo` 和 `foo` 是不同的，`2014` 和 `2014-09-15` 也是不同的。
精确值很容易查询。结果是二进制的：要么匹配查询，要么不匹配。这种查询很容易用 SQL 表示：

```sql
WHERE name    = "John Smith"
  AND user_id = 2
  AND date    > "2014-09-15"
```



***全文*** 
是指文本数据（通常以人类容易识别的语言书写），例如一个推文的内容或一封邮件的内容。

查询全文数据要微妙的多。我们问的不只是“这个文档匹配查询吗”，而是“该文档匹配查询的程度有多大？”换句话说，该文档与给定查询的相关性如ß何？

我们很少对全文类型的域做精确匹配。相反，我们希望在文本类型的域中搜索。不仅如此，我们还希望搜索能够理解我们的 *意图* ：

### 倒排索引

Elasticsearch 使用一种称为 *倒排索引* 的结构，它适用于快速的全文搜索。一个倒排索引由文档中所有**不重复词**的列表构成，对于其中每个词，有一个包含它的文档列表。

例如，假设我们有两个文档，每个文档的 `content` 域包含如下内容：

1. The quick brown fox jumped over the lazy dog
2. Quick brown foxes leap over lazy dogs in summer

```
Term      Doc_1  Doc_2
-------------------------
Quick   |       |  X
The     |   X   |
brown   |   X   |  X
dog     |   X   |
dogs    |       |  X
fox     |   X   |
foxes   |       |  X
in      |       |  X
jumped  |   X   |
lazy    |   X   |  X
leap    |       |  X
over    |   X   |  X
quick   |   X   |
summer  |       |  X
the     |   X   |
------------------------
```

现在，如果我们想搜索 `quick brown` ，我们只需要查找包含每个词条的文档：

```
Term      Doc_1  Doc_2
-------------------------
brown   |   X   |  X
quick   |   X   |
------------------------
Total   |   2   |  1
```

说明`quick brown` 在两个文档里出现的情况。两个文档都匹配，但是第一个文档比第二个匹配度更高。如果我们使用仅计算匹配词条数量的简单 *相似性算法* ，那么，我们可以说，对于我们查询的相关性来讲，第一个文档比第二个文档更佳。

## **分析**

做全文搜索就需要对文档分析、建索引。从文档中提取词元（Token）的算法称为分词器（Tokenizer），在分词前预处理的算法称为字符过滤器（Character Filter），进一步处理词元的算法称为词元过滤器（Token Filter），最后得到词（Term）。这整个分析算法称为分析器（Analyzer）。

文档包含词的数量称为词频（Frequency）。搜索引擎会建立词与文档的索引，称为倒排索引（Inverted Index）。

Analyzer 按顺序做三件事：

1. 使用 CharacterFilter 过滤字符
2. 使用 Tokenizer 分词
3. 使用 TokenFilter 过滤词

每一部分都可以指定多个组件。

剩下的没看懂，晚点在回来

//todo



## 集群的原理

一个运行中的 Elasticsearch 实例称为一个节点，而集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为 *主* 节点时， 它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。 而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。 任何节点都可以成为主节点。我们的示例集群就只有一个节点，所以它同时也成为了主节点。

作为用户，我们可以将请求发送到 *集群中的任何节点* ，包括主节点。 每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。



### 选举原理Bully算法

Elasticsearch采用了master-slave模式， ES会在集群中选取一个节点成为主节点，只有Master节点有资格维护全局的集群状态，在有节点加入或者退出集群的时候，它会重新分配分片，并将集群最新状态发送给集群中其它节点，主节点会以周期性ping的方式以验证其它节点是否存活。Elasticsearch的选举算法基于Bully选举算法，简单的说，在bully算法中，每个节点都有一个编号，只有编号最大的存活节点才能成为master节点。1. 选举时间点

**选举时间点**

1. 某个新节点加入了集群或者某个节点从宕机中恢复
2. 集群中的某个节点检测到leader崩溃

**选举流程**

1. 节点node向所有比自己大的节点发送选举消息(选举为election消息)
2. 如果节点node得不到任何回复(回复为alive消息)，那么节点node成为master，并向所有的其它节点宣布自己是master(宣布为Victory消息)
3. 如果node得到了任何回复，node节点就一定不是master，同时等待Victory消息，如果等待Victory超时那么重新发起选举

**Elasticsearch编号比较**

Elasticsearch编号比较的判断依据有两个，首先是ClusterState版本号的比较，版本号越大优先级越高，然后是节点id的比较，id越小优先级越高。ClusterState是Master向集群中各个节点发送的集群状态，这个状态有一个版本号码，如果集群状态发生了变化，比如集群新增了节点成员或者有节点成员退出了，那么这个版本号就会加一，比对这个版本号的目的是让拥有最新状态的节点成为Master的优先级最高。

### Bully算法的缺陷

**1 Master假死**

Master节点承担的职责负载过重的情况下，可能无法即时对组内成员作出响应，这种便是假死。如果上图中的P6节点假死，于是P5节点成为了Master节点，但是在P6节点负载减轻之后，P6节点又对组内成员作出了响应，P6节点又会成为Master节点，如此反复，整个集群状态就会非常不可靠。

Elasticsearch是如何解决这个问题的呢？在Bully算法中，Master节点P6因为负载重，来不及对P3节点作出响应，所以P3节点通知P4,P5节点进行选举。在Elasticsearch中，P3节点发现Master P6对自己长时间不作出响应，P3节点会请求其它节点判断P6节点是否存活，如果有1/2以上节点都认定P6存活，那么P3就会放弃发起选举

**2 脑裂问题**

脑裂问题指的是一个集群中出现了两个及以上的Master节点。

比如上图中集群因为网络原因分成了两个部分，一个部分称为partition1包含P3,P5,P6节点，另一部分称为partition2包含P2,P1,P4节点，这两个partition因为网络原因比如路由器短时故障造成不能相互通信的问题。

脑裂问题解决方案 ： **Quorum算法**

脑裂问题是所有集群选举算法都要面对的一个问题，Elasticsearch集群机制采用了最小参与节点的方案解决的。假设elasticsearch集群中有资格投票的实例个数是n，节点想要成为master必须要得到n/2 +1票数。上述任何一个master节点所在的分区集群的候选节点数目都小于n/2 +1，更不可能得到n/2 +1个选票，所以整个Elasticsearch集群处于瘫痪状态

我们也可以强制指定elasticsearch节点在有M个候选节点的情况下能选举出一个主节点，但是如果配置数小于上文提到的 n/2 +1 那么会出现脑裂的情况，M的配置参数如下

```text
discovery.zen.minimum_master_nodes
```

如果产生了脑裂情况，为了避免脑裂的Master生成错误数据对整个集群产生影响。Master更新集群状态时还作出了如下防护，Master有两种指令，一种是send指令，另一种是commit指令，Master将最新集群状态推送给其它节点的时候(这是send指令)，Master节点进入等待响应状态，其它节点并不会立刻应用该集群状态，而是首先会响应Master节点表示它已经收到集群状态更新，同时等待Master节点的commit指令。Master节点如果在`discovery.zen.commit_timeout`配置时间内都没有收到`discovery.zen.minimum_master_nodes`个数的节点响应，那么Master节点就不会向其它节点发送commit指令。如果Master收到了足够数量的响应，那么Master会向集群发出提交状态的指令，此时其它节点应用集群最新状态，Master节点再次等待所有节点响应，等待时间为`discovery.zen.publish_timeout`，如果任何一个节点没有发出提交响应，Master再次更新整个集群状态更新。

**3 Master降级**

Master主动降级发生在两种情况。

第一种是master发现自己能连接到的其它节点数目小于n/2 + 1，那么master自动降级为candidate。

第二种是Master在ping其它节点时候，如果发现了其它master，那么当前的master会比较cluster_state的version，如果当前master的version小，那么主动降级为candidate并主动加入另外一个master节点

**4 网络负载问题**

集群中每个节点成员都会维护和其它所有成员的交互，整个集群维护的网络连接的总数是n*(n-1)，如果集群中节点的数目非常的多，那么网络连接数目也会非常的多，网络负载会比较大，但是好在elasticsearch节点数目往往比较少，所以这个缺陷对elasticsearch集群来说不会产生什么影响。

# 与mongoDB的区别

|    MongoDB     |    ElasticSearch     | 备注               |                                                              |
| :------------: | :------------------: | :----------------- | ------------------------------------------------------------ |
|      定位      |    (文档型)数据库    | (文档型)搜索引擎   | 一个管理数据,一个检索数据                                    |
|    资源占用    |         一般         | 高                 | mongo使用c++, es使用Java开发                                 |
|    写入延迟    |          低          | 高                 | es的写入延迟默认1s, 可配置, 但是要牺牲一些东西               |
| 全文索引支持度 |         一般         | 非常好             | es本来就是搜索引擎, 这个没啥可比性                           |
|   有无Schema   |          无          | 无                 | 两者都是无Schema                                             |
|  支持的数据量  |         PB+          | TB+ ~ PB           | 两者支持的量并不好说的太死, 都支持分片和横向扩展, 但是相对来说MongoDB的数据量支持要更大一点 |
|      性能      |        非常好        | 好                 | MongoDB在大部分场景性能比es强的多                            |
|    索引结构    |         B树          | LSM树              | es追求写入吞吐量, MongoDB读写比较均衡                        |
|    操作接口    |         TCP          | Restful(Http)      |                                                              |
|  是否支持分片  |          是          | 是                 |                                                              |
|  是否支持副本  |          是          | 是                 |                                                              |
|    选主算法    |     Bully(霸凌)      | Bully(霸凌)        | 相比于Paxos和Raft算法实现更简单并有一定可靠性上的妥协，但是选举速度比较快 |
|    扩展难度    |         容易         | 非常容易           | es真的是我用过的扩展最方便的存储系统之一                     |
|    配置难度    |          难          | 非常容易           |                                                              |
|    地理位置    |         支持         | 支持               |                                                              |
|    运维工具    |         丰富         | 一般               |                                                              |
|   插件和引擎   | 有多个存储引擎供选择 | 有大量插件可以使用 | -                                                            |

## 定位

`MongoDB`和`Elasticsearch`都属于NoSQL大家族, 且都属于文档型数据存储

所以这两者的很多功能和特性高度重合, 但其实两者定位完全不同

MongoDB 是 **文档型数据库**, 提供 **数据存储和管理服务**
Elasticsearch 是**搜索服务**, 提供 **数据检索服务**

两者的很大区别在于源数据的存储和管理

- MongoDB作为一个数据库产品, 是拥有源数据管理能力的
- Elasticsearch作为一个搜索引擎, 定位是**提供数据检索服务**, 也就是说我只管查, 不管写 _, Elasticsearch的Mapping不可变也是为此服务的, 带来的代价就是`es不适合作为数据管理者`, es可以从其他数据源同步数据过来提供查询, 但是不适合自己对数据进行存储和管理

es更侧重数据的查询, 各种复杂的花式查询支持的很好, 相比来说 MongoDB的查询能力就显得比较平庸了

由此可见, 对于个人, 如果你有一批数据要看, 但是不经常进行修改, 这个时候毫无疑问可以用es, 但是如果你还打算继续修改数据, 最好就是使用MongoDB，但其实对大多数人公司来讲，这两者的数据管理能力并没有多大的影响

es修改Mapping的代价非常高, 所以我们一般都是把新数据重新写入一份新索引，然后直接切换读取的别名到新的索引

## 读写数据的异同

`MongoDB`和`ElasticSearch`都支持全文索引, 虽然MongoDB的全文索引效果完全无法跟es相比(es毕竟是专业的搜索引擎产品, 着重提供数据的检所支持, 这方面吊打MongoDB也是可以理解的)

MongoDB虽然在支持的部分查询功能上稍微弱于es, 但是在大部分场景下性能方面完爆es, 不管是读性能, 还是写性能

es的写入延迟默认为1s, 这个虽然是写入延迟的范畴, 但是毫无疑问是一大缺点, 虽然可以配置为更短的时间, 但是这样就要牺牲一定的数据吞吐量, 会造成更频繁的磁盘刷新操作

es底层使用`Lucene`作为核心引擎, 很多es的设计就是为了匹配Lucene中的概念, 其实es可以看成一个lucene的proxy层包装,将lucene的原生接口封装的更好用, 同时还实现了很多管理和监控等辅助功能, 但是整体来说es上层的模块和lucene的隔阂还是挺明显的, 耦合度上有一定的欠缺

MongoDB则是完整的一个单体数据库产品, 虽然内部的存储引擎也是可插拔式的, 整体而言还是更加的浑然一体

## 部署和资源占用

单机部署的话其实MongoDB和Elasticsearch都十分的方便, 不过es相对来说资源占用更多一点, 性能也比MongoDB要弱一点

集群化的部署, 我们一般都会选择分片+副本的部署方式, 这种方式下, es部署起来比MongoDB方便太多, MongoDB要部署一套完整的分片 + 副本模式还是比较麻烦的, 没有经验的人部署起来需要一定的学习成本

资源占用方面, MongoDB可以支持存储文件类型的数据, 作为数据库也有数据压缩能力, es则因为大量的索引存在需要占用大量的磁盘和内存空间

## 可用性和容错

MongoDB和ElasticSearch作为天生分布式的代表产品都支持数据分片和副本

两者都通过分片支持水平扩展, 同时都通过副本来支持高可用(HA)

分片就是一个数据集的数据分为多份, 同时分布在多个节点上存储和管理, 主流分片方式有两种: hash分片和range分片, 两种分片方式各有优势, 适合不同的场景

副本就是一份数据集同时有一个或者多个复制品(有些地方叫主从), 每份复制品都一模一样, 但是为了保证数据的一致性, 往往多个副本中只有一个作为Primary副本(通过选主算法从多个副本中选出Primary), 提供写服务, 其他副本只提供读, 或者只提供备份服务

ps:es和MongoDB都可以通过副本增强读能力, 这与kafka很不一样(kafka的副本只有备份功能)

### 分布式方案

MongoDB和Elasticsearch虽然都是分布式服务, 但是还是有一些不同方案的选择的

- 分片和副本单位的划分

MongoDB是以节点为单位划分角色, 一旦一个节点被指定为副本, 其上面的数据都是副本

Elasticsearch是以分片为单位划分角色, 一个节点上即可以拥有某分片的主分片和可以同时拥有另一个分片的副本分片, 同时es还支持自动的副本负载均衡, 如果一个新节点上面什么数据都没有, 系统会自动分配分片数据过来

- 架构模式

MongoDB的副本和分片是两种不同的模式, 虽然可以同时使用但是依然有各自的架构设计, 用户可以任意选择选型进行搭配, 每个节点的职责更加专一, 方便据此调整机器配置和进行优化

Elasticsearch中的分片 + 副本是一套统一的架构设计, 每个节点具有接近同等的地位, 配置使用起来更加简单, 但是如果要针对节点所负责的功能对机器进一步做定制就不如MongoDB灵活

## 文档型数据库的特点和问题

### 无schema

文档型数据存储既能享受无schema限制带来的灵活, 又能享受索引查询的快速和类SQL查询的便捷

使他们用起来不像传统的RDBMS那么麻烦, 又不像 Redis,Hbase这种数据库查询功能不够强大, 处在一个传统RDBMS和经典K-V存储之间的比较均衡的位置

我个人很喜欢这个特性, 没有schema的限制, 存储数据更方便也更灵活了, 但是有得有失, 很多固定schema的好处就无法享受到了, 比如: 对数据的高效压缩

### 鸡肋的Collection 和 Type

早期为了跟传统rdbms数据库保持概念一致 ，mongodb和elasticsearch都设计了跟传统数据库里面的`库->表->记录行`对应的概念，具体如下

| RDBMS | MongoDB | Elasticsearch |
| :---- | :------ | :------------ |
| 库    | 库      | 索引          |
| 表    | 集合    | 类型          |
| 记录  | 文档    | 文档          |

其实对于nosql数据库来讲, 集合/类型的意义其实不大, Nosql数据库几乎都是k-v类型的存储结构，完全可以通过key进行业务隔离和区分，真的没有必要为了跟传统数据库对应强行搞出来一个中间概念 _

Elasticsearch从`6.x`版本开始强制只允许一个索引使用一个type, 其实就是意识到这个这个设计的失误, 不想让你用这个type类型, 因为type和传统数据库里面的表概念其实是不一样的，这种概念类比给人造成了误解，到了es的7.x版本会默认取消type类型, 就说明这个type字段真的是鸡肋的不行

### 弱事务

MongoDB以前只是支持同一文档内的原子更新, 以此来实现伪事务功能, 不过Mongo4.0支持Replica Set事务, 大大加强了事务方面的能力

es在这方面倒没有什么进展，因为从应用场景上es对事务的需求不高，不过用户其实也可以使用同文档更新或者通过程序自己来实现事务机制

### 无join支持

文档型数据库大多数都不支持join(也有少量支持的), 但是我一般也用不上多表join的功能, 即便真的需要使用join也可以通过应用层或者通过耦合数据来实现（不过据说未来Mongo4.2版本会带来对join的支持）

不支持join带来的问题就是我们需要自己对数据进行连接, 但是这在擅长使用分布式计算的大数据领域不算什么问题, 相应的缺少join功能可能对善于使用SQL的数据分析师就不大友好

### Bully的选主算法的缺陷

elasticsearch和MongoDB选择的选主算法实现很简单, 但是代价就是有几率出现脑裂的情况, 当然, 具体情况跟配置也有关系(比如:你有三个es节点但是设置的最小主节点数为1, 将最小主节点数设置为2可以避免脑裂情况)

不过脑裂问题一方面发生概率较低，另一方面即使出现了脑裂的情况, 使用`重启大法`一般就能解决 _

总体来说, 这方面不如使用Paxos和Raft算法或者使用zk做协调器的其他分布式系统靠谱

## 适用场景

两者其实在很多使用场景上有重合之处, 是可以互相替代, 比如日志收集

但是某些方面两者又各有特色，比如： 如果打算使用一个文档型的业务数据库， 那最好还是选mongodb, 如果你有要求复杂查询又并发性能要求高的场景，类似搜索服务，那最好的选择是elasticsearch

除此之外：

MongoDB有多个存储引擎可以选择, 而且MongoDB不仅看重数据的分析, 对数据的管理同样看重, 总的来说MongoDB更倾向于数据的存储和管理, 可以作为数据源对外提供， 未来说不定还会有支持join和支持倒排索引的mongo引擎出现

Elasticsearch则有很多插件可以使用, 相对来讲Elasticsearch更倾向于数据的查询, 一般情况下elasticsearch仅作为数据检索服务和数据分析平台, 不直接作为源数据管理者

- MongoDB适合

1. 对服务可用性和一致性有高要求
2. 无schema的数据存储 + 需要索引数据
3. 高读写性能要求, 数据使用场景简单的海量数据场景
4. 有热点数据, 有数据分片需求的数据存储
5. 日志, html, 爬虫数据等半结构化或图片，视频等非结构化数据的存储
6. 有js使用经验的人员(MongoDB内置操作语言为js)

- Elasticsearch适合

1. 已经有其他系统负责数据管理
2. 对复杂场景下的查询需求，对查询性能有要求, 对写入及时性要求不高的场景
3. 监控信息/日志信息检索
4. 小团队但是有多语言服务，es拥有restful接口，用起来最方便