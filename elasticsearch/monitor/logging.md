# 日志记录

Elasticsearch 作为一个服务，本身也会记录很多日志信息。默认情况下，日志都放在 `$ES_HOME/logs/` 目录里。

日志配置在 Elasticsearch 5.0 中改成了使用 `log4j2.properties` 文件配置，包括日志滚动的方式、命名等，都和标准的 log4j2 一样。唯一的特点是：Elasticsearch 导出了一个变量叫 `${sys:es.logs}`，指向你在 `elasticsearch.yml` 中配置的 `path.logs` 地址：

```
appender.index_search_slowlog_rolling.filePattern = ${sys:es.logs}_index_search_slowlog-%d{yyyy-MM-dd}.log
```

具体的级别等级也可以通过 `/_cluster/settings` 接口动态调整。比如说，如果你的节点一直无法正确的加入集群，你可以将集群自动发现方面的日志级别修改成 DEBUG，来关注这方面的问题：

```
# curl -XPUT http://127.0.0.1:9200/_cluster/settings -d'
{
    "transient" : {
        "logger.org.elasticsearch.indices.recovery" : "DEBUG"
    }
}'
```

## 性能日志

除了进程状态的日志输出，ES 还支持跟性能相关的日志输出。针对数据写入，检索，读取三个阶段，都可以设置具体的慢查询阈值，以及不同的输出等级。

此外，慢查询日志是针对索引级别的设置。除了通过 `/_cluster/settings` 接口配置一组集群各索引共用的参数以外，还可以针对每个索引设置不同的参数。

*注：过去的版本，还可以在 `elasticsearch.yml` 中设置，5.0 版禁止在配置文件中添加索引级别的设置！*

比如说，我们可以先设置集群共同的参数：

```
# curl -XPUT http://127.0.0.1:9200/_cluster/settings -d'
{
    "transient" : {
        "logger.index.search.slowlog" : "DEBUG",
        "logger.index.indexing.slowlog" : "WARN",
        "index.search.slowlog.threshold.query.debug" : "10s",
        "index.search.slowlog.threshold.fetch.debug": "500ms",
        "index.indexing.slowlog.threshold.index.warn": "5s"
    }
}'
```

然后针对某个比较大的索引，调高设置：

```
# curl -XPUT http://127.0.0.1:9200/logstash-wwwlog-2015.06.21/_settings -d'
{
    "index.search.slowlog.threshold.query.warn" : "10s",
    "index.search.slowlog.threshold.fetch.debug": "500ms",
    "index.indexing.slowlog.threshold.index.info": "10s"
}
```
