# 批量提交

在 CRUD 章节，我们已经知道 ES 的数据写入是如何操作的了。喜欢自己动手的读者可能已经迫不及待的自己写了程序开始往 ES 里写数据做测试。这时候大家会发现：程序的运行速度非常一般，即使 ES 服务运行在本机，一秒钟大概也就能写入几百条数据。

这种速度显然不是 ES 的极限。事实上，每条数据经过一次完整的 HTTP POST 请求和 ES indexing 是一种极大的性能浪费，为此，ES 设计了批量提交方式。在数据读取方面，叫 mget 接口，在数据变更方面，叫 bulk 接口。mget 一般常用于搜索时 ES 节点之间批量获取中间结果集，对于 ELK Stack 用户，更常见到的是 bulk 接口。

bulk 接口采用一种比较简朴的数据积累格式，示例如下：

```
# curl -XPOST http://127.0.0.1:9200/_bulk -d'
{ "create" : { "_index" : "test", "_type" : "type1"  } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "type1" } }
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value2" }
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
'
```

格式是，每条 JSON 数据的上面，加一行描述性的元 JSON，指明下一行数据的操作类型，归属索引信息等。

采用这种格式，而不是一般的 JSON 数组格式，是因为接收到 bulk 请求的 ES 节点，就可以不需要做完整的 JSON 数组解析处理，直接按行处理简短的元 JSON，就可以确定下一行数据 JSON 转发给哪个数据节点了。这样，一个固定内存大小的 network buffer 空间，就可以反复使用，又节省了大量 JVM 的 GC。

事实上，产品级的 logstash、rsyslog、spark 都是默认采用 bulk 接口进行数据写入的。对于打算自己写程序的读者，建议采用 Perl 的 `Search::Elasticsearch::Bulk` 或者 Python 的 `elasticsearch.helpers.*` 库。

## bulk size

在配置 bulk 数据的时候，一般需要注意的就是请求体大小(bulk size)。

这里有一点细节上的矛盾，我们知道，HTTP 请求，是可以通过 HTTP 状态码 *100 Continue* 来持续发送数据的。但对于 ES 节点接收 HTTP 请求体的 *Content-Length* 来说，是按照整个大小来计算的。所以，首先，要确保 bulk 数据不要超过 `http.max_content_length` 设置。

那么，是不是尽量让 bulk size 接近这个数值呢？当然不是。

依然是请求体的问题，因为请求体需要全部加载到内存，而 JVM Heap 一共就那么多(按 31GB 算)，过大的请求体，会挤占其他线程池的空间，反而导致写入性能的下降。

再考虑网卡流量，磁盘转速的问题，所以一般来说，建议 bulk 请求体的大小，在 15MB 左右，通过实际测试继续向上探索最合适的设置。

注意：这里说的 15MB 是请求体的字节数，而不是程序里里设置的 bulk size。bulk size 一般指数据的条目数。不要忘了，bulk 请求体中，每条数据还会额外带上一行元 JSON。

以 logstash 默认的 `bulk_size => 5000` 为例，假设单条数据平均大小 200B ，一次 bulk 请求体的大小就是 1.5MB。那么我们可以尝试 `bulk_size => 50000`；而如果单条数据平均大小是 20KB，一次 bulk 大小就是 100MB，显然超标了，需要尝试下调至 `bulk_size => 500`。

## UDP

ES 其实还提供了一个连 HTTP header 解析步骤都能省略的 bulk 方法，叫 UDP bulk，即开启 UDP 9700 端口，直接 nc 发送 bulk 数据内容写入。

由于 UDP 的不可靠性，ES 计划从 2.0 版本开始废弃该功能，确实需要高性能写入又不担心数据缺失问题的读者，可以参考 ES 官方文档使用该功能。
