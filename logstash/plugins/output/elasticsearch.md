# 保存进 Elasticsearch

Logstash 可以使用不同的协议实现完成将数据写入 Elasticsearch 的工作。在不同时期，也有不同的插件实现方式。本节以最新版为准，即主要介绍 HTTP 方式。同时也附带一些原有的 node 和 transport 方式的介绍。

## 配置示例

```
output {
    elasticsearch {
        hosts => ["192.168.0.2:9200"]
        index => "logstash-%{type}-%{+YYYY.MM.dd}"
        document_type => "%{type}"
        flush_size => 20000
        idle_flush_time => 10
        sniffing => true
        template_overwrite => true
    }
}
```

## 解释

### 批量发送

在过去的版本中，主要由本插件的 `flush_size` 和 `idle_flush_time` 两个参数共同控制 Logstash 向 Elasticsearch 发送批量数据的行为。以上面示例来说：Logstash 会努力攒到 20000 条数据一次性发送出去，但是如果 10 秒钟内也没攒够 20000 条，Logstash 还是会以当前攒到的数据量发一次。

默认情况下，`flush_size` 是 500 条，`idle_flush_time` 是 1 秒。这也是很多人改大了 `flush_size` 也没能提高写入 ES 性能的原因——Logstash 还是 1 秒钟发送一次。

从 5.0 开始，这个行为有了另一个前提：`flush_size` 的大小不能超过 Logstash 运行时的命令行参数设置的 `batch_size`，否则将以 `batch_size` 为批量发送的大小。

### 索引名

写入的 ES 索引的名称，这里可以使用变量。为了更贴合日志场景，Logstash 提供了 `%{+YYYY.MM.dd}` 这种写法。在语法解析的时候，看到以 + 号开头的，就会自动认为后面是时间格式，尝试用时间格式来解析后续字符串。所以，之前处理过程中不要给自定义字段取个加号开头的名字……

此外，注意索引名中不能有大写字母，否则 ES 在日志中会报 *InvalidIndexNameException*，但是 Logstash 不会报错，这个错误比较隐晦，也容易掉进这个坑中。

### 轮询

Logstash 1.4.2 在 transport 和 http 协议的情况下是固定连接指定 host 发送数据。从 1.5.0 开始，host 可以设置数组，它会从节点列表中选取不同的节点发送数据，达到 Round-Robin 负载均衡的效果。

## 不同版本的协议沿革

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

对于拥有很多索引的大集群，你可以用 *transport* 协议。logstash 进程会转发所有数据到你指定的某台主机上。这种协议跟上面的 *node* 协议是不同的。*node* 协议下的进程是可以接收到整个 Elasticsearch 集群状态信息的，当进程收到一个事件时，它就知道这个事件应该存在集群内哪个机器的分片里，所以它就会直接连接该机器发送这条数据。而 *transport* 协议下的进程不会保存这个信息，在集群状态更新(节点变化，索引变化都会发送全量更新)时，就不会对所有的 logstash 进程也发送这种信息。更多 Elasticsearch 集群状态的细节，参阅本书后续章节。

#### 小贴士

* 经常有同学问，为什么 Logstash 在有多个 conf 文件的情况下，进入 ES 的数据会重复，几个 conf 数据就会重复几次。其实问题原因在之前[启动参数章节](../../get_start/full_config.md)有提过，output 段顺序执行，没有对日志 type 进行判断的各插件配置都会全部执行一次。在 output 段对 type 进行判断的语法如下所示：
```
output {
  if [type] == "nginxaccess" {
    elasticsearch { }
  }
}
```

### 模板

Elasticsearch 支持给索引预定义设置和 mapping(前提是你用的 elasticsearch 版本支持这个 API，不过估计应该都支持)。Logstash 自带有一个优化好的模板，内容如下:

```json
{
    "template" : "logstash-*",
    "version" : 50001,
    "settings" : {
        "index.refresh_interval" : "5s"
    },
    "mappings" : {
        "_default_" : {
            "_all" : {"enabled" : true, "norms" : false},
            "dynamic_templates" : [ {
                "message_field" : {
                    "path_match" : "message",
                    "match_mapping_type" : "string",
                    "mapping" : {
                        "type" : "text",
                        "norms" : false
                    }
                }
            }, {
                "string_fields" : {
                    "match" : "*",
                    "match_mapping_type" : "string",
                    "mapping" : {
                        "type" : "text", "norms" : false,
                        "fields" : {
                            "keyword" : { "type": "keyword"  }
                        }
                    }
                }
            }  ],
            "properties" : {
                "@timestamp": { "type": "date", "include_in_all": false  },
                "@version": { "type": "keyword", "include_in_all": false  },
                "geoip"  : {
                    "dynamic": true,
                    "properties" : {
                        "ip": { "type": "ip"  },
                        "location" : { "type" : "geo_point"  },
                        "latitude" : { "type" : "half_float"  },
                        "longitude" : { "type" : "half_float"  }
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

* multi-field with keyword

Elasticsearch 会自动使用自己的默认分词器(空格，点，斜线等分割)来分析字段。分词器对于搜索和评分是非常重要的，但是大大降低了索引写入和聚合请求的性能。所以 logstash 模板定义了一种叫"多字段"(multi-field)类型的字段。这种类型会自动添加一个 ".keyword" 结尾的字段，并给这个字段设置为不启用分词器。简单说，你想获取 url 字段的聚合结果的时候，不要直接用 "url" ，而是用 "url.keyword" 作为字段名。当你还对分词字段发起聚合和排序请求的时候，直接提示无法构建 fielddata 了！

在 Logstash 5.0 中，同时还保留携带了针对 Elasticsearch 2.x 的 template 文件，在那里，通过旧版本的 mapping 配置，达到和新版本相同的行为效果：对应统计字段明确设置 `"index":"not_analyzed","doc_values":true`，以及对分词字段加上对 fielddata 的 `{"format":"disabled"}`。

* geo\_point

Elasticsearch 支持 *geo_point* 类型， *geo distance* 聚合等等。比如说，你可以请求某个 *geo_point* 点方圆 10 千米内数据点的总数。在 Kibana 的 tilemap 类型面板里，就会用到这个类型的数据。

* half\_float

Elasticsearch 5.0 新引入了 *half_float* 类型。比标准的 *float* 类型占用更少的资源，提供更好的性能。在明确自己数值范围较小的时候可用。刚巧，经纬度就是一个明确的数值范围很小的数据。

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
