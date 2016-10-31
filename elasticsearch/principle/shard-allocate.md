# shard 的 allocate 控制

某个 shard 分配在哪个节点上，一般来说，是由 ES 自动决定的。以下几种情况会触发分配动作：

1. 新索引生成
2. 索引的删除
3. 新增副本分片
4. 节点增减引发的数据均衡

ES 提供了一系列参数详细控制这部分逻辑：

* cluster.routing.allocation.enable
  该参数用来控制允许分配哪种分片。默认是 `all`。可选项还包括 `primaries` 和 `new_primaries`。`none` 则彻底拒绝分片。该参数的作用，本书稍后集群升级章节会有说明。
* cluster.routing.allocation.allow\_rebalance
  该参数用来控制什么时候允许数据均衡。默认是 `indices_all_active`，即要求所有分片都正常启动成功以后，才可以进行数据均衡操作，否则的话，在集群重启阶段，会浪费太多流量了。
* cluster.routing.allocation.cluster\_concurrent\_rebalance
  该参数用来控制**集群内**同时运行的数据均衡任务个数。默认是 2 个。如果有节点增减，且集群负载压力不高的时候，可以适当加大。
* cluster.routing.allocation.node\_initial\_primaries\_recoveries
  该参数用来控制**节点**重启时，允许同时恢复几个主分片。默认是 4 个。如果节点是多磁盘，且 IO 压力不大，可以适当加大。
* cluster.routing.allocation.node\_concurrent\_recoveries
  该参数用来控制**节点**除了主分片重启恢复以外其他情况下，允许同时运行的数据恢复任务。默认是 2 个。所以，节点重启时，可以看到主分片迅速恢复完成，副本分片的恢复却很慢。除了副本分片本身数据要通过网络复制以外，并发线程本身也减少了一半。当然，这种设置也是有道理的——主分片一定是本地恢复，副本分片却需要走网络，带宽是有限的。从 ES 1.6 开始，冷索引的副本分片可以本地恢复，这个参数也就是可以适当加大了。
* indices.recovery.concurrent\_streams
  该参数用来控制**节点**从网络复制恢复副本分片时的数据流个数。默认是 3 个。可以配合上一条配置一起加大。
* indices.recovery.max\_bytes\_per\_sec
  该参数用来控制**节点**恢复时的速率。默认是 40MB。显然是比较小的，建议加大。

此外，ES 还有一些其他的分片分配控制策略。比如以 `tag` 和 `rack_id` 作为区分等。一般来说，Elastic Stack 场景中使用不多。运维人员可能比较常见的策略有两种：

1. 磁盘限额
   为了保护节点数据安全，ES 会定时(`cluster.info.update.interval`，默认 30 秒)检查一下各节点的数据目录磁盘使用情况。在达到 `cluster.routing.allocation.disk.watermark.low` (默认 85%)的时候，新索引分片就不会再分配到这个节点上了。在达到 `cluster.routing.allocation.disk.watermark.high` (默认 90%)的时候，就会触发该节点现存分片的数据均衡，把数据挪到其他节点上去。这两个值不但可以写百分比，还可以写具体的字节数。有些公司可能出于成本考虑，对磁盘使用率有一定的要求，需要适当抬高这个配置：

```
# curl -XPUT localhost:9200/_cluster/settings -d '{
    "transient" : {
        "cluster.routing.allocation.disk.watermark.low" : "85%",
        "cluster.routing.allocation.disk.watermark.high" : "10gb",
        "cluster.info.update.interval" : "1m"
    }
}'
```

2. 热索引分片不均
   默认情况下，ES 集群的数据均衡策略是以各节点的分片总数(*indices_all_active*)作为基准的。这对于搜索服务来说无疑是均衡搜索压力提高性能的好办法。但是对于 Elastic Stack 场景，一般压力集中在新索引的数据写入方面。正常运行的时候，也没有问题。但是当集群扩容时，新加入集群的节点，分片总数远远低于其他节点。这时候如果有新索引创建，ES 的默认策略会导致新索引的所有主分片几乎全分配在这台新节点上。整个集群的写入压力，压在一个节点上，结果很可能是这个节点直接被压死，集群出现异常。
   所以，对于 Elastic Stack 场景，强烈建议大家预先计算好索引的分片数后，配置好单节点分片的限额。比如，一个 5 节点的集群，索引主分片 10 个，副本 1 份。则平均下来每个节点应该有 4 个分片，那么就配置：

```
# curl -s -XPUT http://127.0.0.1:9200/logstash-2015.05.08/_settings -d '{
    "index": { "routing.allocation.total_shards_per_node" : "5" }
}'
```

注意，这里配置的是 5 而不是 4。因为我们需要预防有机器故障，分片发生迁移的情况。如果写的是 4，那么分片迁移会失败。

此外，另一种方式则更加玄妙，Elasticsearch 中有一系列参数，相互影响，最终联合决定分片分配：

* cluster.routing.allocation.balance.shard
  节点上分配分片的权重，默认为 0.45。数值越大越倾向于在节点层面均衡分片。
* cluster.routing.allocation.balance.index
  每个索引往单个节点上分配分片的权重，默认为 0.55。数值越大越倾向于在索引层面均衡分片。
* cluster.routing.allocation.balance.threshold
  大于阈值则触发均衡操作。默认为1。

Elasticsearch 中的计算方法是：

(indexBalance * (node.numShards(index) – avgShardsPerNode(index)) + shardBalance * (node.numShards() – avgShardsPerNode)) <=> weightthreshold

所以，也可以采取加大 `cluster.routing.allocation.balance.index`，甚至设置 `cluster.routing.allocation.balance.shard` 为 0 来尽量采用索引内的节点均衡。

## reroute 接口

上面说的各种配置，都是从策略层面，控制分片分配的选择。在必要的时候，还可以通过 ES 的 reroute 接口，手动完成对分片的分配选择的控制。

reroute 接口支持三种指令：allocate，move 和 cancel。常用的一般是 allocate 和 move：

* allocate 指令

因为负载过高等原因，有时候个别分片可能长期处于 UNASSIGNED 状态，我们就可以手动分配分片到指定节点上。默认情况下只允许手动分配副本分片，所以如果是主分片故障，需要单独加一个 `allow_primary` 选项：

```
# curl -XPOST 127.0.0.1:9200/_cluster/reroute -d '{
  "commands" : [ {
        "allocate" :
            {
              "index" : "logstash-2015.05.27", "shard" : 61, "node" : "10.19.0.77", "allow_primary" : true
            }
        }
  ]
}'
```

注意，如果是历史数据的话，请提前确认一下哪个节点上保留有这个分片的实际目录，且目录大小最大。然后手动分配到这个节点上。以此减少数据丢失。

* move 指令

因为负载过高，磁盘利用率过高，服务器下线，更换磁盘等原因，可以会需要从节点上移走部分分片：

```
curl -XPOST 127.0.0.1:9200/_cluster/reroute -d '{
  "commands" : [ {
        "move" :
            {
              "index" : "logstash-2015.05.22", "shard" : 0, "from_node" : "10.19.0.81", "to_node" : "10.19.0.104"
            }
        }
  ]
}'
```

## 分配失败原因

如果是自己手工 reroute 失败，Elasticsearch 返回的响应中会带上失败的原因。不过格式非常难看，一堆 YES，NO。从 5.0 版本开始，Elasticsearch 新增了一个 allocation explain 接口，专门用来解释指定分片的具体失败理由：

```
curl -XGET 'http://localhost:9200/_cluster/allocation/explain' -d'{
      "index": "logstash-2016.10.31",
      "shard": 0,
      "primary": false

}'
```

得到的响应如下：

```
{
    "shard" : {
        "index" : "myindex",
        "index_uuid" : "KnW0-zELRs6PK84l0r38ZA",
        "id" : 0,
        "primary" : false
    },
    "assigned" : false,
    "shard_state_fetch_pending": false,
    "unassigned_info" : {
        "reason" : "INDEX_CREATED",
        "at" : "2016-03-22T20:04:23.620Z"
    },
    "allocation_delay_ms" : 0,
    "remaining_delay_ms" : 0,
    "nodes" : {
        "V-Spi0AyRZ6ZvKbaI3691w" : {
            "node_name" : "H5dfFeA",
            "node_attributes" : {
                "bar" : "baz"
            },
            "store" : {
                "shard_copy" : "NONE"
            },
            "final_decision" : "NO",
            "final_explanation" : "the shard cannot be assigned because one or more allocation decider returns a 'NO' decision",
            "weight" : 0.06666675,
            "decisions" : [ {
                "decider" : "filter",
                "decision" : "NO",
                "explanation" : "node does not match index include filters [foo:\"bar\"]"
            }  ]
        },
        "Qc6VL8c5RWaw1qXZ0Rg57g" : {
            ...
```

这会是很长一串 JSON，把集群里所有的节点都列上来，挨个解释为什么不能分配到这个节点。

## 节点下线

集群中个别节点出现故障预警等情况，需要下线，也是 Elasticsearch 运维工作中常见的情况。如果已经稳定运行过一段时间的集群，每个节点上都会保存有数量不少的分片。这种时候通过 reroute 接口手动转移，就显得太过麻烦了。这个时候，有另一种方式：

```
curl -XPUT 127.0.0.1:9200/_cluster/settings -d '{
  "transient" :{
      "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
   }
}'
```

Elasticsearch 集群就会自动把这个 IP 上的所有分片，都自动转移到其他节点上。等到转移完成，这个空节点就可以毫无影响的下线了。

和 `_ip` 类似的参数还有 `_host`, `_name` 等。此外，这类参数不单是 cluster 级别，也可以是 index 级别。下一小节就是 index 级别的用例。

## 冷热数据的读写分离

Elasticsearch 集群一个比较突出的问题是: 用户做一次大的查询的时候, 非常大量的读 IO 以及聚合计算导致机器 Load 升高, CPU 使用率上升, 会影响阻塞到新数据的写入, 这个过程甚至会持续几分钟。所以，可能需要仿照 MySQL 集群一样，做读写分离。

### 实施方案

1. N 台机器做热数据的存储, 上面只放当天的数据。这 N 台热数据节点上面的 elasticsearc.yml 中配置 `node.tag: hot`
2. 之前的数据放在另外的 M 台机器上。这 M 台冷数据节点中配置 `node.tag: stale`
3. 模板中控制对新建索引添加 hot 标签：
```
{
    "order" : 0,
    "template" : "*",
    "settings" : {
      "index.routing.allocation.require.tag" : "hot"
    }
}
```
4. 每天计划任务更新索引的配置, 将 tag 更改为 stale, 索引会自动迁移到 M 台冷数据节点
```
# curl -XPUT http://127.0.0.1:9200/indexname/_settings -d'
{
   "index": {
      "routing": {
         "allocation": {
            "require": {
               "tag": "stale"
            }
         }
     }
   }
}'
```

这样，写操作集中在 N 台热数据节点上，大范围的读操作集中在 M 台冷数据节点上。避免了堵塞影响。

该方案运用的，是 Elasticsearch 中的 allocation filter 功能，详细说明见：<https://www.elastic.co/guide/en/elasticsearch/reference/master/shard-allocation-filtering.html>
