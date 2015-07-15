# shard 的 allocate 控制

某个 shard 分配在哪个节点上，一般来说，是由 ES 自动决定的。以下几种情况会触发分配动作：

1. 新索引生成
2. 索引的删除
3. 新增副本分片
4. 节点增减引发的数据均衡


ES 提供了一系列参数详细控制这部分逻辑：

* cluster.routing.allocation.enable
  该参数用来控制允许分配哪种分片。默认是 `all`。可选项还包括 `primaries` 和 `new_primaries`。`none` 则彻底拒绝分片。该参数的作用，本书稍后集群升级章节会有说明。
* cluster.routing.allocation.allow_rebalance
  该参数用来控制什么时候允许数据均衡。默认是 `indices_all_active`，即要求所有分片都正常启动成功以后，才可以进行数据均衡操作，否则的话，在集群重启阶段，会浪费太多流量了。
* cluster.routing.allocation.cluster_concurrent_rebalance
  该参数用来控制**集群内**同时运行的数据均衡任务个数。默认是 2 个。如果有节点增减，且集群负载压力不高的时候，可以适当加大。
* cluster.routing.allocation.node_initial_primaries_recoveries
  该参数用来控制**节点**重启时，允许同时恢复几个主分片。默认是 4 个。如果节点是多磁盘，且 IO 压力不大，可以适当加大。
* cluster.routing.allocation.node_concurrent_recoveries
  该参数用来控制**节点**除了主分片重启恢复以外其他情况下，允许同时运行的数据恢复任务。默认是 2 个。所以，节点重启时，可以看到主分片迅速恢复完成，副本分片的恢复却很慢。除了副本分片本身数据要通过网络复制以外，并发线程本身也减少了一半。当然，这种设置也是有道理的——主分片一定是本地恢复，副本分片却需要走网络，带宽是有限的。从 ES 1.6 开始，冷索引的副本分片可以本地恢复，这个参数也就是可以适当加大了。
* indices.recovery.concurrent_streams
  该参数用来控制**节点**从网络复制恢复副本分片时的数据流个数。默认是 3 个。可以配合上一条配置一起加大。
* indices.recovery.max_bytes_per_sec
  该参数用来控制**节点**恢复时的速率。默认是 20MB。显然是比较小的，建议加大。

此外，ES 还有一些其他的分片分配控制策略。比如以 `tag` 和 `rack_id` 作为区分等。一般来说，ELKstack 场景中使用不多。运维人员可能比较常见的策略有两种：

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
   默认情况下，ES 集群的数据均衡策略是以各节点的分片总数(*indices_all_active*)作为基准的。这对于搜索服务来说无疑是均衡搜索压力提高性能的好办法。但是对于 ELKstack 场景，一般压力集中在新索引的数据写入方面。正常运行的时候，也没有问题。但是当集群扩容时，新加入集群的节点，分片总数远远低于其他节点。这时候如果有新索引创建，ES 的默认策略会导致新索引的所有主分片几乎全分配在这台新节点上。整个集群的写入压力，压在一个节点上，结果很可能是这个节点直接被压死，集群出现异常。
   所以，对于 ELKstack 场景，强烈建议大家预先计算好索引的分片数后，配置好单节点分片的限额。比如，一个 5 节点的集群，索引主分片 10 个，副本 1 份。则平均下来每个节点应该有 4 个分片，那么就配置：

	```
	# curl -s -XPUT http://127.0.0.1:9200/logstash-2015.05.08/_settings -d '{
	    "index": { "routing.allocation.total_shards_per_node" : "5" }
	}'
	```
	
	注意，这里配置的是 5 而不是 4。因为我们需要预防有机器故障，分片发生迁移的情况。如果写的是 4，那么分片迁移会失败。

3. 冷热数据分离(读写分离)
	如果没有读写分离, 一个比较突出的问题是: 用户做一次大的查询的时候, 非常大量的读IO以及聚合计算导致机器Load升高, CPU使用率上升, 会阻塞新数据的写入, 这个过程甚至会持续几分钟.
	
	**读写分离的实施方案:**
	- N台机器做热数据的存储, 上面只放当天的数据.  之前的数据放在另外的M台机器上.
	- N台热数据节点上面的elasticsearc.yml中配置node.tag: hot, M台冷数据节点中配置node.tag: stale
	- 模板中控制对新建索引添加hot tag
	```
		 {
	    "order" : 0,
	    "template" : "*",
	    "settings" : {
	      "index.routing.allocation.require.tag" : "hot"
	    }
		}
	```
	- 每天计划任务更新索引的配置, 将tag更改为stale, 索引会自动迁移到M台冷数据节点.
		```
		PUT indexname/_settings
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
		}
	```

## reroute 接口

上面说的各种配置，都是从策略层面，控制分片分配的选择。在必要的时候，还可以通过 ES 的 reroute 接口，手动完成对分片的分配选择的控制。

reroute 接口支持三种指令：allocate，move 和 cancel。常用的一半是 allocate 和 move：

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

## filter 控制

<https://www.elastic.co/guide/en/elasticsearch/reference/master/shard-allocation-filtering.html>
