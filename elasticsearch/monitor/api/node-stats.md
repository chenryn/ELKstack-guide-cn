# 节点状态监控接口

集群状态是从最上层高度来评估你的集群概况，而节点状态则更底层一些，会返回给你集群里每个节点的统计信息。这个接口的信息极为丰富，从硬件到数据到线程，应有尽有。本节会以单节点为例，分段介绍各部分数据的含义。

首先，通过如下命令获取节点状态：

```
# curl -XGET 127.0.0.1:9200/_nodes/stats
```

## 节点概要

返回数据的第一部分是节点概要，主要就是节点的主机名，网卡地址和监听端口等。这部分内容除了极少数时候(一个主机上运行了多个 ES 节点)一般没有太大用途。

```
{
   "cluster_name": "elasticsearch_zach",
   "nodes": {
      "UNr6ZMf5Qk-YCPA_L18BOQ": {
         "timestamp": 1477886018477,
         "name": "Zach",
         "transport_address": "192.168.1.131:9300",
         "host": "192.168.1.131",
         "ip": "192.168.1.131:9300",
         "roles": [
             "master",
             "data",
             "ingest"
         ],
...
```

## 索引信息

这部分内容会列出该节点上存储的所有索引数据的状态统计。

1. 首先是概要：

```
    "indices": {
        "docs": {
           "count": 6163666,
           "deleted": 0
        },
        "store": {
           "size_in_bytes": 2301398179,
           "throttle_time_in_millis": 122850
        },
```

*docs.count* 是节点上存储的数据条目总数；*store.size_in_bytes* 是节点上存储的数据占用磁盘的实际大小。而 *store.throttle_time_in_millis* 则是 ES 进程在做 segment merge 时出现磁盘限速的时长。如果你在 ES 的日志里经常会看到限速声明，那么这里的数值也会偏大。

2. 写入性能：

```
        "indexing": {
           "index_total": 803441,
           "index_time_in_millis": 367654,
           "index_current": 99,
           "delete_total": 0,
           "delete_time_in_millis": 0,
           "delete_current": 0
           "noop_update_total" : 0,
           "is_throttled" : false,
           "throttle_time_in_millis" : 0
        },
```

*indexing.index_total* 是一个递增累计数，表示节点完成的数据写入总次数。至于后面又删除了多少，额外记录在 *indexing.delete_total* 里；*indexing.is_throttled* 是 2.0 版之后新增的计数，因为 Elasticsearch 从此开始自动管理 throttle，所以有了这个计数。

3. 读取性能：

```
        "get": {
           "total": 6,
           "time_in_millis": 2,
           "exists_total": 5,
           "exists_time_in_millis": 2,
           "missing_total": 1,
           "missing_time_in_millis": 0,
           "current": 0
        },
```

get 这里显示的是直接使用 `_id` 读取数据的状态。

4. 搜索性能：

```
        "search": {
           "open_contexts": 0,
           "query_total": 123,
           "query_time_in_millis": 531,
           "query_current": 0,
           "fetch_total": 3,
           "fetch_time_in_millis": 55,
           "fetch_current": 0
           "scroll_total" : 0,
           "scroll_time_in_millis" : 0,
           "scroll_current" : 0,
           "suggest_total" : 0,
           "suggest_time_in_millis" : 0,
           "suggest_current" : 0
        },
```

*search.open_contexts* 表示当前正在进行的搜索，而 *search.query_total* 表示节点启动以来完成过的总搜索数，*search.query_time_in_millis* 表示完成上述搜索数花费时间的总和。显然，`query_time_in_millis/query_total` 越大，说明搜索性能越差，可以通过 ES 的 slowlog，获取具体的搜索语句，做出针对性的优化。

*search.fetch_total* 等指标含义类似。因为 ES 的搜索默认是 query-then-fetch 式的，所以 fetch 一般是少而快的。如果计算出来 `search.fetch_time_in_millis > search.query_time_in_millis`，说明有人采用了较大的 *size* 参数做分页查询，通过 slowlog 抓到具体的语句，相机优化成 scan 式的搜索。

5. 段合并性能：

```
        "merges": {
           "current": 0,
           "current_docs": 0,
           "current_size_in_bytes": 0,
           "total": 1128,
           "total_time_in_millis": 21338523,
           "total_docs": 7241313,
           "total_size_in_bytes": 5724869463
           "total_stopped_time_in_millis" : 0,
           "total_throttled_time_in_millis" : 0,
           "total_auto_throttle_in_bytes" : 104857600
        },
```

merges 数据分为两部分，current 开头的是当前正在发生的段合并行为统计；total 开头的是历史总计数。一般来说，作为 Elastic Stack 应用，都是以数据写入压力为主的，merges 相关数据会比较突出。

6. 过滤器缓存：

```
        "query_cache": {
           "memory_size_in_bytes": 48,
           "total_count" : 0,
           "hit_count" : 0,
           "miss_count" : 0,
           "cache_size" : 0,
           "cache_count" : 0,
           "evictions": 0
        },
```

*query_cache.memory_size_in_bytes* 表示过滤器缓存使用的内存，*query_cache.evictions* 表示因内存满被回收的缓存大小，这个数如果较大，说明你的过滤器缓存大小不足，或者过滤器本身不太适合缓存。比如在 Elastic Stack 场景中常用的时间过滤器，如果使用 `@timestamp:["now-1d" TO "now"]` 这种表达式的话，需要每次计算 now 值，就没法长期缓存。事实上，Kibana 中通过 timepicker 生成的 filtered 请求里，对 `@timestamp` 部分就并不是直接使用 `"now"`，而是在浏览器上计算成毫秒数值，再发送给 ES 的。

请注意，过滤器缓存是建立在 segment 基础上的，在当天新日志的索引中，存在大量的或多或少的 segments。一个已经 5GB 大小的 segment，和一个刚刚 2MB 大小的 segment，发生一次 *query_cache.evictions* 对搜索性能的影响区别是巨大的。但是节点状态中本身这个计数并不能反应这点区别。所以，尽力减少这个数值，但如果搜索本身感觉不慢，那么有几个也无所谓。

7. id 缓存：

```
        "id_cache": {
           "memory_size_in_bytes": 0
        },
```

*id_cache* 是 parent/child mappings 使用的内存。不过在 Elastic Stack 场景中，一般不会用到这个特性，所以此处数据应该一直是 0。

8. fielddata：

```
        "fielddata": {
           "memory_size_in_bytes": 0,
           "evictions": 0
        },
```

此处显示 fielddata 使用的内存大小。fielddata 用来做聚合，排序等工作。

注意：*fielddata.evictions* 应该永远是 0。一旦发现这个数据大于 0，请立刻检查自己的内存配置，fielddata 限制以及请求语句。

9. segments：

```
        "segments": {
           "count": 1,
           "memory_in_bytes": 2042,
           "terms_memory_in_bytes" : 1510,
           "stored_fields_memory_in_bytes" : 312,
           "term_vectors_memory_in_bytes" : 0,
           "norms_memory_in_bytes" : 128,
           "points_memory_in_bytes" : 0,
           "doc_values_memory_in_bytes" : 92,
           "index_writer_memory_in_bytes" : 0,
           "version_map_memory_in_bytes" : 0,
           "fixed_bit_set_memory_in_bytes" : 0,
           "max_unsafe_auto_id_timestamp" : -1,
           "file_sizes" : {  }
        },
```

*segments.count* 表示节点上所有索引的 segment 数目的总和。一般来说，一个索引通常会有 50-150 个 segments。再多就对写入性能有较大影响了(可能 merge 速度跟不上新 segment 出现的速度)。所以，请根据节点上的索引数据正确评估节点 segment 的情况。

*segments.memory_in_bytes* 表示 segments 本身底层数据结构所使用的内存大小。像索引的倒排表，词典，bloom filter(ES1.4以后已经默认关闭) 等，都是要在内存里的。所以过多的 segments 会导致这个数值迅速变大。

## 操作系统和进程信息

操作系统信息主要包括 CPU，Loadavg，Memory 和 Swap 利用率，文件句柄等。这些内容都是常见的监控项，本书不再赘述。

进程，即 JVM 信息，主要在于 GC 相关数据。

### GC

对不了解 JVM 的 GC 的读者，这里先介绍一下 GC(垃圾收集)以及 GC 对 Elasticsearch 的影响。

Java is a garbage-collected language, which means that the programmer does not manually manage memory allocation and deallocation. The programmer simply writes code, and the Java Virtual Machine (JVM) manages the process of allocating memory as needed, and then later cleaning up that memory when no longer needed.
Java 是一个自动垃圾收集的编程语言，启动 JVM 虚拟机的时候，会分配到固定大小的内存块，这个块叫做 heap(堆)。JVM 会把 heap 分成两个组：

* Young
新实例化的对象所分配的空间。这个空间一般来说只有 100MB 到 500MB 大小。Young 空间又分为两个 survivor(幸存)空间。当 Young 空间满，就会发生一次 young gc，还存活的对象，就被移入幸存空间里，已失效的对象则被移除。
* Old
老对象存储的空间。这些对象应该是长期存活而且在较长一段时间内不会变化的内容。这个空间会大很多，在 ES 来说，一节点上可能就有 30GB 内存是这个空间。前面提到的 young gc 中，如果某个对象连续多次幸存下来，就会被移进 Old 空间内。而等到 Old 空间满，就会发生一次 old gc，把失效对象移除。

听起来很美好的样子，但是这些都是有代价的！在 GC 发生的时候，JVM 需要暂停程序运行，以便自己追踪对象图收集全部失效对象。在这期间，其他一切都不会继续运行。请求没有响应，ping 没有应答，分片不会分配……

当然，young gc 一般来说执行极快，没太大影响。但是 old 空间那么大，稍慢一点的 gc 就意味着程序几秒乃至十几秒的不可用，这太危险了。

JVM 本身对 gc 算法一直在努力优化，Elasticsearch 也尽量复用内部对象，复用网络缓冲，然后还提供像 Doc Values 这样的特性。但不管怎么说，gc 性能总是我们需要密切关注的数据，因为它是集群稳定性最大的影响因子。

如果你的 ES 集群监控里发现经常有很耗时的 GC，说明集群负载很重，内存不足。严重情况下，这些 GC 导致节点无法正确响应集群之间的 ping ，可能就直接从集群里退出了。然后数据分片也随之在集群中重新迁移，引发更大的网络和磁盘 IO，正常的写入和搜索也会受到影响。

在节点状态数据中，以下部分就是 JVM 相关的数据：

```
"jvm": {
    "timestamp": 1408556438203,
    "uptime_in_millis": 14457,
    "mem": {
       "heap_used_in_bytes": 457252160,
       "heap_used_percent": 44,
       "heap_committed_in_bytes": 1038876672,
       "heap_max_in_bytes": 1038876672,
       "non_heap_used_in_bytes": 38680680,
       "non_heap_committed_in_bytes": 38993920,
    },
```

首先可以看到的就是 heap 的情况。其中这个 `heap_committed_in_bytes` 指的是实际被进程使用的内存，以 JVM 的特性，这个值应该等于 `heap_max_in_bytes`。`heap_used_percent` 则是一个更直观的阈值数据。当这个数据大于 75% 的时候，ES 就要开始 GC。也就是说，如果你的节点这个数据长期在 75% 以上，说明你的节点内存不足，GC 可能会很慢了。更进一步，如果到 85% 或者 95% 了，估计节点一次 GC 能耗时 10s 以上，甚至可能会发生 OOM 了。

继续看下一段：

```
   "pools": {
      "young": {
         "used_in_bytes": 138467752,
         "max_in_bytes": 279183360,
         "peak_used_in_bytes": 279183360,
         "peak_max_in_bytes": 279183360
      },
      "survivor": {
         "used_in_bytes": 34865152,
         "max_in_bytes": 34865152,
         "peak_used_in_bytes": 34865152,
         "peak_max_in_bytes": 34865152
      },
      "old": {
         "used_in_bytes": 283919256,
         "max_in_bytes": 724828160,
         "peak_used_in_bytes": 283919256,
         "peak_max_in_bytes": 724828160
      }
   }
},
```

这段里面列出了 young, survivor, 和 old GC 区域的情况，不过一般来说用途不大。再看下一段：

```
    "gc": {
       "collectors": {
          "young": {
             "collection_count": 13,
             "collection_time_in_millis": 923
          },
          "old": {
             "collection_count": 0,
             "collection_time_in_millis": 0
          }
       }
    }
```

这里显示的 young 和 old gc 的计数和耗时。young gc 的 count 一般比较大，这是正常情况。old gc 的 count 应该就保持在比较小的状态，包括耗时的 `collection_time_in_millis` 也应该很小。注意这两个计数都是累计的，所以对于一个长期运行的系统，不能拿这个数值直接做报警的判断，应该是取两次节点数据的差值。有了差值之后，再来看耗时的问题，一般来说，一次 young gc 的耗时应该在 1-2 ms，old gc 在 100 ms 的水平。如果这个耗时有量级上的差距，建议打开 slow-GC 日志，具体研究原因。

## 线程池信息

Elasticsearch 内部是保持着几个线程池的，不同的工作由不同的线程池负责。一般来说，每个池子的工作线程数跟你的 CPU 核数一样。之前有传言中的优化配置是加大这方面的配置项，其实没有什么实际帮助 —— 能干活的 CPU 就那么些个数。所以这段状态数据目的不是用作 ES 配置调优，而是作为性能监控，方便优化你的读写请求。

ES 在 index、bulk、search、get、merge 等各种操作都有专门的线程池，大家的统计数据格式都是类似的：

```
  "index": {
     "threads": 1,
     "queue": 0,
     "active": 0,
     "rejected": 0,
     "largest": 1,
     "completed": 1
  }
```

这些数据中，最重要的是 rejected 数据。当线程中中所有的工作线程都在忙，即 active == threads，后续的请求就会暂时放到排队的队列里，即 queue > 0。但是每个线程池的 queue 也是有大小限制的，默认是 100。如果后续请求超过这个大小，意味着 ES 真的接受不过来这个请求了，就会把后续请求 reject 掉。

### Bulk Rejections

如果你确实注意到了上面数据中的 rejected，很可能就是你在发送 bulk 写入的时候碰到 HTTP 状态码 429 的响应报错了。事实上，集群的承载能力是有上限的。如果你集群每秒就能写入 10000 条数据，以其浪费内存多放几条数据在排队，还不如直接拒绝掉。至少可以让你知道到瓶颈了。

另外有一点可以指出的是，因为 bulk queue 里的数据是维护在内存中，所以节点发生意外死机的时候，是会丢失的。

如果你碰到 bulk rejected，可以尝试以下步骤：

1. 暂停所有的写入进程。
2. 从 bulk 响应中过滤出来 rejected 的那部分。因为 bulk index 中可能大部分已经成功了。
3. 重发一次失败的请求。
4. 恢复写入进程，或者重新来一次上述步骤。

大家可能看出来了，没错，对 rejected 其实压根没什么特殊的操作，重试一次而已。

当然，如果这个 rejected 是持续存在并增长的，那重试也无济于事。你可能需要考虑自己集群是否足以支撑当前的写入速度要求。

如果确实没问题，那么可能是因为客户端并发太多，超过集群的 bulk threads 总数了。尝试减少自己的写入进程个数，改成加大每次 bulk 请求的 size。

## 文件系统和网络

数据继续往下走，是文件系统和网络的数据。文件系统方面，不管是剩余空间还是 IO 数据，都推荐大家还是通过更传统的系统层监控手段来做。而网络数据方面，主要有两部分内容：

```
        "transport": {
            "server_open": 13,
            "rx_count": 11696,
            "rx_size_in_bytes": 1525774,
            "tx_count": 10282,
            "tx_size_in_bytes": 1440101928
         },
         "http": {
            "current_open": 4,
            "total_opened": 23
         },
```

我们知道 ES 同时运行着 transport 和 http ，默认分别是 9300 和 9200 端口。由于 ES 使用了一些 transport 连接来维护节点内部关系，所以 `transport.server_open` 正常情况下一直会有一定大小。而 `http.current_open` 则是实际连接上来的 HTTP 客户端的数量，考虑到 HTTP 建联的消耗，强烈建议大家使用 keep-alive 长连接的客户端。

## Circuit Breaker

继续往下，是 circuit breaker 的数据，包括 request、fielddata、in\_flight\_requests 和 parent 四种：

```
         "in_flight_requests": {
            "maximum_size_in_bytes": 623326003,
            "maximum_size": "594.4mb",
            "estimated_size_in_bytes": 0,
            "estimated_size": "0b",
            "overhead": 1.03,
            "tripped": 0
         }
```

in\_flight\_requests 是 5.0 版本新加入的一个控制。在过去版本中，索引速度较慢，而入口流量过大会导致 Client 节点在分发 bulk 流量的时候没有限速而 OOM，现在可以直接对过大的流量返回失败了。

## ingest

最后是 ingest 节点独有的 ingest 状态数据。

```
"ingest" : {
    "total" : {
        "count" : 0,
        "time_in_millis" : 0,
        "current" : 0,
        "failed" : 0
    },
    "pipelines" : {
        "set-something" : {
            "count" : 0,
            "time_in_millis" : 0,
            "current" : 0,
            "failed" : 0
        }
    }
}
```

会列出每个定义好的 pipeline 以及最终总体的 ingest 处理量、当前处理中的数据量和处理耗时等。

## hot_threads 状态

除了 stats 信息以外，`/_nodes/` 下还有另一个监控接口：

```
# curl -XGET 'http://127.0.0.1:9200/_nodes/_local/hot_threads?interval=60s'
```

该接口会返回在 `interval` 时长内，该节点消耗资源最多的前三个线程的堆栈情况。这对于性能调优初期，采集现状数据，极为有用。

默认的采样间隔是 500ms，一般来说，这个时间范围是不太够的，建议至少 60s 以上。

默认的，资源消耗是按照 CPU 来衡量，还可以用 `?type=wait` 或者 `?type=block` 来查看在等待和堵塞状态的当前线程排名。
