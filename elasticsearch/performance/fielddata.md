# 字段数据的存储方式

字段数据(fielddata)，在 Lucene 中又叫 uninverted index。我们都知道，搜索引擎会使用倒排索引(inverted index)来映射单词到文档>的 ID 号。而同时，为了提供对文档内容的聚合，Lucene 还可以在运行时将每个字段的单词以字典序排成另一个 uninverted index，可以>大大加速计算性能。

作为一个加速性能的方式，fielddata 当然是被全部加载在内存的时候最为有效。这也是 ES 默认的运行设置。但是相比较集群庞大的数据>量，内存本身是远远不够的。为了解决这个问题，ES 引入了另一个特性，可以对精确索引的字段，指定 fielddata 的存储方式。这个配置>项叫：`doc_values`。

所谓 `doc_values`，其实就是在 ES 将数据写入索引的时候，提前生成好 fielddata 内容，并记录到磁盘上。因为 fielddata 数据是顺序读写的，所以即使在磁盘上，通过文件系统层的缓存，也可以获得相当不错的性能。

注意：因为 `doc_values` 是在数据写入时即生成内容，所以，它只能应用在精准索引的字段上，因为索引进程没法知道后续会有什么分词>器生成的结果。所以，字段设置应该是这样：

```
    "myfieldname": {
        "type":       "string",
        "index":      "not_analyzed",
        "doc_values": true
    }
```
