# 集群版本升级

Elasticsearch 作为一个新兴项目，版本更新非常快。而且每次版本更新都或多或少带有一些重要的性能优化、稳定性提升等特性。可以说，ES 集群的版本升级，是目前 ES 运维必然要做的一项工作。

按照 ES 官方设计，有 restart upgrade 和 rolling upgrade 两种可选的升级方式。对于 1.0 版本以上的用户，推荐采用 rolling upgreade 方式。

**但是，对于主要负载是数据写入的 Elastic Stack 场景来说，却并不是这样！**

rolling upgrade 的步骤大致如下：

1. 暂停分片分配；
2. 单节点下线升级重启；
3. 开启分片分配；
4. 等待集群状态变绿后继续上述步骤。

实际运行中，步骤 2 的 ES 单节点从 restart 到加入集群，大概要 100s 左右的时间。也就是说，这 100s 内，该节点上的所有分片都是 unassigned 状态。而按照 Elasticsearch 的设计，数据写入需要至少达到 `replica/2+1` 个分片完成才能算完成。也就意味着你所有索引都必须至少有 1 个以上副本分片开启。

但事实上，很多日志场景，由于写入性能上的要求要高于数据可靠性的要求，大家普遍减小了副本数量，甚至直接关掉副本复制。这样一来，整个 rolling upgrade 期间，数据写入就会受到严重影响，完全丧失了 rolling 的必要性。

其次，步骤 3 中的 ES 分片均衡过程中，由于 ES 的副本分片数据都需要从主分片走网络复制重新传输一次，而由于重启，新升级的节点上的分片肯定全是副本分片(除非压根没副本)。在数据量较大的情况下，这个步骤耗时可能是几十分钟甚至以小时计。而且并发和限速上稍微不注意，可能导致分片均衡的带宽直接占满网卡，正常写入也还是受到影响。

所以，对于写入压力较大，数据可靠性要求偏低的实时日志场景，依然建议大家进行主动停机式的 restart upgrade。

restart upgrade 的步骤如下：

1. 首先适当加大集群的数据恢复和分片均衡并发度以及磁盘限速：

```
# curl -XPUT http://127.0.0.1:9200/_cluster/settings -d '{
  "persistent" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "disable_allocation" : "false",
          "cluster_concurrent_rebalance" : "5",
          "node_concurrent_recoveries" : "5",
          "enable" : "all"
        }
      }
    },
    "indices" : {
      "recovery" : {
        "concurrent_streams" : "30",
        "max_bytes_per_sec" : "2gb"
      }
    }
  },
  "transient" : {
    "cluster" : {
      "routing" : {
        "allocation" : {
          "enable" : "all"
        }
      }
    }
  }
}'
```

2. 暂停分片分配：

```
# curl -XPUT http://127.0.0.1:9200/_cluster/settings -d '{
  "transient" : {
    "cluster.routing.allocation.enable" : "none"
  }
}'
```

3. 通过配置管理工具下发新版本软件包。
4. 公告周知后，停止数据写入进程(即 logstash indexer 等)
5. 如果使用 Elasticsearch 1.6 版本以上，可以手动运行一次 synced flush，同步副本分片的 commit id，缩小恢复时的网络传输带宽：

```
# curl -XPOST http://127.0.0.1:9200/_flush/synced
```

6. 全集群统一停止进程，更新软件包，重新启动。
7. 等待各节点都加入到集群以后，恢复分片分配：

```
# curl -XPUT http://127.0.0.1:9200/_cluster/settings -d '{
  "transient" : {
    "cluster.routing.allocation.enable" : "all"
  }
}'
```

由于同时启停，主分片几乎可以同时本地恢复，整个集群从 red 变成 yellow 只需要 2 分钟左右。而后的副本分片，如果有 synced flush，同样本地恢复，否则网络恢复总耗时，视数据大小而定，会明显大于单节点恢复的耗时。

8. 如果有 synced flush，建议等待集群变成 green 状态后，恢复写入；否则在集群变成 yellow 状态之后，即可着手开始恢复数据写入进程。
