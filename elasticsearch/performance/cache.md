# 缓存

ES 内针对不同阶段，设计有不同的缓存。以此提升数据检索时的响应性能。主要包括节点层面的 filter cache 和分片层面的 request cache。下面分别讲述。

## filter cache

ES 的 query DSL 在 2.0 版本之前分为 query 和 filter 两种，很多检索语法，是同时存在 query 和 filter 里的。比如最常用的 term、prefix、range 等。怎么选择是使用 query 还是 filter 成为很多用户头疼的难题。于是从 2.0 版本开始，ES 干脆合并了 filter 统一归为 query。但是具体的检索语法本身，依然有 query 和 filter 上下文的区别。ES 依靠这个上下文判断，来自动决定是否启用 filter cache。

query 跟 filter 上下文的区别，简单来说：

* query 是要相关性评分的，filter 不要；
* query 结果无法缓存，filter 可以。

所以，选择也就出来了：

* 全文搜索、评分排序，使用 query；
* 是非过滤，精确匹配，使用 filter。

不过我们要怎么写，才能让 ES 正确判断呢？看下面这个请求：

```
# curl -XGET http://127.0.0.1:9200/_search -d '
{
    "query": {
        "bool": {
            "must_not": [
                { "match": { "title": "Search" } }
            ],
            "must": [
                { "match": { "content": "Elasticsearch" } }
            ],
            "filter": [
                { "term":  { "status": "published" } },
                { "range": { "publish_date": { "gte": "2015-01-01" } } }
            ]
        }
    }
}'
```

在这个请求中，

1. ES 先看到一个 **query**，那么进入 query 上下文；

需要注意的是，filter cache 是节点层面的缓存设置，每个节点上所有数据在响应请求时，是共用一个缓存空间的。当空间用满，按照 LRU 策略淘汰掉最冷的数据。

可以用 `indices.cache.filter.size` 配置来设置这个缓存空间的大小，默认是 JVM 堆的 10%，也可以设置一个绝对值。注意这是一个静态值，必须在 `elasticsearch.yml` 中提前配置。

## shard request cache

ES 还有另一个分片层面的缓存，叫 shard request cache。5.0 之前的版本中，request cache 的用途并不大，因为 query cache 要起作用，还有几个先决条件：

1. 分片数据不再变动，也就是对当天的索引是无效的(如果 `refresh_interval` 很大，那么在这个间隔内倒也算有效)；
2. 使用了 `"now"` 语法的请求无法被缓存，因为这个是要即时计算的；
3. 缓存的键是请求的整个 JSON 字符串，整个字符串发生任何字节变动，缓存都无效。

以 Elastic Stack 场景来说，Kibana 里几乎所有的请求，都是有 `@timestamp` 作为过滤条件的，而且大多数是以*最近 N 小时/分钟*这样的选项，也就是说，页面每次刷新，发出的请求 JSON 里的时间过滤部分都是在变动的。query cache 在处理 Kibana 发出的请求时，完全无用。

而 5.0 版本的一大特性，叫 instant aggregation。解决了这个先决条件的一大阻碍。

在之前的版本，Elasticsearch 接收到请求之后，直接把请求原样转发给各分片，由各分片所在的节点自行完成请求的解析，进行实际的搜索操作。所以缓存的键是原始 JSON 串。

而 5.0 的重构后，接收到请求的节点先把请求的解析做完，发送到各节点的是统一拆分修改好的请求，这样就不再担心 JSON 串多个空格啥的了。

其次，上面说的『拆分修改』是怎么回事呢？

比如，我们在 Kibana 里搜索一个最近 7 天(`@timestamp:["now-7d" TO "now"]`)的数据，ES 就可以根据按天索引的判断，知道从 6 天前到昨天这 5 个索引是肯定全覆盖的。那么这个横跨 7 天的 `date range` query 就变成了 5 个 `match_all` query 加 2 个短时间的 `date_range` query。

现在你的仪表盘过 5 分钟自动刷新一次，再提交上来一次最近 7 天的请求，中间这 5 个 `match_all` 就完全一样了，直接从 request cache 返回即可，需要重新请求的，只有两头真正在变动的 `date_range` 了。

*注1：`match_all` 不用遍历倒排索引，比直接查询 `@timestamp:*` 要快很多。*
*注2：判断覆盖修改为 `match_all` 并不是真的按照索引名称，而是 ES 从 2.x 开始提供的 `field_stats` 接口可以直接获取到 `@timestamp` 在本索引内的 max/min 值。当然从概念上如此理解也是可以接受的。*

### field_stats 接口

```
curl -XGET "http://localhost:9200/logstash-2016.11.25/_field_stats?fields=timestamp"
```

响应结果如下：

```
{
    "_shards": {
        "total": 1,
        "successful": 1,
        "failed": 0
    },
    "indices": {
        "logstash-2016.11.25": {
            "fields": {
                "timestamp": {
                    "max_doc": 1326564,
                    "doc_count": 564633,
                    "density": 42,
                    "sum_doc_freq": 2258532,
                    "sum_total_term_freq": -1,
                    "min_value": "2008-08-01T16:37:51.513Z",
                    "max_value": "2013-06-02T03:23:11.593Z",
                    "is_searchable": "true",
                    "is_aggregatable": "true"
                }
            }
        }
    }
}
```

和 filter cache 一样，request cache 的大小也是以节点级别控制的，配置项名为 `indices.requests.cache.size`，其默认值为 `1%`。
