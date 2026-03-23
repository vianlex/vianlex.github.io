# ElasticSearch 语法查询学习笔记

## DSL 查询条件

```json
{
    // 指定匹配模式和具体的查询条件
    "query":{
        // 匹配模式：match_all(全索引查询), match(模糊查询), term(精确查询)、exists(判断字段是否存在)
        "match": {
            // 查询条件
        }
    },
    // 指定查询结果返回值
    "_source": ["field1", "field2"],
    // 指定查询结果的页大小，默认是 10
    "size": 100,
    // 指定查询结果的起始页，默认是 0
    "from": 1,
    // 指定排序字段，会让得分失效
    "sort" :{
        "orderNumber": "asc"
    }
}

```

## DSL 查询结果

```json
{
  "took": 9, // 查询花费时间
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    // 符合条件总条数
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.2876821,
    // 查询结果集
    "hits": [
      {
        "_index": "order_index",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "orderNumber": "123456789",
          "materialCode": "苹果手机 15Pro Max"
        }
      }
    ]
  }
```


## 测试数据

```bash
# 创建索引
PUT /order_index
{
    "settings": {
        "number_of_replicas": 1,
        "index.number_of_shards": 1,
        "index": {
            "analysis.analyzer.default.type":"ik_max_word"
        }
    }
}

# 批量添加文档，添加文档是会自动映射字段
PUT /order_index/_bulk
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123001" } }
{ "orderNumber": "ORD20241123001", "address": "北京市朝阳区建国门外大街1号", "quantity": 2, "price": 150.5, "amount": 301.0, "remark": "加急配送，请优先处理" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123002" } }
{ "orderNumber": "ORD20241123002", "address": "上海市浦东新区陆家嘴环路100号", "quantity": 1, "price": 899.0, "amount": 899.0, "remark": "商品需包装精美，作为礼物" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123003" } }
{ "orderNumber": "ORD20241123003", "address": "深圳市南山区深南大道10000号", "quantity": 5, "price": 45.8, "amount": 229.0, "remark": "部分商品有轻微瑕疵可接受" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123004" } }
{ "orderNumber": "ORD20241123004", "address": "广州市天河区天河路208号", "quantity": 3, "price": 120.0, "amount": 360.0, "remark": "发票抬头：广州科技有限公司" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123005" } }
{ "orderNumber": "ORD20241123005", "address": "杭州市西湖区文三路500号", "quantity": 10, "price": 28.5, "amount": 285.0, "remark": "分批配送，每批5件" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123006" } }
{ "orderNumber": "ORD20241123006", "address": "成都市锦江区红星路三段1号", "quantity": 1, "price": 2500.0, "amount": 2500.0, "remark": "需安装服务，请提前联系预约" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123007" } }
{ "orderNumber": "ORD20241123007", "address": "南京市鼓楼区中山路100号", "quantity": 4, "price": 75.0, "amount": 300.0, "remark": "收货人白天不在家，请晚上配送" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123008" } }
{ "orderNumber": "ORD20241123008", "address": "武汉市江汉区解放大道1000号", "quantity": 2, "price": 450.0, "amount": 900.0, "remark": "需提供产品检验报告" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123009" } }
{ "orderNumber": "ORD20241123009", "address": "西安市雁塔区小寨西路100号", "quantity": 8, "price": 36.5, "amount": 292.0, "remark": "部分颜色随机发货" }
{ "index" : { "_index" : "order_index", "_id" : "ORD20241123010" } }
{ "orderNumber": "ORD20241123010", "address": "重庆市渝中区解放碑步行街1号", "quantity": 1, "price": 1680.0, "amount": 1680.0, "remark": "商品需防震包装，易碎品" }

```


## match_all 全索引匹配

match_all 查询，表示匹配查询所有文档，默认返回 10 条数据

```bash
GET /order_index/_search
{
    "query":{
        "match_all":{}
    },
    # 表示查询结果返回哪些字段
    "_source": ["orderNumber","materialDesc"]
}
```

## Term 精确匹配查询

精确匹配查询是指的搜索内容不经过文本分析直接用于文本匹配查询，term 匹配查询主要用于字段的精准匹配场景。一般针对的是非 Text 类型字段查询。

**注意** Term 精准匹配，表示的是精准匹配倒排索引，而不是直接匹配文档字段的。

1、单值匹配

```json
GET /order_index/_search
{
  "query": {
    // 表示匹配条件不经过分词，直接匹配倒排索引表
    "term": {
      // [加急配送，请优先处理] 字符串经过 ik_max_word 分词后的倒排索引为：[加急、配送、请、优先、处理]
      "remark": "配送" // "配送"不经过分词，直接精准匹配倒排索引表
    }
  }
}

GET /order_index/_search
{
  "query": {
    "term": {
      /** 
       * 如 text 类型的字段设置子字段 keyword，那么可以使用该 keyword 子字段来精准匹配查询
       * 注意匹配的还是倒排索引表，remark.keyword 和 remark 是两个不同的索引字段
       * remark.keywrod 索引是 keyword 类型，在倒排索引表中，只有一条倒排索引记录
       * remark 索引是 text 类型，在倒排索引表中，都多条分词的倒排索引记录，但是没有全词的倒排索引。
       */
      "remark.keyword": "配送" 
    }
  }
}
```

2、多值匹配

```json
GET /order_index/_search
{
  "query": {
    // terms 表示使用 in 的方式，不分词精准匹配倒排索引表
    "terms": {
      // 精准匹配倒排索引表中的“加急”和“配送”索引
      "remark": ["加急", "配送"] 
    }
  }
}
```

3、将 Query 查询改成 filter 过滤方式提高性能

```json
GET /order_index/_search
{
    "query": {
        // 表示常数分数，精准查询是不需要计算分数的，可以设置常量，来提示查询性能
        "constant_score": {
          "filter": {
            "term": {
              "remark": "加急"
            }
          },
          // 可选，返回默认分数
          "boost": 1.2
        }
    }
}
```

## range 精准范围查询

range 范围查询一般用于 text 类型字段查询。

```json
GET /order_index/_search
{
    "query": {
        "range":{
            "quantity": {
              "gte": 5,
              "lte": 10
            }
        }
    }
}
```

ES 中常见时间表达式

- now 表示当前时间
- now-1d 表示从当前时间减一天的时间
- now-1w 表示从当前时间减去一周的时间
- now-1m 表示从当前时间减去一个月的时间
- now-1y 表示从当前时间减去一年的时间
- now+1h 表示从当前时间加上一个小时的时间


```json
GET /order_index/_search
{
    "query": {
        "range": {
            "orderDate": { "gte": "now-1m"}
        }
    }
}
```


## exists 是否存在查询

查询索引库中存在某个字段的文档

```json
GET /order_index/_search
{
    "query": {
        "exists": {
            // 表示只要文档中存在 remark 字段，结果就返回该文档
            "field": "remark"
        }
    }
}
```


## ids 精准查询

根据文档 ids 查询文档

```json
GET /order_index/_search
{
    "query": {
        // 根据文档的 _id 查询
        "ids": {
            "values": ["ORD20241123001","ORD20241123002"]
        }
    }
}
```


## prefix 精准前缀匹配

prefx 精准前缀匹配，表示匹配条件值，不经过分词，直接匹配倒排索引表。

```json
GET /order_index/_search
{
    "query": {
        "prefix": {
          // 精准前缀匹配查询，不经过分词，直接前缀匹配倒排索引
          "orderNumber": {
            // ORD20241123001 通过 ik_max_work 分词结果为[ord20241123001，ord，20241123001] 直接通过 ORD20241123001 精准前缀匹配，匹配不成功
            // "value": "ORD20241123001"
            // 通过 ord 或者 20241123001 能查询到结果
            "value": "ord"
          }
        }
    }
}


GET /order_index/_search
{
    "query": {
        "prefix": {
          // orderNumber.keyword 索引，在倒排索引中有记录，能匹配到值
          "orderNumber.keyword": {
            "value": "ORD20241123001"
          }
        }
    }
}
```

## wildcard 精准通配符匹配

wildcard 精准通配符匹配，表示匹配条件值，不经过分词，直接就匹配倒排索引表。
wildcard 支持 * 和 ? 通配符号，* 表示模糊匹配一个或者多个字符，? 表示模糊匹配一个字符


```json
GET /order_index/_search
{
  "query": {
    "wildcard": {
      // 不经过分词，直接模糊匹配 remark 在倒排索引表中索引记录
      "remark": "?送" 
    }
  }
}
```

## fuzzy 精准模糊匹配

fuzzy 精准模糊匹配，表示匹配条件值，不经过分词，直接模糊性的匹配倒排索引表。

```json
GET /order_index/_search
{
    "query": {
        // 模糊性匹配
        "fuzzy": {
          "remark": {
            "value": "加送",
            // 表示模糊匹配的距离,fuzziness=1时, "加送" 的模糊匹配规则为: "?送" 或 "加?",当 fuzziness=2 时，则匹配规则为："??"
            // 其他例子：fuzziness=1 时，"你们好" 的模糊匹配规则为："?们好", "你?好", "你们?"，"们好"，"你好",
            "fuzziness": 1 // fuzziness 可选值 auto(默认是 0)，1，2
          }
        }
    }
}


GET /order_index/_search
{
    "query": {
        // 模糊性匹配
        "fuzzy": {
           // remark="加急配送，请优先处理"，经过 ik_max_word 分词后的倒排索引为：[加急、配送、请、优先、处理]
          "remark": {
            "value": "加急配送",
            "fuzziness": 1 // 匹配不成功，因为倒排索引表里没有如下索引，[?急配送, 加?配送, 加急?送, 加急配?, 急配送, 加配送, 加急配]
            // 将 fuzziness 值改为 2 时，可以匹配到结果，因为倒排索引表里有【配送、加急】索引
          }
        }
    }
}

```


## term_set 精准多条件值匹配

```json
GET /order_index/_search
{
    "query": {
        "terms_set":{
            // [加急配送，请优先处理]字符串经过 ik_max_word 分词后的倒排索引为：[加急、配送、请、优先、处理]
            "remark": {
                "terms":["加急","配送"],
                // 表示必须精准匹配 terms 中两个 remark 的倒排索引
                "minimum_should_match": 2
            } 
        }
    }
}

// 通过脚本的方式计算需要匹配的个数
GET /order_index/_search
{
    "query": {
        "terms_set":{
            "remark": {
                "terms":["加急","配送"],
                // 注意 quantiy 是 order_index 索引文档的 quantity 字段
                "minimum_should_match_script": {
                    // 动态计算匹配数量
                    "source": "doc['quantity'].value * 0.5"
                }
            } 
        }
    }
}

```


## match 分词匹配查询

match 查询是一种全文匹配查询，它使用分词器将查询字符串拆分成一个一个单独的分词，然后在倒排索引检索这些分词。

match 查询流程分为三步
1. 分词：使用分词器处理查询字符串，将字符串拆分成一个一个单独的分词，如单词，短语，特定字符。
2. 匹配计算：
3. 结果返回：


1、match 分词查询，不带参数查询的例子如下：

```json
GET /order_index/_search
{
    "query": {
        // match 匹配，先将查询字符串先分词，再根据分词匹配倒排索引
        "match": {
          // 通过 ik_max_word 分词器将 "加急配送"，分词为: ["加急"、"配送"]后，在使用["加急"、"配送"]去通过 or 关系精准匹配倒排索引表
          "remark": "加急配送"
        }
    }
}
```

2、match 分词查询，带参数查询的例子如下：

```json
GET /order_index/_search
{
    "query": {
        "match": {
           "address": {
                // query 经过 ik_max_word 分词结果为: ["加急", "配送"]
                "query": "加急 配送",
                // 默认是 or，表示多个分词匹配倒排索引的逻辑关系
                "operator": "and" ,
                // 表示要匹配多少个分词，如 75%、2，注意：不能和 operator=and 同时使用会有冲突问题，可以 operator=or 使用
                // "minimum_should_match": 2, 
                // 表示模糊性匹配查询 AUTO 为 0，表示不模糊匹配
                "fuzziness": "AUTO",  
                "analyzer": "ik_max_word"  // 使用IK分词器（中文）
           }
        }
    }
}
```


## mutil_match 一值匹配多字段分词查询

mutil_match 查询会对多个字段的倒排索引进行匹配查询。

```json
GET /order_index/_search
{
  "query": {
    "multi_match": {
      "query": "北京分批",
      // 多字段匹配查询, ^ 指定查询字段的权重来计算分数，分数越高越排前面
      "fields": ["remark^2", "address"]
    }
  }
}
```



## match_phrase 短语匹配查询

match_phrase 短语匹配查询，表示查询的字符串分词后按 and 的方式查询，并且要顺序控制。

1、match_phrase 短语匹配，不带参数查询例子如下：

```json
GET /orders/_search
{
  "query": {
    /**
    * 如 remark="加急配送，请优先处理"，经过 ik_max_word 分词后，结果如下：
    * [{token: "加急", postion: 0 }, {token:"配送", postion: 1}, {token: "请", position: 2}, {token: "优先", posistion: "3"}, {token: "处理", postion: 4}]
    */
    "match_phrase": {
      // 查询条件 "加急配送" 通过 ik_max_word 分词后为 [加急，配送]
      // remark 倒排索引中，必须存在 [加急，配送] 分词才能查询成功，并且在倒排索引中是 position 连续的分词，如果非连续的可以通过 slop 指定间隔
      "remark": "加急配送" 
      //"remark":"加急请" // ik_max_word 分词后为[加急，请]，匹配不到结果，
    }
  }
}
```

1、match_phrase 短语匹配，带参数查询例子如下：

```json
GET /orders/_search
{
  "query": {
    /**
    * 如 remark="加急配送，请优先处理"，经过 ik_max_word 分词后，结果如下：
    * [{token: "加急", postion: 0 }, {token:"配送", postion: 1}, {token: "请", position: 2}, {token: "优先", posistion: "3"}, {token: "处理", postion: 4}]
    */
    "match_phrase": {
      // 查询条件 "加急配送" 通过 ik_max_word 分词后为 [加急，配送]
      // ik_max_word 分词后为[加急，请]，查询条件的分词，在 remark 倒排索引中非连续的，需要通过 slop 指定间隔
      "remark":"加急请", 
      "slop": 1 // 不指定间隔，查询不到数据
    }
  }
}
```


## query_string 分词与或非表达式查询


1、单个字段查询的例子

```json
GET /order_index/_search
{
    "query": {
        "query_string": {
          // 指定查询倒排索引的字段，如不指定，则查询文档在倒排索引中的维护所有索引字段
          "default_field": "remark",
          // AND 表示要同时满足两个查询字符串“加急请”和“配送”分词后的查询，
          // 注意单独查询字符串的匹配是OR的关系，比如“加急请”分词后为["加急","请"]，只要满足一个即可
          "query": "加急请 AND 配送"
        }
    }
}
```

2、多个字段查询的例子

```json
GET /order_index/_search
{
    "query": {
        "query_string": {
          "fields": ["remark", "address"],
          "query": "加急请 AND 配送"
        }
    }
}
```

## bool 组合查询

在 bool 组合查询条件中，可以分为两种不同的上下文
1. 搜索上下文（query context）：使用搜索上下文时，ES 会计算每个文档和查询条件的相关度得分，计算公式比较复杂，会消耗性能。
2. 过滤上下文（filter context）：使用过滤上下文时，ES 只会判断搜索条件和文档数据是否匹配，不会计算相关度得分，并且会缓存结果，提升查询速度。

bool 组合查询支持4种组合类型查询如下：

- must：可以包含多个查询条件，并且都满足才能成功检索文档，每次查询都会计算文档和搜索条件的相关度得分（计算得分会消耗性能）属于搜索上下文。
- should：可以包含多个查询条件，不存在 must 和 filter 条件时，至少要满足其中一个查询条件，才能检索到文档，注意该查询也会计算文档和搜索条件的相关度得分，属于搜索上下文。
- filter：可以包含多个过滤条件，并且每个过滤条件都满足后才能检索到文档， 每个过滤条件不计算相关度得分，结果在一定条件下会被缓存，属于过滤上下文。
- must_not：可以包含多个过滤条件，每个过滤条件都不满足得文档带能被检索到，不计算相关度得分，结果在一定条件下会被缓存，属于过滤上下文。
  
**注意**：对于精确匹配建议通过 filter 来实现组合查询，因为精准查询可以使用常量分数，而不需要根据相关度计算分数。



1、must 组合查询例子如下：

```json
GET /order_index/_search
{
    "query": {
        "bool": {
            "must": [
              {"match": {"remark": "加急配送"}},
              {"match": {"address": "杭州"}}
            ]
        }
    }
}


GET /order_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "remark": {
              "query": "配送请",
              // 表示匹配所有分词， 等价 operator = and
              "minimum_should_match": "100%"
            }
          }
        }
      ]
    }
  }
}
```

2、完整组合查询例子：

```json
GET /order_index/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "北京" } },
        { "match": { "remark": "配送" } }
      ],
      "filter": [
        { "range": { "quantity": { "gte": 2 } } }
      ],
      "should": [
        { "match": { "remark": "加急" }}
      ],
      // 指定 should 必须满足多少个条件，默认就是1个
      "minimum_should_match": 1,
      "must_not": [
        { "match": { "remark": "瑕疵" } }
      ]
    }
  }
}
```

##  highlight 高亮

highlight 高亮，通过 html 标签，将查询结果文档中与查询条件分词匹配字符串的高亮显示。

highlight 相关属性：
- pre_tags 前缀标签，如 <span>
- post_tags 后缀标签，如 </span>
- tags_schema 设置为 styled 可以使用内置高亮样式
- require_field_match 多字段高亮需要设置为 false

1、高亮所有查询条件的分词例子

```json
GET /order_index/_search
{
  "query": {
    "multi_match": {
      "query": "配送",
      "fields": ["remark", "address"]
    }
  },
  "highlight": {
    "fields": {
      // 表示查询字段都高亮
      "*": {}
    }
  }
}
```

2、高亮指定字段

```json
GET /order_index/_search
{
  "query": {
    "multi_match": {
      "query": "配送",
      "fields": ["remark", "address"]
    }
  },
  "highlight": {
    "fields": {
      "address": {},
      "remark": {
        "pre_tags": ["<span class=\"highlight\">"],
        "post_tags": ["</span>"],
        "encoder": "html",  // 编码类型：html(default) / 不编码
      }
    }
  }
}
```

3、高亮全局参数设置

```json
GET /order_index/_search
{
  "query":{
    "term": { "remark": "配送" }
  },
  "highlight": {
    "pre_tags": ["<em class=\"highlight\">"],
    "post_tags": ["</em>"],
    "encoder": "html",  // 编码类型：html(default) / 不编码
    "tags_schema": "styled",  // 标签方案
    "max_analyzed_offset": 1000000,  // 最大分析偏移量
    "fields": {
      "remark": {
        "force_source": true  // 强制从_source获取
      }
    }
  }
}
```

## 地理位置查询

地理位置查询的字段必须是 geo_point 或者 geo_shape 类型。


1、测试数据 

```json
// 创建索引
PUT /locations
{
  "mappings": {
    "properties": {
      "name": { "type": "text" },
      "location": { "type": "geo_point" }
    }
  }
}

// 创建测试数据
POST /locations/_doc/1
{
  "name": "天安门广场",
  "location": "39.9087,116.3975"  // 字符串格式：lat,lon
}

POST /locations/_doc/2
{
  "name": "故宫",
  "location": {  // 对象格式
    "lat": 39.9163,
    "lon": 116.3972
  }
}

POST /locations/_doc/3
{
  "name": "王府井",
  "location": [116.4170, 39.9150]  // 数组格式：[lon, lat]
}

```

2、geo_distance 距离查询例子

```json
GET /locations/_search
{
  "query": {
    "geo_distance": {
      // 支持单位：m(米)、km(千米)、mi(英里)、yd(码)、ft(英尺)、in(英寸)
      "distance": "5km",  // 距离：5公里内
      "location": {
        "lat": 39.9042,
        "lon": 116.4074
      }
    }
  }
}

```

3、geo_bounding_box 矩形范围查询例子

```json
GET /locations/_search
{
  "query": {
    "geo_bounding_box": {
      "location": {
        "top_left": {  // 左上角坐标
          "lat": 40.0,
          "lon": 116.0
        },
        "bottom_right": {  // 右下角坐标
          "lat": 39.8,
          "lon": 116.5
        }
      }
    }
  }
}
```

4、geo_polygon 多边形范围查询例子

```json
GET /locations/_search
{
  "query": {
    "geo_polygon": {
      "location": {
        "points": [
          { "lat": 40.0, "lon": 116.0 },
          { "lat": 40.0, "lon": 116.5 },
          { "lat": 39.5, "lon": 116.5 },
          { "lat": 39.5, "lon": 116.0 }
        ]
      }
    }
  }
}
```

5、按距离排序

```json
GET /locations/_search
{
  "sort": [
    {
      "_geo_distance": {
        "location": [116.4074, 39.9042],  // 中心点
        "order": "asc",  // 升序：从近到远
        "unit": "km",    // 单位
        "distance_type": "plane",  // 计算方式：plane/arc
        "mode": "min",   // 模式：min/max/median/avg
        "ignore_unmapped": true  // 忽略未映射字段
      }
    }
  ]
}
```


## 聚合查询

### max 等聚合查询

```json
GET /order_index/_search
{
    "query": {
        "term": {"remark": "配送"}
    },
    "_source": ["quantity"],
    "aggs": {
      "maxQuantity": {"max": { "field": "quantity" }},
      "minQuantity": {"min": { "field": "quantity" }},
      "sumQuantity": {"sum": {"field": "quantity"}}
    }
}
```

### stats 聚合查询

stats 聚合查询会直接返回 max、min、count、sum 等聚合结果

```json
GET /order_index/_search
{
    "query": {
        "term": {"remark": "配送"}
    },
    "_source": ["quantity"],
    "aggs": {
      "statsQuantity": {"stats": { "field": "quantity" }}
    }
}

// 输出结果
"aggregations": {
    "statsQuantity": {
      "count": 3,
      "min": 2,
      "max": 10,
      "avg": 5.333333333333333,
      "sum": 16
    }
}
```

### cardinate 去重查询

```json
GET /order_index/_search
{
    "query": {
        "term": {"remark": "配送"}
    },
    "aggs": {
      "cardinate": {
        // 注意不能 text 类型去重，如果 text 类型想去重需要字段映射属性需要设置 fielddata=true
        "cardinality": {"field": "orderNumber.keyword"}
      }
    }
}
```


## 桶(分组)聚合查询


1、按 terms(分组)统计数量的例子

```json
GET /order_index/_search
{
    "query": {
        "match": {"remark": "配送"}
    },
    "aggs": {
       // 返回结果字段名称
      "remarkGroup": {
        "terms": {
            // 按 remark.keyword 分组统计数量
            "field": "remark.keyword",
            // 默认返回全部数据
            "size": 10,
            // 对分组统计结果进行排序
            "order":{"_count": "desc"}
        }
      }
    }
}
```

2、按 range 范围统计数量的例子

```json
GET /order_index/_search
{
    "query": {
        "match": {"remark": "配送"}
    },
    "aggs": {
      "quantityRangeCount": {
        "range": {
            "field": "quantity",
            "ranges": [
              {"key":"大于等0小于2", "to": 2},
              {"key":"2<=quantity<5", "from": 2, "to": 5},
              {"key": "统计大于10的", "to": 10}
            ]
        }
      }
    }
}
```

3、多字段嵌套分组

```json
GET /order_index/_search
{
    "query": {
        "match": {"remark": "配送"}
    },
    "aggs": {
      "group1": {
        "terms": {
            "field": "remark.keyword"
        },
        "aggs": {
          "group2": {
            // 统计 min、max、count、sum
            "stats": {
                "field": "quantity"
            }
          }
        }
      }
    }
}
```

## 管道聚合

通过管道的方式，将上一个聚合操作的结果，传给下一个聚合操作。


```json
GET /order_index/_search
{
  "aggs": {
    "totalQuantity": {
      "terms": {"field": "orderNumber.keyword"},
      "aggs": {
        "totalAmount": {
          "sum": { "field": "amount" }
        }
      }
    },
    // 获取 totalQuantity 和 totalAmount 分组后最小桶
    "min_amount_bucket": {
        "min_bucket": {
        "buckets_path": "totalQuantity>totalAmount"
        }
    }
  }
}
```