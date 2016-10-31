# 时序数据

之前已经介绍过，ES 默认存储数据时，是有索引数据、*_all* 全文索引数据、*_source* JSON 字符串三份的。其中，索引数据由于倒排索引的结构，压缩比非常高。因此，在某些特定环境和需求下，可以只保留索引数据，以极小的容量代价，换取 ES 灵活的数据结构和聚合统计功能。

在监控系统中，对监控项和监控数据的设计一般是这样：

> metric_path value timestamp (Graphite 设计)
> { "host": "Host name 1", "key": "item_key", "value": "33", "clock": 1381482894 } (Zabbix 设计)

这些设计有个共同点，数据是二维平面的。以最简单的访问请求状态监控为例，一次请求，可能转换出来的 `metric_path` 或者说 `key` 就有：**{city,isp,host,upstream}.{urlpath...}.{status,rt,ut,size,speed}** 这么多种。假设 urlpath 有 1000 个，就是 20000 个组合。意味着需要发送 20000 条数据，做 20000 次存储。

而在 ES 里，这就是实实在在 1000 条日志。而且在多条日志的时候，因为词元的相对固定，压缩比还会更高。所以，使用 ES 来做时序监控数据的存储和查询，是完全可行的办法。

对时序数据，关键就是定义缩减数据重复。template 示例如下：

```
{
  "order" : 2,
  "template" : "logstash-monit-*",
  "settings" : {
  },
  "mappings" : {
    "_default_" : {
      "_source" : {
        "enabled" : false
      },
      "_all" : {
        "enabled" : false
      }
    }
  },
  "aliases" : { }
}
```

如果有些字段，是完全不用 Query ，只参加 Aggregation 的，还可以设置：

```
      "properties" : {
        "sid" : {
          "index" : "no",
          "type" : "keyword"
        }
      },
```

关于 Elasticsearch 用作 rrd 用途，与 MongoDB 等其他工具的性能测试与对比，可以阅读腾讯工程师写的系列文章：<http://segmentfault.com/a/1190000002690600>
