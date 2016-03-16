# 字段数据

**Elasticsearch 2.x 已经默认所有 not\_analyzed 字段自动启用 doc\_values ！本节不再重要，仅作为知识共享保留。**

字段数据(fielddata)，在 Lucene 中又叫 uninverted index。我们都知道，搜索引擎会使用倒排索引(inverted index)来映射单词到文档的 ID 号。而同时，为了提供对文档内容的聚合，Lucene 还可以在运行时将每个字段的单词以字典序排成另一个 uninverted index，可以大大加速计算性能。

作为一个加速性能的方式，fielddata 当然是被全部加载在内存的时候最为有效。这也是 ES 默认的运行设置。但是，内存是有限的，所以 ES 同时也需要提供对 fielddata 内存的限额方式：

* indices.fielddata.cache.size
  节点用于 fielddata 的最大内存，如果 fielddata 达到该阈值，就会把旧数据交换出去。该参数可以设置百分比或者绝对值。默认设置是不限制，所以强烈建议设置该值，比如 `10%`。
* indices.fielddata.cache.expire
  进入 fielddata 内存中的数据多久自动过期。注意，因为 ES 的 fielddata 本身是一种数据结构，而不是简单的缓存，所以过期删除 fielddata 是一个非常消耗资源的操作。ES 官方在文档中特意说明，这个参数绝对绝对**不要**设置！

### Circuit Breaker

Elasticsearch 在 total，fielddata，request 三个层面上都设计有 circuit breaker 以保护进程不至于发生 OOM 事件。在 fielddata 层面，其设置为：

* indices.breaker.fielddata.limit
  默认是 JVM 堆内存大小的 60%。注意，为了让设置正常发挥作用，如果之前设置过 `indices.fielddata.cache.size` 的，一定要确保 `indices.breaker.fielddata.limit` 的值大于 `indices.fielddata.cache.size` 的值。否则的话，fielddata 大小一到 limit 阈值就报错，就永远道不了 size 阈值，无法触发对旧数据的交换任务了。

## doc values

但是相比较集群庞大的数据量，内存本身是远远不够的。为了解决这个问题，ES 引入了另一个特性，可以对精确索引的字段，指定 fielddata 的存储方式。这个配置项叫：`doc_values`。

所谓 `doc_values`，其实就是在 ES 将数据写入索引的时候，提前生成好 fielddata 内容，并记录到磁盘上。因为 fielddata 数据是顺序读写的，所以即使在磁盘上，通过文件系统层的缓存，也可以获得相当不错的性能。

注意：因为 `doc_values` 是在数据写入时即生成内容，所以，它只能应用在精准索引的字段上，因为索引进程没法知道后续会有什么分词器生成的结果。所以，字段设置应该是这样：

```
    "myfieldname": {
        "type":       "string",
        "index":      "not_analyzed",
        "doc_values": true
    }
```

