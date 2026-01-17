# Elasticsearch 基础概念笔记

## 安装 Elasticsearch 

```yml
services:
  elasticsearch:
    image: elasticsearch:8.19.10
    container_name: elasticsearch-single
    environment:
      - node.name=elasticsearch-single
      - cluster.name=es-docker-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - xpack.security.enabled=false  # 开发环境可关闭安全
      - network.host=0.0.0.0
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./data:/usr/share/elasticsearch/data
      - ./plugins:/usr/share/elasticsearch/plugins
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - es-network
    restart: unless-stopped

  kibana:
    image: kibana:8.19.10
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=["http://elasticsearch:9200"] # 如果开启了安全认证，这里需要配置用户名密码
    networks:
      - es-network
    depends_on:
      - elasticsearch

networks:
  es-network:
    driver: bridge
```

## 常用插件安装

### `ik` 中文分词器插件

下载地址：`https://github.com/medcl/elasticsearch-analysis-ik` 或者 `https://release.infinilabs.com/analysis-ik/stable/`

离线安装方式：将插件解压到 `elasticsearch` 安装目录中的 `plugins` 目录下（注意将插件的文件夹名称改成 ik），然后重启 `elasticsearch` 服务即可。  

`ik` 分词器支持三种分词规则： standard、 ik_smart、 ik_max_word（一般采用这种方式）

测试例子：

```json
POST http://127.0.0.1:9200/_analyze
{ "analyzer": "ik_smart", "text": "中华人民共和国" }
```


## Elasticsearch 基本概念

### 全文检索（Full-Text Search）

全文检索（Full-Text Search）是一种从大量文本数据中快速检索出包含指定词汇或短语的信息的技术。

### 倒排索引

通过文本分词和索引（index）文档 ID 预先构建倒排索引关联表，当查询数据时，先根据倒排索引表查出索引文档 ID，然后在根据索引文档 ID 去查询索引文档。

倒排索引表的映射结构如下：

```txt
关键词 | 文档ID 
中国   | 100, 101
你好   | 100 
Java   | 102, 105
```

### 索引、映射、文档

- 索引（Index）: 相当于 MySql 中的表
- 映射（Mapping）：相当于 MySql 表中的字段
- 文档（Document）：相当于 MySqll 表中的记录


### 索引

**注意**索引名称必须是小写字母，可以包含数字和下划线

#### 索引的组成部分

- alias：索引别名
- setings：索引设置，设置分片和副本的数量等。
- mapping：映射，定义了索引|中包含哪些字段，以及字段的类型、长度、分词器等。


#### 索引设置（settings）

1. 分片数量（`number_of_shards`），表示将索引拆分成多少个主分片（只负责写和修改操作），作用是实现索引的负载均衡和数据分布。
2. 副本数量（`number_of_replicas`），设置每个分片有多少个副本数量（副本分片只负责读操作），作用是提高索引的可用性和容错能力。


#### 索引映射

字段属性（`properties`）定义索引中文档的字段及其类型。常用字段类型包括：`text，keyword，integer, float, date `等。简单例子如下：

**注意**：只有 text 类型是支持分词和指定分词器的，如果字符串不想要分词可以设置为 keyword 类型。


```json
"properties" :{
    "materialDesc": {
        "type": "text", // 字段类型
        "analyzer": "ik_max_word", // 指定倒排索引的分词器
        "search_analyzer": "ik_max_word", // 指定查询条件的分词器
        "index": false, // 表示不需要该字段作为查询条件，即不会生成倒排索引
    },
    // 倒排索引关联分词和不分词的 materialName 
    "materialName": {
        "type": "text",
        "analyzer": "ik_max_word", // 指定分词器
        "fields": {
            // 添加 keyword 子属性，表示将 materialName 不分词的值存到倒排索引中，查询时可以使用 materialName.keyword 作为查询条件
            // 注意 materialName 和 materialName.keyword 是两个不同索引字段
            "keyword": {
                "type": "keyword"
            },
            // 通过 materialName.keyword 和 materialName.hello 来查询效果是一样的，因为它们都是 keyword 类型，在索引表中都只有一条倒排记录
            "hello": {
              "type": "keyword"
            }
        }
    },
    "orderNumber": {
        "type": "keyword", // keyword 类型，表示不需要对改字段进行分词，直接存完整的值到倒排索引表表中。
    },
    "orderNumber": {
      "type": "date",
      // 支持多种格式
      "format": "strict_date_optional_time||epoch_millis||yyyy-MM-dd HH:mm:ss||dd/MM/yyyy"
    },
}
```

#### 索引基本操作

1、创建索引（添加文档时，如果索引不存在，会自动创建）

```json
// 简单创建索引
PUT http://127.0.0.1:9200/order_index

// 创建索引时指定配置和字段信息
PUT http://127.0.0.1:9200/order_index
{
    "settings": {
        "number_of_replicas": 1,
        "index.number_of_shards": 1
    },
    "mappings": {
        "properties":{
            "orderNumber":{
            "type":"text"
        },
        "materialCode":{
            "type":"text"
        }
        }
    }
}

```

2、查询索引

```json
GET http://127.0.0.1:9200/order_index


// 查询索引的数据
GET /order_index/_search
{
    "query": {
        "match":{
            "orderNumber": "12331121"
        }
    }
}

```

3、删除索引

```
DELETE http://127.0.0.1:9200/order_index
```

4、修改索引

```bash
PUT /order_index/_settings
{ 
    "index":{
        "number_of_replicas":2 
    }
}
```

5、重建索引

```bash
# 表示将 order_index 索引的数据，导到 order2_index 索引中。
PUT /_reindex
{
    "source": {"index" : "order_index"}
    "dest": {"index": "order2_index"}
}
# 删除 order_index 索引
DELETE /order_index
# 将 order2_index 索引的别名设置为 order_index
PUT /order2_index/_alias/order_index
# 通过 order_index 别名查询 order2_index 索引
GET /order2_index 
GET /order_index 
```


#### 索引别名

##### 索引别名的使用场景

1. 如 PB 级别的增量订单数据，当我们按日期分片创建订单索引时，可以通过别名来关联订单分片索引，查询时我们按索引别名查询即可。
2. 索引的设计不合理，比如某个字段的分词定义不准确，那么如果有索引别名，我们就能在不停止服务和修改业务代码的情况，更换索引。
3. 创建别名时，指定别名的过滤器，可以实现索引的多租户隔离。
4. 通过不同操作创建不同别名，如实现读写分离。
5. 通过索引模板（自动创建按月索引），实现时间序列索引滚动。
6. 创建别名时，指定别名的过滤器按用户组过滤，按用户路由查询，实现 A/B 测试。

##### 索引别名的命名规范

```txt
建议命名模式:
- 业务别名: orders, products, users
- 读写分离: orders_write, orders_read
- 时间分层: orders_current, orders_2023, orders_archive
- 环境隔离: orders_dev, orders_prod
- 租户隔离: tenant_a_orders, tenant_b_orders
- 功能别名: orders_paid, orders_shipped, orders_cancelled

避免:
- 使用特殊字符
- 与索引同名
- 过于简短的名称
```

##### 索引别名的基本操作

1、一个别名可以指向多个索引
```json
// 别名 orders 同时指向两个索引
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "orders_202401",
        "alias": "orders"
      }
    },
    {
      "add": {
        "index": "orders_202312",
        "alias": "orders"
      }
    }
  ]
}

// 查询时自动搜索所有关联索引
GET /orders/_search
{
  "query": { "match_all": {} }
}
```

2、一个索引创建多个别名

```json
// 一个索引拥有多个别名
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "orders_current",
        "alias": "orders_hot"
      }
    },
    {
      "add": {
        "index": "orders_current", 
        "alias": "orders_latest"
      }
    },
    {
      "add": {
        "index": "orders_current",
        "alias": "orders_read"
      }
    }
  ]
}
```

3、索引的别名可以自带过滤条件

```json
// 创建带过滤条件的别名
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "orders_202401",
        "alias": "orders_paid",
        "filter": {
          "term": { "status": "paid" }
        }
      }
    }
  ]
}

// 查询 orders_paid 只会返回已支付订单
GET /orders_paid/_search
{
  "query": { "match_all": {} }
}
```

4、别名可以指定路由

```json
// 创建带路由的别名
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "orders_by_user",
        "alias": "user_orders",
        "routing": "user_123"  // 固定路由值
      }
    }
  ]
}

// 或者指定路由值字段
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "orders_by_user",
        "alias": "orders_by_user_alias",
        "index_routing": "user_id",  // 索引时使用的路由字段
        "search_routing": "user_id"  // 搜索时使用的路由字段
      }
    }
  ]
}
```

### 文档

#### 文档数据结构

```
{
  "_index": "users",   // 文档所属的索引
  "_id": "1",          // 文档的唯一 ID
  "_version": 1,       // 文档的版本号，文档更新或者删除后，版本号都会自动增加 1
  "_seq_no" : 10,      // 文档的全局序列号，更新或者删除操作，都是自增 1，多线程操作文档时，能保证并发性
  "_primary_term": 1,   //
  "_source": {         // 实际 Json 数据
    "name": "张三",
    "age": 25,
    "email": "zhangsan@example.com"
  }
}
```

#### 文档基本操作

1、写入文档（index 索引不存在时，会自动创建）

```bash
# 写入文档时，指定文档 ID
PUT /users/_doc/1
{
  "name": "张三",
  "age": 25,
  "email": "zhangsan@example.com",
  "created_at": "2024-01-15T10:00:00Z"
}

# 自动创建文档 ID
POST /users/_doc
{
  "name": "李四",
  "age": 30,
  "email": "lisi@example.com"
}

# 带条件创建文档
PUT /users/_doc/1?op_type=create
{
  "name": "王五",
  "age": 28
}

```

2、批量添加文档

```bash
# 批量添加的固定格式为: 索引信息 + 数据信息
POST /user_index/_bulk
{"index":{"_id":1}}
{"name":"小明","birthday":"2026-01-18", "interest":["跑步","篮球"]}
{"index":{"_id":2}}
{"name": "小红", "birthday":"2026-01-16", "interest": ["跳舞", "画画"]}
```


3、文档查询（支持两种查询方式，一种是 URL Query，另一种是 DSL Query）

```bash
# 基于 URL Query 查询, from 和 size 表示分页查询
GET /order_index/_doc/_search?q=orderNumber:123121312&from=0&size=10

# 基于 DSL Query 查询
GET /order_index/_search 
{
    "query":{
        "match" :{
            "materialDesc": "苹果手机"
        }
    }
}
```


#### 文档的关联关系

1、嵌套对象（Nested Object）

优点是嵌套对象将文档存储在一起，能提高读取性能，缺点是更新子文档，需要更新整个文档，同时查询的效率也相对较慢，适用场景是对少量子文档偶尔更新和查询频繁的情况。如商品和商品属性的关联关系就比较适合适用嵌套对象。


2、Join 父子文档类型

优点是父子文档可以独立跟新，互相不影响，缺点需要维护 Join 关系比较消耗内存，同时查询读取效率比 Nested 还差，适用场景是子文档更新比较频繁的情况下。如博客文章与评论之间的关联关系，或者商品和商品评论之间的关联关系。

3、宽表冗余存储

宽表适用于一对多或者多对多的关联关系。以空间换取时间，宽表的优点是速度快。缺点则是索引更新或删除数据时，应用程序不得不处理宽表的余数据，并且冗余字段会造成存储空间的浪费。适用一些业务报表查询的场景，将几个业务表冗余在一起。


4、业务端关联

多个索引通过分开多次请求来完成。业务端关联适用于数据量少的多表关联业务场景。数据量少时，用户体验好；而数据量多时，两次查询耗时肯定会比较长，反而影响用户体验。


#### 文档设计实践

1. 一个文档避免适用大量的字段，默认最大字段数是1000， 可以设置 index.mapping.total_fields.limit 限定最大字段数。
    - 过多的字段不容易维护
    - 删除和修改文档需要重建 Index 比较耗时。
    - 尽量不要将 Dynamic 动态新增字段的值设置为 strict，这样新增字段不会被索引，并且文档写入失败。

2. 避免使用正则、通配符、前缀查询，它们都属于 term 查询，查询性能很差，解决方案可以将字段拆分多个字段存储。

    ```json
      // 拆成多个字段
      { "version": { "full_name": "8.19.10", "marjar": 8, "minor":  19, "hot_fix": 10 } }
    ```
3. 字段的值避免使用 null 空值，最好设置一个默认值。

4. 为索引的 mapping 添加 meta 信息，方便进行版本管理，同时可以考虑将 mapping 文件使用 git 管理，方便我们进行回退。 


