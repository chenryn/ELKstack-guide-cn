# cat 接口的命令行使用

之前介绍的各种接口数据，其响应数据都是 JSON 格式，更适用于程序处理。对于我们日常运维，在 Linux 命令行终端环境来说，简单的分行和分列表格才是更方便的样式。为此，Elasticsearch 提供了 cat 接口。

cat 接口可以读取各种监控数据，可用接口列表如下：

* /_cat/nodes
* /_cat/shards
* /_cat/shards/{index}
* /_cat/aliases
* /_cat/aliases/{alias}
* /_cat/tasks
* /_cat/master
* /_cat/plugins
* /_cat/fielddata
* /_cat/fielddata/{fields}
* /_cat/pending_tasks
* /_cat/count
* /_cat/count/{index}
* /_cat/snapshots/{repository}
* /_cat/recovery
* /_cat/recovery/{index}
* /_cat/segments
* /_cat/segments/{index}
* /_cat/thread_pool
* /_cat/thread_pool/{thread_pools}/_cat/nodeattrs
* /_cat/allocation
* /_cat/repositories
* /_cat/health
* /_cat/indices
* /_cat/indices/{index}

## 集群状态

还是以最基础的集群状态为例，采用 cat 接口查询集群状态的命令如下：

```
# curl -XGET http://127.0.0.1:9200/_cat/health
1434300299 00:44:59 es1003 red 39 27 2589 1505 4 1 0 0 - 100.0%
```

如果单看这行输出，或许不熟悉的用户会有些茫然。可以通过添加 `?v` 参数，输出表头：

```
# curl -XGET http://127.0.0.1:9200/_cat/health?v
epoch      timestamp cluster status node.total node.data shards  pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1434300334 00:45:34  es1003  green          39        27   2590 1506    5    0        0             0                  -                100.0%
```

## 节点状态

```
# curl -XGET http://127.0.0.1:9200/_cat/nodes?v
host      ip            heap.percent ram.percent load_1m load_5m load_15m node.role master name
esnode109 10.19.0.109             62          69 6.37                     d         -      10.19.0.109 
esnode096 10.19.0.96              63          69 0.29                     -         -      10.19.0.96  
esnode100 10.19.0.100             56          79 0.07                     -         m      10.19.0.100 
```

跟集群状态不一样的是，节点状态数据太多，cat 接口不方便在一行表格中放下所有数据。所以默认的返回，只是最基本的内存和负载数据。具体想看某方面的数据，也是通过请求参数的方式额外指明。比如想看 heap 百分比和最大值：

```
# curl -XGET 'http://127.0.0.1:9200/_cat/nodes?v&h=ip,port,heapPercent,heapMax'
ip            port heapPercent heapMax
192.168.1.131 9300          66 25gb
```

`h` 请求参数可用的值，可以通过 `?help` 请求参数来查询：

```
# curl -XGET http://127.0.0.1:9200/_cat/nodes?help
id               | id,nodeId               | unique node id
host             | h                       | host name
ip               | i                       | ip address
port             | po                      | bound transport port
heap.percent     | hp,heapPercent          | used heap ratio
heap.max         | hm,heapMax              | max configured heap
ram.percent      | rp,ramPercent           | used machine memory ratio
ram.max          | rm,ramMax               | total machine memory
load             | l                       | most recent load avg
node.role        | r,role,dc,nodeRole      | d:data node, c:client node
...
```

中间第二列就是对应的请求参数的值及其缩写。也就是说上面示例还可以写成：

```
# curl -XGET http://127.0.0.1:9200/_cat/nodes?v&h=i,po,hp,hm
```

## 索引状态

查询索引列表和存储的数据状态是也是 cat 接口最常用的功能之一。为了方便阅读，默认输出时会把数据大小以更可读的方式自动换算成合适的单位，比如 *3.2tb* 这样。

如果你打算通过 shell 管道做后续处理，那么可以加上 `?bytes` 参数，指明统一采用字节数输出，这样保证在同一个级别上排序：

```
# curl -XGET http://127.0.0.1:9200/_cat/indices?bytes=b | sort -rnk8
green open logstash-mweibo-2015.06.12        26 1  754641614   0 2534955821580 1256680767317 
green open logstash-mweibo-2015.06.14        27 1  855516794   0 2419569433696 1222355996673 
```

## 分片状态

```
# curl -XGET http://127.0.0.1:9200/_cat/shards?v
index                             shard prirep state            docs    store ip        node
logstash-mweibo-h5view-2015.06.10 20    p      STARTED       4690968  679.2mb 127.0.0.1 10.19.0.108
logstash-mweibo-h5view-2015.06.10 20    r      STARTED       4690968  679.4mb 127.0.0.1 10.19.0.39
logstash-mweibo-h5view-2015.06.10 2     p      STARTED       4725961  684.3mb 127.0.0.1 10.19.0.53
logstash-mweibo-h5view-2015.06.10 2     r      STARTED       4725961  684.3mb 127.0.0.1 10.19.0.102
```

同样，可以用 `?help` 查询其他可用数据细节。比如每个分片的 *segment.count*：

```
# curl -XGET 'http://127.0.0.1:9200/_cat/shards/logstash-mweibo-nginx-2015.06.14?v\&h=n,iic,sc'
n           iic sc
10.19.0.72    0 42
10.19.0.41    0 36
10.19.0.104   0 32
10.19.0.102   0 40
```

## 恢复状态

在出现集群状态波动时，通过这个接口查看数据迁移和恢复速度也是一个非常有用的功能。不过默认输出是把集群历史上所有发生的 recovery 记录都返回出来，所以一般会加上 `?active_only` 参数，仅列出当前还在运行的恢复状态：

```
# curl -XGET 'http://127.0.0.1:9200/_cat/recovery?active_only&v&h=i,s,shost,thost,fp,bp,tr,trp,trt'
i                          s  shost     thost     fp    bp    tr trp    trt
logstash-mweibo-2015.06.12 10 esnode041 esnode080 87.6% 35.3% 0  100.0% 0
logstash-mweibo-2015.06.13 10 esnode108 esnode080 95.5% 88.3% 0  100.0% 0
logstash-mweibo-2015.06.14 17 esnode102 esnode080 96.3% 92.5% 0  0.0%   118758
```

## 线程池状态

```
curl -s -XGET http://127.0.0.1:9200/_cat/thread_pool?v
node_name   name                active queue rejected
esnode073   bulk                     1     0    20669
esnode073   fetch_shard_started      0     0        0
esnode073   fetch_shard_store        0     0        0
esnode073   flush                    0     0        0
esnode073   force_merge              0     0        0
esnode073   generic                  0     0        0
esnode073   get                      0     0        0
esnode073   index                    0     0       50
esnode073   listener                 0     0        0
esnode073   management               1     0        0
esnode073   refresh                  0     0        0
esnode073   search                   4     0        0
esnode073   snapshot                 0     0        0
esnode073   warmer                   0     0        0
```

这个接口的输出形式和 5.0 之前的版本有了较大变化，把不同类型的线程状态做了一次行列转换，大大减少了列数以后，对人眼更加合适了。
