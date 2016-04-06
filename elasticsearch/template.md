# 映射与模板的定制

Elasticsearch 是一个 schema-less 的系统，但 schema-less 并不代表 no schema，而是 ES 会尽量根据 JSON 源数据的基础类型猜测你想要的字段类型映射。如果你对这种动态生成的映射关系不满意，或者想要使用一些更高级的映射设置，那么就需要使用自定义映射。

## 创建和更新映射

正如上面所说，ES 可以随时根据数据中的新字段来创建新的映射关系。所以，我们也可以自己在还没有正式数据写入之前，先创建一个基础的映射。等后续数据有其他字段时，ES 也一样会自动处理。

映射的的创建方式如下：

```
# curl -XPUT http://127.0.0.1:9200/logstash-2015.06.20/_mapping -d '
{
  "mappings": {
    "syslog" : {
      "properties" : {
        "@timestamp" : {
          "type" : "date"
        },
        "message" : {
          "type" : "string"
        },
        "pid" : {
          "type" : "long"
        }
      }
    }
  }
}'
```

注意：对于已存在的映射，ES 的自动处理仅限于新字段出现。已经生成的字段映射，是不可变更的。如果确实需要，请参阅之前的 reindex 接口小节，采用重新导入数据的方式完成。

而如果是新增一个字段映射的更新，那还是可以通过 `/_mapping` 接口直接完成的：

```
# curl -XPUT http://127.0.0.1:9200/logstash-2015.06.21/_mapping/syslog -d '
{
  "properties" : {
    "syslogtag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}'
```

没错，这里只需要单独写这个新字段的内容就够了。ES 会自动合并进去。

## 删除映射

删除数据并不代表会删除数据的映射。比如：

```
# curl -XDELETE http://127.0.0.1:9200/logstash-2015.06.21/syslog
```

删除了索引下 syslog 的全部数据，但是 syslog 的映射还在。删除映射(同时也就删掉了数据)的命令是：

```
# curl -XDELETE http://127.0.0.1:9200/logstash-2015.06.21/_mapping/syslog
```

当然，如果删除整个索引，那映射也是同时被清除的。

## 核心类型

mapping 中主要就是针对字段设置类型以及类型相关参数。那么，我们首先来了解一下 Elasticsearch 支持的核心类型：

1. JSON 基础类型

* 字符串: string
* 数字: byte, short, integer, long, float, double
* 时间: date
* 布尔值: true, false
* 数组: array
* 对象: object

2. ES 独有类型

* 多重: multi
* 经纬度: geo\_point
* 网络地址: ip
* 堆叠对象: nested object
* 二进制: binary
* 附件: attachment

前面提到，ES 是根据收到的 JSON 数据里的类型来猜测的。所以，一个内容为 `"123"` 的数据，猜测出来的类型应该是 string 而不是 long。除非这个字段已经有了确定为 long 的映射关系，那么 ES 会尝试做一次转换。如果转换失败，这条数据写入就会报错。

**注意：**

ES 的映射虽然有 index 和 type 两层关系，但是实际索引时是以 index 为基础的。如果同一个 index 下不同 type 的字段出现 mapping 不一致的情况，虽然数据依然可以成功写入并生成各自的 mapping，但实际 fielddata 中的索引结果却依然是以 index 内第一个 mapping 类型来生成的。这种情况下可能会有比较奇怪的事情发生。比如：

* 看似 double 的数据实际存储成 long，导致数值比较的搜索结果异常。
* 同样的字段，一部分采用标准分词器，一部分采用中文分词器，导致索引查询异常。

从 Kibana4 开始，会在 Object Setting 页对该情况做出冲突预警；并从 ES 2.0 版本开始正式拒绝这种冲突数据写入。

## 查看已有数据的映射

学习索引映射最直接的方式，就是查看已有数据索引的映射。我们用 logstash 写入 ES 的数据，都会根据 logstash 自带的 template，生成一个很有学习意义的映射：

```
# curl -XGET http://127.0.0.1:9200/logstash-2015.06.16/_mapping/tweet
{
   "gb": {
      "mappings": {
         "tweet": {
            "properties": {
               "date": {
                  "type": "date",
                  "format": "dateOptionalTime"
               },
               "name": {
                  "type": "string"
               },
               "tweet": {
                  "type": "string"
               },
               "user_id": {
                  "type": "long"
               }
            }
         }
      }
   }
}
```

## 自定义字段映射

大家可以通过上面一个现存的映射发现其实所有的字段都有好几个属性，这些都是我们可以自己定义修改的。除了已经看到的这些基本内容外，ES 还支持其他一些可能会比较常用的映射属性：

* 全文索引还是精确索引
* 自定义分词器
* 自定义日期格式

### 精确索引

字段都有几个基本的映射选项，类型(type)和索引方式(index)。以字符串类型为例，index 有三个可设置项：

* analyzed
  默认选项，以标准的全文索引方式，分析字符串，完成索引。
* not\_analyzed
  精确索引，不对字符串做分析，直接索引字段内数据的精确内容。
* no
  不索引该字段。

对于日志应用来说，很多字段都是不需要在 Elasticsearch 里做解析这步的，所以，我们可以设置：

```
    "myfieldname": {
        "type":     "string",
        "index":    "not_analyzed"
    }
```

### 时间格式

稍微见过 ELK Stack 示例的人，都对其中 `@timestamp` 字段的特殊格式有深刻的印象。这个时间格式在 Nginx 中叫 `$time_iso8601`，在 Rsyslog 中叫 `date-rfc3339`，在 ES 中叫 `dateOptionalTime`。但事实上，ES 完全可以接收其他时间格式作为时间字段的内容。对于 ES 来说，时间字段内容实际都是转换成 long 类型作为内部存储的。所以，接收段的时间格式，可以任意配置：

```
"@timestamp" : {
    "type" : "date"
    "index" : "not_analyzed",
    "doc_values" : true,
    "format" : "dd/MMM/YYYY:HH:mm:ss Z",
}
```

而 ES 默认的时间字段格式，除了 `dateOptionalTime` 以外，还有一种，就是 `epoch_millis`，毫秒级的 UNIX 时间戳。因为这个数值 ES 可以直接毫不修改的存成内部实际的 long 数值。此外，从 ES 2.0 开始，新增了对秒级 UNIX 时间戳的支持，其 `format` 定义为：`epoch_second`。

注意：在 ES 2.x 中，同名 date 字段的 `format` 也必须保持一致。

### 多重索引

多重索引是 logstash 用户最习惯的一个映射，因为这是 logstash 默认设置开启的配置：

```
"title": {
    "type": "string",
    "fields": {
        "raw": { "type": "string", "index": "not_analyzed" }
    }
}
```

其作用是，在 *title* 字段数据写入的时候，ES 会自动生成两个字段，分别是 `title` 和 `title.raw`。这样，在可能同时需要分词与不分词结果的环境下，就可以很灵活的使用不同的索引字段了。比如，查看标题中最常用的单词，应该使用 `title` 字段；查看阅读数最多的文章标题，应该使用 `title.raw` 字段。

注意：raw 这个名字你可以自己随意取。比如说，如果你绝大多数时候用的是精确索引，那么你完全可以为了方便反过来定义：

```
"title": {
    "type": "string",
    "index": "not_analyzed",
    "fields": {
        "alz": { "type": "string" }
    }
}
```

## 特殊字段

上面介绍的，都是对普通数据字段的一些常用设置。而实际上，ES 默认还有一些特殊字段，在默默的发挥着作用。这些字段，统一以 `_` 下划线开头。在之前 CRUD 章节中，我们就已经看到一些，比如 `_index`，`_type`，`_id`。默认不开启的还有 `_ttl`，`_timestamp`，`_size`，`_parent` 等。这里需要单独介绍两个，对我们索引和检索的效果和性能，都有较大影响的：

### _all

`_all` 里存储了各字段的数据内容。其作用是，在检索的时候，如果无法或者未指明具体搜索哪个字段的数据，那么 ES 默认就会是从 `_all` 里去查找。
 
对于日志场景，如果你的日志划分出来的字段比较少且数目固定。那么，完全可以关闭掉 `_all` 功能，节省这部分 IO 和 CPU。

```
"_all" : {
    "enabled" : false
}
```

### _source

`_source` 里存储了该条记录的 JSON 源数据内容。这部分内容只是按照 ES 接收到的内容原样存储下来，并不经过索引过程。对于 ES 的请求过程来说，它不参与 Query 阶段，而只用于 Fetch 阶段。我们在 GET 或者 `/_search` 时看到的数据内容，都是从 `_source` 里获取到的。

所以，虽然 `_source` 也重复了一遍索引中的数据，一般我们并不建议关闭这个功能。因为一旦关闭，你搜索的结果除了一个 `_id`，啥都看不到。对于日志场景，意义不是很大。

当然，也有少数场景是可以关闭 `_source` 的：

1. 把 ES 作为时间序列数据库使用，只要聚合统计结果，不要源数据内容。
2. 把 ES 作为纯检索工具使用，`_id` 对应的内容在 HDFS 上另外存储，搜索后使用所得 `_id` 去 HDFS 上读取内容。

## 动态模板映射

不想使用默认识别的结果，单独设置一个字段的映射的方法，上面已经介绍完毕。那么，如果你有一类相似的数据字段，想要统一设置其映射，就可以用到下一项功能：动态模板映射(`dynamic_templates`)。

```
    "_default_" : {
      "dynamic_templates" : [ {
        "message_field" : {
          "mapping" : {
            "index" : "analyzed",
            "omit_norms" : true,
            "store" : false,
            "type" : "string"
          },
          "match" : "*msg",
          "match_mapping_type" : "string"
        }
      }, {
        "string_fields" : {
          "mapping" : {
            "index" : "not_analyzed",
            "ignore_above" : 256,
            "store" : false,
            "doc_values" : true,
            "type" : "string"
          },
          "match" : "*",
          "match_mapping_type" : "string"
        }
      } ],
      "properties" : {
      }
    }
```

这样，只要字符串类型字段名以 msg 结尾的，都会经过全文索引，其他字符串字段则进行精确索引。同理，还可以继续书写其他类型(`long`, `float`, `date` 等)的 `match_mapping_type` 和 `match`。

## 索引模板

对每个希望自定义映射的索引，都要定时提前通过发送 PUT 请求的方式创建索引的话，未免太过麻烦。ES 对此设计了索引模板功能。我们可以针对同一类索引，定制相同的模板。

模板中的内容包括两大类，setting(设置)和 mapping(映射)。setting 部分，多为在 `elasticsearch.yml` 中可以设置全局配置的部分，而 mapping 部分，则是这节之前介绍的内容。

如下为定义所有以 te 开头的索引的模板：

```
# curl -XPUT http://localhost:9200/_template/template_1 -d '
{
    "template" : "te*",
    "settings" : {
        "number_of_shards" : 1
    },
    "mappings" : {
        "type1" : {
            "_source" : { "enabled" : false }
        }
    }
}'
```

同时，索引模板是有序合并的。如果我们在同一类索引里，又想单独修改某一小类索引的一两处单独设置，可以再累加一层模板：

```
# curl -XPUT http://localhost:9200/_template/template_2 -d '
{
    "order" : 1,
    "template" : "tete*",
    "settings" : {
        "number_of_shards" : 2
    },
    "mappings" : {
        "type1" : {
            "_all" : { "enabled" : false }
        }
    }
}'
```

默认的 order 是 0，那么新创建的 order 为 1 的 *template_2* 在合并时优先级大于 *template_1*。最终，对 `tete*/type1` 的索引模板效果相当于：

```
{
    "settings" : {
        "number_of_shards" : 2
    },
    "mappings" : {
        "type1" : {
            "_source" : { "enabled" : false },
            "_all" : { "enabled" : false }
        }
    }
}
```
