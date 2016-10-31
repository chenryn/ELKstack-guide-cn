## Watcher 产品

Watcher 也是 Elastic.co 公司的商业产品，和 Shield，Marvel 一样插件式安装即可：

```
bin/plugin -i elasticsearch/license/latest
bin/plugin -i elasticsearch/watcher/latest
```

Watcher 使用方面，也提供标准的 RESTful 接口，示例如下：

```
# curl -XPUT http://127.0.0.1:9200/_watcher/watch/error_status -d'
{
    "trigger": {
        "schedule" : { "cron" : "0/5 * * * * ?" }
    },
    "input" : {
        "search" : {
            "request" : {
                "indices" : [ "<logstash-{now/d}>", "<logstash-{now/d-1d}>" ],
                "body" : {
                    "query" : {
                        "filtered" : {
                            "query" : { "match" : { "status" : "error" }},
                            "filter" : { "range" : { "@timestamp" : { "from" : "now-5m" }}}
                        }
                    }
                }
            }
        }
    },
    "condition" : {
        "compare" : { "ctx.payload.hits.total" : { "gt" : 0 }}
    },
    "transform" : {
        "search" : {
            "request" : {
                "indices" : [ "<logstash-{now/d}>", "<logstash-{now/d-1d}>" ],
                "body" : {
                    "query" : {
                        "filtered" : {
                            "query" : { "match" : { "status" : "error" }},
                            "filter" : { "range" : { "@timestamp" : { "from" : "now-5m" }}}
                        }
                    },
                    "aggs" : {
                        "topn" : {
                            "terms" : {
                                "field" : "userid"
                            }
                        }
                    }
                }
            }
        }
    },
    "actions" : {
        "email_admin" : {
            "throttle_period" : "15m",
            "email" : {
                "to" : "admin@domain",
                "subject" : "Found {{ctx.payload.hits.total}} Error Events at {{ctx.trigger.triggered_time}}",
                "priority" : "high",
                "body" : "Top10 users:\n{{#ctx.payload.aggregations.topn.buckets}}\t{{key}} {{doc_count}}\n{{/ctx.payload.aggregations.topn.buckets}}"
            }
        }
    }
}'
```

上面这行命令，意即:

1. 每 5 分钟，向最近两天的 `logstash-yyyy.MM.dd` 索引发起一次条件为最近五分钟，status 字段内容为 error 的查询请求;
2. 对查询结果做 hits 总数大于 0 的判断;
3. 如果为真，再请求一次上述条件下，userid 字段的 Top 10 数据集作为后续处理的来源;
4. 如果最近 15 分钟内未发送过报警，则向 `admin@domain` 邮箱发送一个标题为 "Found N erroneous events at yyyy-MM-ddTHH:mm:ssZ"，内容为 "Top10 users" 列表的报警邮件。

整个请求体顺序执行。目前 trigger 只支持 scheduler 方式(但是 schedule 下有 crontab、interval、hourly、daily、weekly、monthly、yearly 等多种写法)，input 支持 search 和 http 方式，actions 支持 email，logging，webhook 方式，transform 是可选项，而且可以设置在 actions 里，不同 actions 做不同的 payload 转换。

crontab 定义语法和 Linux 标准不太一致，采用的是 Quartz，文件见：<http://www.quartz-scheduler.org/documentation/quartz-1.x/tutorials/crontrigger>。

condition, transform 和 actions 中，默认使用 Watcher 增强版的 xmustache 模板语言（示例中的数组循环就是一例）。也可以使用固化的脚本文件，比如有 `threshold_hits.groovy` 的话，可以执行：

```
  "condition" : {
    "script" : {
      "file" : "threshold_hits",
      "params" : {
        "threshold" : 0
      }
    }
  }
```

Watcher 中可用的 `ctx` 变量包括：

* `ctx.watch_id`
* `ctx.execution_time`
* `ctx.trigger.triggered_time`
* `ctx.trigger.scheduled_time`
* `ctx.metadata.*`
* `ctx.payload.*`

完整的 Watcher 插件内部执行流程如下图。相信有编程能力的读者都可以用 crontab/at 配合 curl，email 工具仿造出来类似功能的 shell 脚本。

![](https://www.elastic.co/guide/en/watcher/current/images/watch-execution.jpg)

**注意**：

在 search 中，对 indices 内容可以写完整的索引名比如 `syslog`，也可以写通配符比如 `logstash-*`，也可以写时序索引动态定义方式如 `<logstash-{now/d}>`。而这个动态定义，Watcher 是支持根据时区来确定的，这个需要在 `elasticsearch.yml` 里配置一行：

```
watcher.input.search.dynamic_indices.time_zone: '+08:00'
```

笔者仿照 watcher 的配置语法，开源了一个基于 Kibana 扩展的类 watcher 监控项目，本书稍后 Kibana 章节将会有详细介绍。
