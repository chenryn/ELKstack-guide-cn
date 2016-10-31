# 集群自动发现

ES 是一个 P2P 类型(使用 gossip 协议)的分布式系统，除了集群状态管理以外，其他所有的请求都可以发送到集群内任意一台节点上，这个节点可以自己找到需要转发给哪些节点，并且直接跟这些节点通信。

所以，从网络架构及服务配置上来说，构建集群所需要的配置极其简单。在 Elasticsearch 2.0 之前，无阻碍的网络下，所有配置了相同 `cluster.name` 的节点都自动归属到一个集群中。

2.0 版本之后，基于安全的考虑，Elasticsearch 稍作了调整，避免开发环境过于随便造成的麻烦。

## unicast 方式

ES 从 2.0 版本开始，默认的自动发现方式改为了单播(unicast)方式。配置里提供几台节点的地址，ES 将其视作 gossip router 角色，借以完成集群的发现。由于这只是 ES 内一个很小的功能，所以 gossip router 角色并不需要单独配置，每个 ES 节点都可以担任。所以，采用单播方式的集群，各节点都配置相同的几个节点列表作为 router 即可。

此外，考虑到节点有时候因为高负载，慢 GC 等原因可能会有偶尔没及时响应 ping 包的可能，一般建议稍微加大 Fault Detection 的超时时间。

同样基于安全考虑做的变更还有监听的主机名。现在默认只监听本地 lo 网卡上。所以正式环境上需要修改配置为监听具体的网卡。

```
network.host: "192.168.0.2" 
discovery.zen.minimum_master_nodes: 3
discovery.zen.ping.timeout: 100s
discovery.zen.fd.ping_timeout: 100s
discovery.zen.ping.unicast.hosts: ["10.19.0.97","10.19.0.98","10.19.0.99","10.19.0.100"]
```

上面的配置中，两个 timeout 可能会让人有所迷惑。这里的 **fd** 是 fault detection 的缩写。也就是说：

* discovery.zen.ping.timeout 参数仅在加入或者选举 master 主节点的时候才起作用；
* discovery.zen.fd.ping\_timeout 参数则在稳定运行的集群中，master 检测所有节点，以及节点检测 master 是否畅通时长期有用。

既然是长期有用，自然还有运行间隔和重试的配置，也可以根据实际情况调整：

```
discovery.zen.fd.ping_interval: 10s
discovery.zen.fd.ping_retries: 10
```
