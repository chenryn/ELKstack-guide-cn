# 集群健康状态监控

说到 Elasticsearch 集群监控，首先我们肯定是需要一个从总体意义上的概要。不管是多大规模的集群，告诉我正常还是不正常？没错，集群健康状态接口就是用来回答这个问题的，而且这个接口的信息已经出于意料的丰富了。

## 命令示例

```
# curl -XGET 127.0.0.1:9200/_cluster/health?pretty
{
  "cluster_name" : "es1003",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 38,
  "number_of_data_nodes" : 27,
  "active_primary_shards" : 1332,
  "active_shards" : 2381,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "number_of_pending_tasks" : 0
  "delayed_unassigned_shards" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

## 状态信息

输出里最重要的就是 **status** 这行。很多开源的 ES 监控脚本，其实就是拿这行数据做报警判断。status 有三个可能的值：

* green
绿灯，所有分片都正确运行，集群非常健康。
* yellow
黄灯，所有主分片都正确运行，但是有副本分片缺失。这种情况意味着 ES 当前还是正常运行的，但是有一定风险。注意，在 Kibana4 的 server 端启动逻辑中，即使是黄灯状态，Kibana 4 也会拒绝启动，死循环等待集群状态变成绿灯后才能继续运行。
* red
红灯，有主分片缺失。这部分数据完全不可用。而考虑到 ES 在写入端是简单的取余算法，轮到这个分片上的数据也会持续写入报错。

对 Nagios 熟悉的读者，可以直接将这个红黄绿灯对应上 Nagios 体系中的 Critical，Warning，OK 。

## 其他数据解释

* number_of_nodes 集群内的总节点数。
* number_of_data_nodes 集群内的总数据节点数。
* active_primary_shards 集群内所有索引的主分片总数。
* active_shards 集群内所有索引的分片总数。
* relocating_shards 正在迁移中的分片数。
* initializing_shards 正在初始化的分片数。
* unassigned_shards 未分配到具体节点上的分片数。
* delayed_unassigned_shards 延时待分配到具体节点上的分片数。

显然，后面 4 项在正常情况下，一般都应该是 0。但是如果真的出来了长期非 0 的情况，怎么才能知道这些长期 unassign 或者 initialize 的分片影响的是哪个索引呢？本书随后还有有更多接口获取相关信息。不过在集群健康这层，本身就可以得到更详细一点的内容了。

## level 请求参数

接口请求的时候，可以附加一个 level 参数，指定输出信息以 indices 还是 shards 级别显示。当然，一般来说，indices 级别就够了。

```
# curl -XGET http://127.0.0.1:9200/_cluster/health?level=indices
{
   "cluster_name": "es1003",
   "status": "red",
   "timed_out": false,
   "number_of_nodes": 38,
   "number_of_data_nodes": 27,
   "active_primary_shards": 1332,
   "active_shards": 2380,
   "relocating_shards": 0,
   "initializing_shards": 0,
   "unassigned_shards": 1
   "delayed_unassigned_shards" : 0,
   "number_of_in_flight_fetch" : 0,
   "task_max_waiting_in_queue_millis" : 0,
   "active_shards_percent_as_number" : 99.0
   "indices": {
      "logstash-2015.05.31": {
         "status": "green",
         "number_of_shards": 81,
         "number_of_replicas": 0,
         "active_primary_shards": 81,
         "active_shards": 81,
         "relocating_shards": 0,
         "initializing_shards": 0,
         "unassigned_shards": 0
      },
      "logstash-2015.05.30": {
         "status": "red",
         "number_of_shards": 81,
         "number_of_replicas": 0,
         "active_primary_shards": 80,
         "active_shards": 80,
         "relocating_shards": 0,
         "initializing_shards": 0,
         "unassigned_shards": 1
      },
      ...
   }
}
```

这就看到了，是 *logstash-2015.05.30* 索引里，有一个分片一直未能成功分配，导致集群状态异常的。

不过，一般来说，集群健康接口，还是只用来简单监控一下集群状态是否正常。一旦收到异常报警，具体确定 unassign shard 的情况，更推荐使用 kopf 工具在页面查看。
