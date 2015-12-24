# 集群自动发现

ES 是一个 P2P 类型(使用 gossip 协议)的分布式系统，除了集群状态管理以外，其他所有的请求都可以发送到集群内任意一台节点上，这个节点可以自己找到需要转发给哪些节点，并且直接跟这些节点通信。

所以，从网络架构及服务配置上来说，构建集群所需要的配置极其简单。在无阻碍的网络下，所有配置了相同 `cluster.name` 的节点都自动归属到一个集群中。

## multicast 方式

只配置 `cluster.name` 的集群，其实就是采用了默认的自动发现协议，即组播(multicast)方式。节点会在本机所有网卡接口上，使用组播地址 224.2.2.4 ，以 54328 端口建立组播组发送 clustername 信息。

但是，并不是所有的路由交换设备都支持并且开启了组播信息传输！甚至可以说，默认情况下，都是不开启组播信息传输的。

所以在没有网络工程师帮助的情况下，ES 以默认组播方式，只有在同一个交换机下的节点，能自动发现，跨交换机的节点，是无法收到组播信息的。

此外，由于节点是以所有网卡接口发送组播信息，而操作系统内核层面对组播信息来源的验证中，却对网卡接口地址有一步校验，有可能发生内核层面的信息丢弃，导致多网卡的节点也无法正常使用组播方式。

Elasticsearch 2.0 开始，为安全考虑，默认不分发 multicast 功能。依然希望使用 multicast 自动发现的用户，需要单独安装:

    bin/plugin install discovery-multicast

## unicast 方式

除了组播方式，ES 还支持单播(unicast)方式。配置里提供几台节点的地址，ES 将其视作 gossip router 角色，借以完成集群的发现。由于这只是 ES 内一个很小的功能，所以 gossip router 角色并不需要单独配置，每个 ES 节点都可以担任。所以，采用单播方式的集群，各节点都配置相同的几个节点列表作为 router 即可。

此外，考虑到节点有时候因为高负载，慢 GC 等原因可能会有偶尔没及时响应 ping 包的可能，一般建议稍微加大 Fault Detection 的超时时间。

```
discovery.zen.minimum_master_nodes: 3
discovery.zen.ping.timeout: 100s
discovery.zen.fd.ping_timeout: 100s
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["10.19.0.97","10.19.0.98","10.19.0.99","10.19.0.100"]
```
