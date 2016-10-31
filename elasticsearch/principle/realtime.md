# segment、buffer和translog对实时性的影响

既然介绍数据流向，首先第一步就是：写入的数据是如何变成 Elasticsearch 里可以被检索和聚合的索引内容的？

以单文件的静态层面看，每个全文索引都是一个词元的倒排索引，具体涉及到全文索引的通用知识，这里不单独介绍，有兴趣的读者可以阅读《Lucene in Action》等书籍详细了解。

## 动态更新的 Lucene 索引

以在线动态服务的层面看，要做到实时更新条件下数据的可用和可靠，就需要在倒排索引的基础上，再做一系列更高级的处理。

其实总结一下 Lucene 的处理办法，很简单，就是一句话：**新收到的数据写到新的索引文件里**。

Lucene 把每次生成的倒排索引，叫做一个段(segment)。然后另外使用一个 commit 文件，记录索引内所有的 segment。而生成 segment 的数据来源，则是内存中的 buffer。也就是说，动态更新过程如下：

1. 当前索引有 3 个 segment 可用。索引状态如图 2-1；
![A Lucene index with a commit point and three segments](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1101.png)
图 2-1

2. 新接收的数据进入内存 buffer。索引状态如图 2-2；
![A Lucene index with new documents in the in-memory buffer, ready to commit](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1102.png)
图 2-2

3. 内存 buffer 刷到磁盘，生成一个新的 segment，commit 文件同步更新。索引状态如图 2-3。
![After a commit, a new segment is added to the index and the buffer is cleared](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1103.png)
图 2-3

## 利用磁盘缓存实现的准实时检索

既然涉及到磁盘，那么一个不可避免的问题就来了：磁盘太慢了！对我们要求实时性很高的服务来说，这种处理还不够。所以，在第 3 步的处理中，还有一个中间状态：

3. 内存 buffer 生成一个新的 segment，刷到文件系统缓存中，Lucene 即可检索这个新 segment。索引状态如图 2-4。
![The buffer contents have been written to a segment, which is searchable, but is not yet commited](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1105.png)
图 2-4
4. 文件系统缓存真正同步到磁盘上，commit 文件更新。达到图 2-3 中的状态。

这一步刷到文件系统缓存的步骤，在 Elasticsearch 中，是默认设置为 1 秒间隔的，对于大多数应用来说，几乎就相当于是实时可搜索了。Elasticsearch 也提供了单独的 `/_refresh` 接口，用户如果对 1 秒间隔还不满意的，可以主动调用该接口来保证搜索可见。

*注：5.0 中还提供了一个新的请求参数：`?refresh=wait_for`，可以在写入数据后不强制刷新但一直等到刷新才返回。*

不过对于 Elastic Stack 的日志场景来说，恰恰相反，我们并不需要如此高的实时性，而是需要更快的写入性能。所以，一般来说，我们反而会通过 `/_settings` 接口或者定制 template 的方式，加大 `refresh_interval` 参数：

```
# curl -XPOST http://127.0.0.1:9200/logstash-2015.06.21/_settings -d'
{ "refresh_interval": "10s" }
'
```

如果是导入历史数据的场合，那甚至可以先完全关闭掉：

```
# curl -XPUT http://127.0.0.1:9200/logstash-2015.05.01 -d'
{
  "settings" : {
    "refresh_interval": "-1"
  }
}'
```

在导入完成以后，修改回来或者手动调用一次即可：

```
# curl -XPOST http://127.0.0.1:9200/logstash-2015.05.01/_refresh
```

## translog 提供的磁盘同步控制

既然 refresh 只是写到文件系统缓存，那么第 4 步写到实际磁盘又是有什么来控制的？如果这期间发生主机错误、硬件故障等异常情况，数据会不会丢失？

这里，其实有另一个机制来控制。Elasticsearch 在把数据写入到内存 buffer 的同时，其实还另外记录了一个 translog 日志。也就是说，第 2 步并不是图 2-2 的状态，而是像图 2-5 这样：

![New documents are added to the in-memory buffer and appended to the transaction log](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1106.png)
图 2-5

在第 3 和第 4 步，refresh 发生的时候，translog 日志文件依然保持原样，如图 2-6：

![The transaction log keeps accumulating documents](https://www.elastic.co/guide/en/elasticsearch/guide/current/images/elas_1108.png)
图 2-6

也就是说，如果在这期间发生异常，Elasticsearch 会从 commit 位置开始，恢复整个 translog 文件中的记录，保证数据一致性。

等到真正把 segment 刷到磁盘，且 commit 文件进行更新的时候， translog 文件才清空。这一步，叫做 flush。同样，Elasticsearch 也提供了 `/_flush` 接口。

对于 flush 操作，Elasticsearch 默认设置为：每 30 分钟主动进行一次 flush，或者当 translog 文件大小大于 512MB (老版本是 200MB)时，主动进行一次 flush。这两个行为，可以分别通过 `index.translog.flush_threshold_period` 和 `index.translog.flush_threshold_size` 参数修改。

如果对这两种控制方式都不满意，Elasticsearch 还可以通过 `index.translog.flush_threshold_ops` 参数，控制每收到多少条数据后 flush 一次。

### translog 的一致性

索引数据的一致性通过 translog 保证。那么 translog 文件自己呢？

默认情况下，Elasticsearch 每 5 秒，或每次请求操作结束前，会强制刷新 translog 日志到磁盘上。

后者是 Elasticsearch 2.0 新加入的特性。为了保证不丢数据，每次 index、bulk、delete、update 完成的时候，一定触发刷新 translog 到磁盘上，才给请求返回 200 OK。这个改变在提高数据安全性的同时当然也降低了一点性能。

如果你不在意这点可能性，还是希望性能优先，可以在 index template 里设置如下参数：

```json
{
    "index.translog.durability": "async"
}
```

## Elasticsearch 分布式索引

大家可能注意到了，前面一段内容，一直写的是"Lucene 索引"。这个区别在于，Elasticsearch 为了完成分布式系统，对一些名词概念作了变动。*索引*成为了整个集群级别的命名，而在单个主机上的*Lucene 索引*，则被命名为*分片(shard)*。至于数据是怎么识别到自己应该在哪个分片，请阅读稍后有关 routing 的章节。

