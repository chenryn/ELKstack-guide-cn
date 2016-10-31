# gateway

gateway 是 ES 设计用来长期存储索引数据的接口。一般来说，大家都是用本地磁盘来存储索引数据，即 `gateway.type` 为 `local`。

数据恢复中，有很多策略调整我们已经在之前分片控制小节讲过。除开分片级别的控制以外，gateway 级别也还有一些可优化的地方：

* gateway.recover_after_nodes
  该参数控制集群在达到多少个节点的规模后，才开始数据恢复任务。这样可以避免集群自动发现的初期，分片不全的问题。

* gateway.recover_after_time
  该参数控制集群在达到上条配置设置的节点规模后，再等待多久才开始数据恢复任务。

* gateway.expected_nodes
  该参数设置集群的预期节点总数。在达到这个总数后，即认为集群节点已经完全加载，即可开始数据恢复，不用再等待上条设置的时间。

注意：gateway 中说的节点，仅包括主节点和数据节点，纯粹的 client 节点是不算在内的。如果你有更明确的选择，也可以按需求写：

* gateway.recover_after_data_nodes
* gateway.recover_after_master_nodes
* gateway.expected_data_nodes
* gateway.expected_master_nodes

## 共享存储上的影子副本

虽然 ES 对 gateway 使用 NFS，iscsi 等共享存储的方式极力反对，但是对于较大量级的索引的副本数据，ES 从 1.5 版本开始，还是提供了一种节约成本又不特别影响性能的方式：影子副本(shadow replica)。

首先，需要在集群各节点的 `elasticsearch.yml` 中开启选项：

```
node.enable_custom_paths: true
```

同时，确保各节点使用相同的路径挂载了共享存储，且目录权限为 Elasticsearch 进程用户可读可写。

然后，创建索引：

```
# curl -XPUT 'http://127.0.0.1:9200/my_index' -d '
{
    "index" : {
        "number_of_shards" : 1,
        "number_of_replicas" : 4,
        "data_path": "/var/data/my_index",
        "shadow_replicas": true
    }
}'
```

针对 shadow replicas ，ES 节点不会做实际的索引操作，而是单纯的每次 flush 时，把 segment 内容 fsync 到共享存储磁盘上。然后 refresh 让其他节点能够搜索该 segment 内容。

如果你已经决定把数据放到共享存储上了，采用 shadow replicas 还是有一些好处的：

1. 可以帮助你节省一部分不必要的多副本分片的数据写入压力；
2. 在节点出现异常，需要在其他节点上恢复副本数据的时候，可以避免不必要的网络数据拷贝。

但是请注意：主分片节点还是要承担一个副本的写入过程，并不像 Lucene 的 FileReplicator 那样通过复制文件完成，所以达不到完全节省 CPU 的效果。

shadow replicas 只是一个在某些特定环境下有用的方式。在资源允许的情况下，还是应该使用 local gateway。而另外采用 snapshot 接口来完成数据长期备份到 HDFS 或其他共享存储的需要。
