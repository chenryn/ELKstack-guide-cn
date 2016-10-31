# script

Elasticsearch 中，可以使用自定义脚本扩展功能。包括评分、过滤函数和聚合字段等方面。内置脚本引擎历经 MVEL、Groovy、Lucene expression 的变换后，Elastic.co 最终决定实现一个自己专用的 Painless 脚本语言，并在 5.0 版正式发布。

作为 Elastic Stack 场景，我们只介绍在聚合字段方面使用 script 的方式。

## 动态提交

最简单易用的方式，就是在正常的请求体中，把 `field` 换成 `script` 提交。比如一个标准的 terms agg 改成 script 方式，写法如下：

```
# curl 127.0.0.1:9200/logstash-2015.06.29/_search -d '{
    "aggs" : {
        "clientip_top10" : {
            "terms" : {
                "script" : {
                    "lang" : "painless",
                    "inline" : "doc['clientip'].value"
                }
            }
        }
    }
}'
```

在 script 中，有三种方式引用数据：`doc['clientip'].value`、`_field['clientip'].value` 和 `_source.clientip`。其区别在于：

* `doc[].value` 读取 doc value 内的数据；
* `_field[]` 读取 field 设置 `"store":true` 的存储内容；
* `_source.obj.attr` 读取 `_source` 的 JSON 内容。

这也意味着，前者必须读取的是最终的词元字段数据，而后者可以返回任意的数据结构。

**注意**：如果有分词，且未禁用 fielddata 的话，`doc[].value` 读取到的是分词后的数据。所以请注意使用 `doc['clientip.keyword'].value` 写法。

## 固定文件

为了和动态提交的语法有区别，调用固定文件的写法如下：

```
# curl 127.0.0.1:9200/logstash-2015.06.29/_search -d '{
    "aggs" : {
        "clientip_subnet_top10" : {
            "terms" : {
                "script" : {
                   "file" : "getvalue",
                    "lang" : "groovy",
                    "params" : {
                        "fieldname": "clientip.keyword",
                        "pattern": "^((?:\d{1,3}\.?){3})\.\d{1,3}$"
                    }
                }
            }
        }
    }
}'
```

上例要求在 ES 集群的所有数据节点上，都保存有一个 `/etc/elasticsearch/scripts/getvalue.groovy` 文件，并且该脚本文件可以接收 `fieldname` 和 `pattern` 两个变量。试举例如下：

```
#!/usr/bin/env groovy
matcher = ( doc[fieldname].value =~ /${pattern}/ )
if (matcher.matches()) {
    matcher[0][1]
}
```

**注意**：ES 进程默认每分钟扫描一次 `/etc/elasticsearch/scripts/` 目录，并尝试加载该目录下所有文件作为 script。所以，不要在该目录内做文件编辑等工作，不要分发 .svn 等目录到生成环境，这些临时或者隐藏文件都会被 ES 进程加载然后报错。

## 其他语言

ES 支持通过插件方式，扩展脚本语言的支持，目前默认自带的语言包括：

* painless
* lucene expression
* groovy
* mustache

而 github 上目前已有以下语言插件支持，基本覆盖了所有 JVM 上的可用语言：

* <https://github.com/elastic/elasticsearch-lang-mvel>
* <https://github.com/elastic/elasticsearch-lang-javascript>
* <https://github.com/elastic/elasticsearch-lang-python>
* <https://github.com/hiredman/elasticsearch-lang-clojure>
* <https://github.com/felipehummel/elasticsearch-lang-scala>
* <https://github.com/fcheung/elasticsearch-jruby>
