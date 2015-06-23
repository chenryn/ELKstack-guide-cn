# 搜索请求

上节介绍的，都是针对单条数据的操作。在 ES 环境中，更多的是搜索和聚合请求。在之前章节中，我们也介绍过数据获取和数据搜索的一点区别：刚写入的数据，可以通过 translog 立刻获取；但是却要等到 refresh 成为一个 segment 后，才能被搜索到。本节就介绍一下 ES 的搜索语法。

## 全文搜索

ES 对搜索请求，有简易语法和完整语法两种方式。简易语法作为以后在 Kibana 上最常用的方式，一定是需要学会的。而在命令行里，我们可以通过最简单的方式来做到。还是上节输入的数据：

```
# curl -XGET http://127.0.0.1:9200/logstash-2015.06.21/testlog/_search?q=first
```

可以看到返回结果：

```
{"took":240,"timed_out":false,"_shards":{"total":27,"successful":27,"failed":0},"hits":{"total":1,"max_score":0.11506981,"hits":[{"_index":"logstash-2015.06.21","_type":"testlog","_id":"AU4ew3h2nBE6n0qcyVJK","_score":0.11506981,"_source":{
    "date" : "1434966686000",
    "user" : "chenlin7",
    "mesg" : "first message into Elasticsearch"
}}]}}
```

还可以用下面语句搜索，结果是一样的。

```
# curl -XGET http://127.0.0.1:9200/logstash-2015.06.21/testlog/_search?q=user:"chenlin7"
```

### querystring 语法

上例中，`?q=`后面写的，就是 querystring 语法。鉴于这部分内容会在 Kibana 上经常使用，这里详细解析一下语法：

* 全文检索：直接写搜索的单词，如上例中的 `first`；
* 单字段的全文检索：在搜索单词之前加上字段名和冒号，比如如果知道单词 first 肯定出现在 mesg 字段，可以写作 `mesg:first`；
* 单字段的精确检索：在搜索单词前后加双引号，比如 `user:"chenlin7"`；
* 多个检索条件的组合：可以使用 `NOT`, `AND` 和 `OR` 来组合检索，注意必须是大写。比如 `user:("chenlin7" OR "chenlin") AND NOT mesg:first`；
* 字段是否存在：`_exists_:user` 表示要求 user 字段存在，`_missing_:user` 表示要求 user 字段不存在；
* 通配符：用 `?` 表示单字母，`*` 表示任意个字母。比如 `fir?t mess*`；
* 正则：需要比通配符更复杂一点的表达式，可以使用正则。比如 `mesg:/mes{2}ages?/`。注意 ES 中正则性能很差，而且支持的功能也不是特别强大，尽量不要使用。ES 支持的正则语法见：<https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-regexp-query.html#regexp-syntax>；
* 近似搜索：用 `~` 表示搜索单词可能有一两个字母写的不对，请 ES 按照相似度返回结果。比如 `frist~`；
* 范围搜索：对数值和时间，ES 都可以使用范围搜索，比如：`rtt:>300`，`date:["now-6h" TO "now"}` 等。其中，`[]` 表示端点数值包含在范围内，`{}` 表示端点数值不包含在范围内；

### 完整语法

ES 支持各种类型的检索请求，除了可以用 querystring 语法表达的以外，还有很多其他类型，具体列表和示例可参见：<https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-queries.html>。

作为最简单和常用的示例，这里展示一下 term query 的写法，相当于 querystring 语法中的 `user:"chenlin7"`：

```
# curl -XGET http://127.0.0.1:9200/_search -d '
{
  "query": {
    "term": {
      "user": "chenlin7" 
    }
  }
}'
```

## 聚合请求

在检索范围确定之后，ES 还支持对结果集做聚合查询，返回更直接的聚合统计结果。在 ES 1.0 版本之前，这个接口叫 Facet，1.0 版本之后，这个接口改为 Aggregation。

Kibana 分别在 v3 中使用 Facet，v4 中使用 Aggregation。不过总的来说，Aggregation 是 Facet 接口的强化升级版本，我们直接了解 Aggregation 即可。本书后续章节也会介绍如何在 Kibana 的 v3 版本中使用 aggregation 接口做二次开发。

Aggregation 分为 bucket 和 metric 两种，分别用作词元划分和数值计算。而其中的 bucket aggregation，还支持在自身结果集的基础上，叠加新的 aggregation。这就是 aggregation 比 facet 最领先的地方。比如实现一个时序百分比统计，在 facet 接口就无法直接完成，而在 aggregation 接口就很简单了：

```
# curl -XPOST 'http://127.0.0.1:9200/logstash-2015.06.22/_search?size=0&pretty' -d'{
    "aggs" : {
        "percentile_over_time" : {
            "date_histogram" : {
                "field"    : "@timestamp",
                "interval" : "1h"
            },
            "aggs" : {
                "percentile_one_time" : {
                    "percentiles" : {
                        "field" : "requesttime"
                    }
                }
            }
        }
    }
}'
```

得到结果如下：

```
{
  "took" : 151595,
  "timed_out" : false,
  "_shards" : {
    "total" : 81,
    "successful" : 81,
    "failed" : 0
  },
  "hits" : {
    "total" : 3307142043,
    "max_score" : 1.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "percentile_over_time" : {
      "buckets" : [ {
        "key_as_string" : "22/Jun/2015:22:00:00 +0000",
        "key" : 1435010400000,
        "doc_count" : 459273981,
        "percentile_one_time" : {
          "values" : {
            "1.0" : 0.004,
            "5.0" : 0.006,
            "25.0" : 0.023,
            "50.0" : 0.035,
            "75.0" : 0.08774675719725569,
            "95.0" : 0.25732934416125663,
            "99.0" : 0.7508899754871812
          }
        }
      }, {
        "key_as_string" : "23/Jun/2015:00:00:00 +0000",
        "key" : 1435017600000,
        "doc_count" : 768620219,
        "percentile_one_time" : {
          "values" : {
            "1.0" : 0.004,
            "5.0" : 0.007000000000000001,
            "25.0" : 0.025,
            "50.0" : 0.03987809503972864,
            "75.0" : 0.10297843567746187,
            "95.0" : 0.30047269327062875,
            "99.0" : 1.015495933753329
          }
        }
      }, {
        "key_as_string" : "23/Jun/2015:02:00:00 +0000",
        "key" : 1435024800000,
        "doc_count" : 849467060,
        "percentile_one_time" : {
          "values" : {
            "1.0" : 0.004,
            "5.0" : 0.008,
            "25.0" : 0.027000000000000003,
            "50.0" : 0.0439999899006102,
            "75.0" : 0.1160416197625958,
            "95.0" : 0.3383140614483838,
            "99.0" : 1.0275839684542212
          }
        }
      } ]
    }
  }
}
```

ES 目前能支持的聚合请求列表，参见：<https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html>。
