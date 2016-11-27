# profiler

profiler 是 Elasticsearch 5.0 的一个新接口。通过这个功能，可以看到一个搜索聚合请求，是如何拆分成底层的 Lucene 请求，并且显示每部分的耗时情况。

启用 profiler 的方式很简单，直接在请求里加一行即可：

```
curl -XPOST 'http://localhost:9200/_search' -d '{
    "profile": true,
    "query": { ... },
    "aggs": { ... }
}'
```

可以看到其中对 query 和 aggs 部分的返回是不太一样的。

## query

query 部分包括 collectors、rewrite 和 query 部分。对复杂 query，profiler 会拆分 query 成多个基础的 TermQuery，然后每个 TermQuery 再显示各自的分阶段耗时如下：

```
"breakdown": {
    "score": 51306,
    "score_count": 4,
    "build_scorer": 2935582,
    "build_scorer_count": 1,
    "match": 0,
    "match_count": 0,
    "create_weight": 919297,
    "create_weight_count": 1,
    "next_doc": 53876,
    "next_doc_count": 5,
    "advance": 0,
    "advance_count": 0
}
```



## aggs

```
        "time": "1124.864392ms",
        "breakdown": {
            "reduce": 0,
            "reduce_count": 0,
            "build_aggregation": 1394,
            "build_aggregation_count": 150,
            "initialise": 2883,
            "initialize_count": 150,
            "collect": 1124860115,
            "collect_count": 900
        }
```

我们可以很明显的看到聚合统计在初始化阶段、收集阶段、构建阶段、汇总阶段分别花了多少时间，遍历了多少数据。

*注意其中 reduce 阶段还没实现完毕，所有都是 0。因为目前 profiler 只能在 shard 级别上做统计。*

collect 阶段的耗时，有助于我们调整对应 aggs 的 `collect_mode` 参数选择。目前 Elasticsearch 支持 `breadth_first` 和 `depth_first` 两种方式。

initialise 阶段的耗时，有助于我们调整对应 aggs 的 `execution_hint` 参数选择。目前 Elasticsearch 支持 `map`、`global_ordinals_low_cardinality`、`global_ordinals` 和 `global_ordinals_hash` 四种选择。在计算离散度比较大的字段统计值时，适当调整该参数，有益于节省内存和提高计算速度。

*对高离散度字段值统计性能很关注的读者，可以关注 <https://github.com/elastic/elasticsearch/pull/21626> 这条记录的进展。*
