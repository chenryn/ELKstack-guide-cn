# rollover

Elasticsearch 从 5.0 开始，为日志场景的用户提供了一个很不错的接口，叫 rollover。其作用是：当某个别名指向的实际索引过大的时候，自动将别名指向下一个实际索引。

因为这个接口是操作的别名，所以我们依然需要首先自己创建一个开始滚动的起始索引：

```
# curl -XPUT 'http://localhost:9200/logstash-2016.11.25-1' -d '{
    "aliases": {
        "logstash": {}
    }
}'
```

然后就可以尝试发起 rollover 请求了：

```
# curl -XPOST 'http://localhost:9200/logstash/_rollover' -d '{
    "conditions": {
        "max_age":   "1d",
        "max_docs":  10000000
    }
}'
```

上面的定义意思就是：当索引超过 1 天，或者索引内的数据量超过一千万条的时候，自动创建并指向下一个索引。

这时候有几种可能性：

* 条件都没满足，直接返回一个 false，索引和别名都不发生实际变化；
```
{
    "old_index" : "logstash-2016.11.25-1",
    "new_index" : "logstash-2016.11.25-1",
    "rolled_over" : false,
    "dry_run" : false,
    "acknowledged" : false,
    "shards_acknowledged" : false,
    "conditions" : {
        "[max_docs: 10000000]" : false,
        "[max_age: 1d]" : false
    }
}
```
* 还没满一天，满了一千万条，那么下一个索引名会是：`logstash-2016.11.25-000002`；
* 还没满一千万条，满了一天，那么下一个索引名会是：`logstash-2016.11.26-000002`。

# shrink

Elasticsearch 一直以来都是固定分片数的。这个策略极大的简化了分布式系统的复杂度，但是在一些场景，比如存储 metric 的 TSDB、小数据量的日志存储，人们会期望在多分片快速写入数据以后，把老数据合并存储，节约过多的 cluster state 容量。从 5.0 版本开始，Elasticsearch 新提供了 shrink 接口，可以成倍数的合并分片数。

*注：所谓成倍数的，就是原来有 15 个分片，可以合并缩减成 5 个或者 3 个或者 1 个分片。*

整个合并缩减的操作流程，大概如下：

1. 先把所有主分片都转移到一台主机上；
2. 在这台主机上创建一个新索引，分片数较小，其他设置和原索引一致；
3. 把原索引的所有分片，复制（或硬链接）到新索引的目录下；
4. 对新索引进行打开操作恢复分片数据。
5. (可选)重新把新索引的分片均衡到其他节点上。

## 准备工作

* 因为这个操作流程需要把所有分片都转移到一台主机上，所以作为 shrink 主机，它的磁盘要足够大，至少要能放得下一整个索引。
* 最好是一整块磁盘，因为硬链接是不能跨磁盘的。靠复制太慢了。
* 开始迁移：
```
# curl -XPUT 'http://localhost:9200/metric-2016.11.25/_settings' -d '
{
    "settings": {
        "index.routing.allocation.require._name": "shrink_node_name", 
        "index.blocks.write": true 
    }
}'
```

## shrink 操作

```
curl -XPOST 'http://localhost:9200/metric-2016.11.25/_shrink/oldmetric-2016.11.25' -d'
{
    "settings": {
        "index.number_of_replicas": 1,
        "index.number_of_shards": 3
    },
    "aliases": {
        "metric-tsdb": {}
    }
}'
```

这个命令执行完会立刻返回，但是 Elasticsearch 会一直等到 shrink 操作完成的时候，才会真的开始做 replica 分片的分配和重均衡，此前分片都处于 initializing 状态。

*注意：Elasticsearch 有一个硬编码限制，单个分片内的文档总数不得超过 2147483519 个。一般来说这个限制在日志场景下是不太会触发的，但是如果做 TSDB 用，则需要多加注意！*
