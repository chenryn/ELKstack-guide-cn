# 缓存

ES 内针对不同阶段，设计有不同的缓存。以此提升数据检索时的响应性能。主要包括节点层面的 filter cache 和索引层面的 query cache。下面分别讲述。

## filter cache

ES 的 query DSL 分为 query 和 filter 两种，很多检索语法，是同时存在 query 和 filter 里的。比如最常用的 term、prefix、range 等。那么，怎么选择是使用 query 还是 filter 呢？

首先，要明白 query 跟 filter 的区别：

* query 是要相关性评分的，filter 不要；
* query 结果无法缓存，filter 可以。

所以，选择也就出来了：

* 全文搜索、评分排序，使用 query；
* 是非过滤，精确匹配，使用 filter。

上面做对比时，说 filter 能用缓存 query 不能，其实是不精确的。默认情况下，并不是所有的 filter 都能用缓存。常用的比如 term、terms、prefix、range、bool 等 filter，其过滤结果明确，也容易设置缓存，ES 就对这几个默认开启了 filter cache 功能。而更复杂的一些比如 geo、script 等 filter，从 fielddata 数据到过滤结果还需要进行一系列计算的，ES 默认是不开启 filter cache 的。而像 and、not、or 这几个关系型 filter，也是不开启的。

如果想要强制开启这些默认没有的 filter cache，需要在请求的 JSON 中带上 `"cache": true` 参数。

检索的时候想尽量使用 filter cache，但是 ES 接口有要求必须传递 query 参数，这时候，有一个特殊的 query，叫 **filtered query**。其作用是在 query 中使用 filter，这样，我们就可以实现如下这样的请求：

```
# curl -XGET http://127.0.0.1:9200/_search -d '
{
  "query": {
    "filtered": {
      "filter": {
        "range": { "@timestamp": { "gte": "now - 1d / d" }}
      }
    }
  }
}'
```

事实上，Kibana3 中，就大量使用了这种 filtered query 语法。

需要注意的是，filter cache 是节点层面的缓存设置，每个节点上所有数据在响应请求时，是共用一个缓存空间的。当空间用满，按照 LRU 策略淘汰掉最冷的数据。

可以用 `indices.cache.filter.size` 配置来设置这个缓存空间的大小，默认是 JVM 堆的 10%，也可以设置一个绝对值。

## shard query cache

ES 还有另一个索引层面的缓存，叫 shard query cache。之前章节中说过，ES 集群的任意节点都可以接受请求，它会自动转发给数据所在的各个节点，等待各节点把各自的结果返回后，完成数据的汇聚处理再返回给客户端。

这里可以把这个过程再细化一下。ES 对请求的处理过程，是有不同类型的，默认的叫 *query_then_fetch*。在这种情况下，各数据节点处理检索请求后，返回的，是只包含文档 id 和相关性分值的结果，这一段处理，叫做 query 阶段；汇聚到这份结果后，按照分值排序，得到一个全集群最终需要的文档 id，再向对应节点发送一次文档获取请求，拿到文档内容，这一段处理，叫做 fetch 阶段。两段都结束后才返回响应。在稍后的 ES 日志记录章节，我们可以看到 ES 对这两个阶段，甚至都有分别的慢查询记录。

此外，还有 *DFS_query_then_fetch* 类型，提高小数据量时的精确度；*query_and_fetch* 类型，在有明确 routing 时可以省略一个数据来回；*count* 类型，再不关心文档内容只需要计数时省略 fetch 阶段，这是 ELK Stack 聚合统计场景最常用的类型；*scan* 类型，批量获取数据省略 query 阶段，在 reindex 时就是使用这种类型。

回到 query cache，这里这个 query，就是处理过程中 query 阶段的意思。各个节点上的数据分片，会在处理完 query 阶段时，将得到的本分片有关该请求的计数值，缓存起来。

根据上面的请求类型介绍，显然，只有当 `?search_type=count` 的时候，这个 query cache 才能起到作用。

不过，query cache 默认并不开启。因为 query cache 要起作用，还有几个先决条件：

1. 分片数据不再变动，也就是对当天的索引是无效的(如果 `refresh_interval` 很大，那么在这个间隔内倒也算有效)；
2. 使用了 `"now"` 语法的请求无法被缓存，因为这个是要即时计算的；
3. 缓存的键是请求的整个 JSON 字符串，整个字符串发生任何字节变动，缓存都无效。

以 ELK Stack 场景来说，Kibana 里几乎所有的请求，都是有 `@timestamp` 作为过滤条件的，而且大多数是以*最近 N 小时/分钟*这样的选项，也就是说，页面每次刷新，发出的请求 JSON 里的时间过滤部分都是在变动的。query cache 在处理 Kibana 发出的请求时，完全无用。

所以，虽然官网宣称 query cache 对日志场景非常有用，但是对使用 Kibana 的一般用户来说，完全没法体会到这个优势。

如果是自己写程序做历史统计分析和展示的，想办法固化时间过滤条件或者干脆去掉这个条件，那么用上这个特性还是不错的。要对某个索引开启 query cache 的话，利用 index settings 接口动态调整即可：

```
# curl -XPUT http://127.0.0.1:9200/logstash-2015.06.23/_settings -d'
{
    "index.cache.query.enable": true 
}'
```

此外，还可以针对具体某个请求单独开启：

```
# curl 'http://127.0.0.1:9200/logstash-2015.06.23/_search?search_type=count&query_cache=true' -d'
{
  "aggs": {
    "popular_colors": {
      "terms": {
        "field": "colors"
      }
    }
  }
}'
```

和 filter cache 一样，query cache 的大小也是以节点级别控制的，配置项名为 `indices.cache.query.size`，其默认值为 `1%`。
