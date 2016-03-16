# 保存进 Elasticsearch

Logstash 可以试用不同的协议实现完成将数据写入 Elasticsearch 的工作。在不同时期，也有不同的插件实现方式。本节以最新版为准，即主要介绍 HTTP 方式。同时也附带一些原有的 node 和 transport 方式的介绍。

## 配置示例

```
output {
    elasticsearch {
        hosts => ["192.168.0.2:9200"]
        index => "logstash-%{type}-%{+YYYY.MM.dd}"
        document_type => "%{type}"
        workers => 1
        flush_size => 20000
        idle_flush_time => 10
        template_overwrite => true
    }
}
```

## 解释

### 批量发送

`flush_size` 和 `idle_flush_time` 共同控制 Logstash 向 Elasticsearch 发送批量数据的行为。以上面示例来说：Logstash 会努力攒到 20000 条数据一次性发送出去，但是如果 10 秒钟内也没攒够 20000 条，Logstash 还是会以当前攒到的数据量发一次。

默认情况下，`flush_size` 是 500 条，`idle_flush_time` 是 1 秒。这也是很多人改大了 `flush_size` 也没能提高写入 ES 性能的原因——Logstash 还是 1 秒钟发送一次。

### 索引名

写入的 ES 索引的名称，这里可以使用变量。为了更贴合日志场景，Logstash 提供了 `%{+YYYY.MM.dd}` 这种写法。在语法解析的时候，看到以 + 号开头的，就会自动认为后面是时间格式，尝试用时间格式来解析后续字符串。所以，之前处理过程中不要给自定义字段取个加号开头的名字……

此外，注意索引名中不能有大写字母，否则 ES 在日志中会报 *InvalidIndexNameException*，但是 Logstash 不会报错，这个错误比较隐晦，也容易掉进这个坑中。

### Java 协议

1.4.0 版本之前，有 `logstash-output-elasticsearch`, `logstash-output-elasticsearch_http`, `logstash-output-elasticsearch_river` 三个插件。

1.4.0 到 2.0 版本之间，配合 Elasticsearch 废弃 river 方法，只剩下 `logstash-output-elasticsearch` 一个插件，同时实现了 node、transport、http 三种协议。

2.0 版本开始，为了兼容性和调试方便，`logstash-output-elasticsearch` 改为只支持 *http* 协议。想继续使用 *node* 或者 *transport* 协议的用户，需要单独安装 `logstash-output-elasticsearch_java` 插件。

一个小集群里，使用 *node* 协议最方便了。Logstash 以 elasticsearch 的 client 节点身份(即不存数据不参加选举)运行。如果你运行下面这行命令，你就可以看到自己的 logstash 进程名，对应的 `node.role` 值是 **c**：

```
# curl 127.0.0.1:9200/_cat/nodes?v
host       ip      heap.percent ram.percent load node.role master name
local 192.168.0.102  7      c         -      logstash-local-1036-2012
local 192.168.0.2    7      d         *      Sunstreak

```

Logstash-1.5 以后，也不再分发一个**内嵌**的 elasticsearch 服务器。如果你想变更 node 协议下的这些配置，在 `$PWD/elasticsearch.yml` 文件里写自定义配置即可，logstash 会尝试自动加载这个文件。

对于拥有很多索引的大集群，你可以用 *transport* 协议。logstash 进程会转发所有数据到你指定的某台主机上。这种协议跟上面的 *node* 协议是不同的。*node* 协议下的进程是可以接收到整个 Elasticsearch 集群状态信息的，当进程收到一个事件时，它就知道这个事件应该存在集群内哪个机器的分片里，所以它就会直接连接该机器发送这条数据。而 *transport* 协议下的进程不会保存这个信息，在集群状态更新(节点变化，索引变化都会发送全量更新)时，就不会对所有的 logstash 进程也发送这种信息。更多 Elasticsearch 集群状态的细节，参阅<http://www.elasticsearch.org/guide>。

#### 小贴士

* Logstash 1.4.2 在 transport 和 http 协议的情况下是固定连接指定 host 发送数据。从 1.5.0 开始，host 可以设置数组，它会从节点列表中选取不同的节点发送数据，达到 Round-Robin 负载均衡的效果。
* Kibana4 强制要求 ES 全集群所有 node 版本在 1.4 以上，Kibana4.2 要求 ES 2.0 以上。所以采用 node 方式发送数据的 logstash-1.4(携带的 Elasticsearch.jar 库是 1.1.1 版本) 会导致 Kibana4 无法运行，采用 Kibana4 的读者务必改用 http 方式。
* 开发者在 IRC freenode#logstash 频道里表示："高于 1.0 版本的 Elasticsearch 应该都能跟最新版 logstash 的 node 协议一起正常工作"。此信息仅供参考，请认真测试后再上线。
* 经常有同学问，为什么 Logstash 在有多个 conf 文件的情况下，进入 ES 的数据会重复，几个 conf 数据就会重复几次。其实问题原因在之前[启动参数章节](../../get_start/full_config.md)有提过，output 段顺序执行，没有对日志 type 进行判断的各插件配置都会全部执行一次。在 output 段对 type 进行判断的语法如下所示：
```
output {
  if [type] == "nginxaccess" {
    elasticsearch { }
  }
}
```

#### 老版本的性能问题

Logstash 1.4.2 在 http 协议下默认使用作者自己的 ftw 库，随同分发的是 0.0.39 版。该版本有[内存泄露问题](https://github.com/elasticsearch/logstash/issues/1604)，长期运行下输出性能越来越差！

解决办法：

1. 对性能要求不高的，可以在启动 logstash 进程时，配置环境变量 `ENV["BULK"]`，强制采用 elasticsearch 官方 Ruby 库。命令如下：

    export BULK="esruby"

2. 对性能要求高的，可以尝试采用 logstash-1.5.0RC2 。新版的 outputs/elasticsearch 放弃了 ftw 库，改用了一个 JRuby 平台专有的 [Manticore 库](https://github.com/cheald/manticore/wiki/Performance)。根据测试，性能跟 ftw 比[相当接近](https://github.com/elasticsearch/logstash/pull/1777)。
3. 对性能要求极高的，可以手动更新 ftw 库版本，目前最新版是 0.0.42 版，据称内存问题在 0.0.40 版即解决。

### 模板

Elasticsearch 支持给索引预定义设置和 mapping(前提是你用的 elasticsearch 版本支持这个 API，不过估计应该都支持)。Logstash 自带有一个优化好的模板，内容如下:

```json
{
  "template" : "logstash-*",
  "settings" : {
    "index.refresh_interval" : "5s"
  },
  "mappings" : {
    "_default_" : {
      "_all" : {"enabled" : true, "omit_norms" : true},
      "dynamic_templates" : [ {
        "message_field" : {
          "match" : "message",
          "match_mapping_type" : "string",
          "mapping" : {
            "type" : "string", "index" : "analyzed", "omit_norms" : true,
            "fielddata" : { "format" : "disabled"  }
          }
        }
      }, {
        "string_fields" : {
          "match" : "*",
          "match_mapping_type" : "string",
          "mapping" : {
            "type" : "string", "index" : "analyzed", "omit_norms" : true,
            "fielddata" : { "format" : "disabled"  },
            "fields" : {
              "raw" : {"type": "string", "index" : "not_analyzed", "doc_values" : true, "ignore_above" : 256}
            }
          }
        }
      }, {
        "float_fields" : {
          "match" : "*",
          "match_mapping_type" : "float",
          "mapping" : { "type" : "float", "doc_values" : true  }
        }
      }, {
        "double_fields" : {
          "match" : "*",
          "match_mapping_type" : "double",
          "mapping" : { "type" : "double", "doc_values" : true  }
        }
      }, {
        "byte_fields" : {
          "match" : "*",
          "match_mapping_type" : "byte",
          "mapping" : { "type" : "byte", "doc_values" : true  }
        }
      }, {
        "short_fields" : {
          "match" : "*",
          "match_mapping_type" : "short",
          "mapping" : { "type" : "short", "doc_values" : true  }
        }
      }, {
        "integer_fields" : {
          "match" : "*",
          "match_mapping_type" : "integer",
          "mapping" : { "type" : "integer", "doc_values" : true  }
        }
      }, {
        "long_fields" : {
          "match" : "*",
          "match_mapping_type" : "long",
          "mapping" : { "type" : "long", "doc_values" : true  }
        }
      }, {
        "date_fields" : {
          "match" : "*",
          "match_mapping_type" : "date",
          "mapping" : { "type" : "date", "doc_values" : true  }
        }
      }, {
        "geo_point_fields" : {
          "match" : "*",
          "match_mapping_type" : "geo_point",
          "mapping" : { "type" : "geo_point", "doc_values" : true  }
        }
      } ],
      "properties" : {
        "@timestamp": { "type": "date", "doc_values" : true  },
        "@version": { "type": "string", "index": "not_analyzed", "doc_values" : true  },
        "geoip"  : {
          "type" : "object",
          "dynamic": true,
          "properties" : {
            "ip": { "type": "ip", "doc_values" : true  },
            "location" : { "type" : "geo_point", "doc_values" : true  },
            "latitude" : { "type" : "float", "doc_values" : true  },
            "longitude" : { "type" : "float", "doc_values" : true  }
          }
        }
      }
    }
  }
}
```

这其中的关键设置包括：

* template for index-pattern

只有匹配 `logstash-*` 的索引才会应用这个模板。有时候我们会变更 Logstash 的默认索引名称，记住你也得通过 PUT 方法上传可以匹配你自定义索引名的模板。当然，我更建议的做法是，把你自定义的名字放在 "logstash-" 后面，变成 `index => "logstash-custom-%{+yyyy.MM.dd}"` 这样。

* refresh\_interval for indexing

Elasticsearch 是一个*近*实时搜索引擎。它实际上是每 1 秒钟刷新一次数据。对于日志分析应用，我们用不着这么实时，所以 logstash 自带的模板修改成了 5 秒钟。你还可以根据需要继续放大这个刷新间隔以提高数据写入性能。

* multi-field with not\_analyzed

Elasticsearch 会自动使用自己的默认分词器(空格，点，斜线等分割)来分析字段。分词器对于搜索和评分是非常重要的，但是大大降低了索引写入和聚合请求的性能。所以 logstash 模板定义了一种叫"多字段"(multi-field)类型的字段。这种类型会自动添加一个 ".raw" 结尾的字段，并给这个字段设置为不启用分词器。简单说，你想获取 url 字段的聚合结果的时候，不要直接用 "url" ，而是用 "url.raw" 作为字段名。

* geo\_point

Elasticsearch 支持 *geo_point* 类型， *geo distance* 聚合等等。比如说，你可以请求某个 *geo_point* 点方圆 10 千米内数据点的总数。在 Kibana 的 tilemap 类型面板里，就会用到这个类型的数据。

* doc\_values

doc\_values 是 Elasticsearch 1.0 版本引入的新特性。启用该特性的字段，索引写入的时候会在磁盘上构建 fielddata。而过去，fielddata 是固定只能使用内存的。在请求范围加大的时候，很容易触发 OOM 或者 circuit breaker 报错：

> ElasticsearchException[org.elasticsearch.common.breaker.CircuitBreakingException: Data too large, data for field [@timestamp] would be larger than limit of [639015321/609.4mb]]

doc\_values 只能给不分词(对于字符串字段就是设置了 `"index":"not_analyzed"`，数值和时间字段默认就没有分词) 的字段配置生效。

doc\_values 虽然用的是磁盘，但是系统本身也有自带 VFS 的 cache 效果并不会太差。据官方测试，经过 1.4 的优化后，只比使用内存的 fielddata 慢 15% 。所以，在数据量较大的情况下，**强烈建议开启**该配置。

**Elasticsearch 2.0 以后，`doc_values` 变成默认设置。这部分可以不再单独指定了。**

* fielddata

和 doc_values 对应的，则是 fielddata。在 Elasticsearch 2.x 全面启用 doc_values 后，Logstash 的默认 template 更干脆的加上了对 fielddata 的 `{"format":"disabled"}`。当你还对分词字段发起聚合和排序请求的时候，直接提示无法构建 fielddata 了！

#### 其他模板配置建议

* order

如果你有自己单独定制 template 的想法，很好。这时候有几种选择：

1. 在 logstash/outputs/elasticsearch 配置中开启 `manage_template => false` 选项，然后一切自己动手；
2. 在 logstash/outputs/elasticsearch 配置中开启 `template => "/path/to/your/tmpl.json"` 选项，让 logstash 来发送你自己写的 template 文件；
3. 避免变更 logstash 里的配置，而是另外发送一个 template ，利用 elasticsearch 的 templates order 功能。

这个 order 功能，就是 elasticsearch 在创建一个索引的时候，如果发现这个索引同时匹配上了多个 template ，那么就会先应用 order 数值小的 template 设置，然后再应用一遍 order 数值高的作为覆盖，最终达到一个 merge 的效果。

比如，对上面这个模板已经很满意，只想修改一下 `refresh_interval` ，那么只需要新写一个：

```json
{
  "order" : 1,
  "template" : "logstash-*",
  "settings" : {
    "index.refresh_interval" : "20s"
  }
}
```

然后运行 `curl -XPUT http://localhost:9200/_template/template_newid -d '@/path/to/your/tmpl.json'` 即可。

logstash 默认的模板， order 是 0，id 是 logstash，通过 logstash/outputs/elasticsearch 的配置选项 `template_name` 修改。你的新模板就不要跟这个名字冲突了。
