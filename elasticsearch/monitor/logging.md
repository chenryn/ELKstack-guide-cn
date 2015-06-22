# 日志记录

Elasticsearch 作为一个服务，本身也会记录很多日志信息。默认情况下，日志都放在 `$ES_HOME/logs/` 目录里。

日志级别可以通过 `elasticsearch.yml` 配置设置，也可以通过 `/_cluster/settings` 接口动态调整。比如说，如果你的节点一直无法正确的加入集群，你可以将集群自动发现方面的日志级别修改成 DEBUG，来关注这方面的问题：

```
# curl -XPUT http://127.0.0.1:9200/_cluster/settings -d'
{
    "transient" : {
        "logger.discovery" : "DEBUG"
    }
}'
```

## 性能日志

除了进程状态的日志输出，ES 还支持跟性能相关的日志输出。针对数据写入，检索，读取三个阶段，都可以设置具体的慢查询阈值，以及不同的输出等级。

此外，慢查询日志是针对索引级别的设置。除了在 `elasticsearch.yml` 中设置(注意，默认是全注释不开启的状态)以及通过 `/_cluster/settings` 接口配置一组集群各索引共用的参数以外，还可以针对每个索引设置不同的参数。

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
