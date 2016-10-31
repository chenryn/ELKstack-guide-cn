# 集群状态维护

我们都知道，ES 中的 master 跟一般 MySQL、Hadoop 的 master 是不一样的。它即不是写入流量的唯一入口，也不是所有数据的元信息的存放地点。所以，一般来说，ES 的 master 节点负载很轻，集群性能是可以近似认为随着 data 节点的扩展线性提升的。

但是，上面这句话并不是完全正确的。

ES 中有一件事情是只有 master 节点能管理的，这就是集群状态(cluster state)。

集群状态中包括以下信息：

* 集群层面的设置
* 集群内有哪些节点
* 各索引的设置，映射，分析器和别名等
* 索引内各分片所在的节点位置

这些信息在集群的任意节点上都存放着，你也可以通过 `/_cluster/state` 接口直接读取到其内容。注意这最后一项信息，之前我们已经讲过 ES 怎么通过简单地取余知道一条数据放在哪个分片里，加上现在集群状态里又记载了分片在哪个节点上，那么，整个集群里，任意节点都可以知道一条数据在哪个节点上存储了。所以，数据读写才可以发送给集群里任意节点。

至于修改，则只能由 master 节点完成！显然，集群状态里大部分内容是极少变动的，唯独有一样除外——索引的映射。因为 ES 的 schema-less 特性，我们可以任意写入 JSON 数据，所以索引中随时可能增加新的字段。这个时候，负责容纳这条数据的主分片所在的节点，会暂停写入操作，将字段的映射结果传递给 master 节点；master 节点合并这段修改到集群状态里，发送新版本的集群状态到集群的所有节点上。然后写入操作才会继续。一般来说，这个操作是在一二十毫秒内就可以完成，影响也不大。

但是也有一些情况会是例外。

## 批量新索引创建

在较大规模的 Elastic Stack 应用场景中，这是比较常见的一个情况。因为 Elastic Stack 建议采用日期时间作为索引的划分方式，所以定时(一般是每天)，会统一产生一批新的索引。而前面已经讲过，ES 的集群状态每次更新都是阻塞式的发布到全部节点上以后，节点才能继续后续处理。

这就意味着，如果在集群负载较高的时候，批量新建新索引，可能会有一个显著的阻塞时间，无法写入任何数据。要等到全部节点同步完成集群状态以后，数据写入才能恢复。

不巧的是，中国使用的是北京时间，UTC +0800。也就是说，默认的 Elastic Stack 新建索引时间是在早上 8 点。这个时间点一般日志写入量已经上涨到一定水平了(当然，晚上 0 点的量其实也不低)。

对此，可以通过定时任务，每天在最低谷的早上三四点，提前通过 POST mapping 的方式，创建好之后几天的索引。就可以避免这个问题了。

**如果你的日志是比较严重的非结构化数据，这个问题在 2.0 版本后会变得更加严重。** Elasticsearch 从 2.0 版本开始，对 mapping 更新做了重构。为了防止字段类型冲突和减少 master 定期下发全量 cluster state 导致的大流量压力，新的实现和旧实现的区别在：

* 过去：每次 bulk 请求，本地生成索引后，将更新的 mapping，按照 `_type` 为单位构成 mapping 更新请求发给 master；
* 现在：每次 bulk 请求，遍历每条数据，将每条数据要更新的 mapping，都单独发给 master，等到 master 通知完全集群，本地才能生成这一条数据的索引。

也就是说，一旦你日志中字段数量较多，在新创建索引的一段时间内，可能长达几十分钟一直被反复锁死！

## 过多字段持续更新

这是另一种常见的滥用。在使用 Elastic Stack 处理访问日志时，为了查询更方便，可能会采用 logstash-filter-kv 插件，将访问日志中的每个 URL 参数，都切分成单独的字段。比如一个 "/index.do?uid=1234567890&action=payload" 的 URL 会被转换成如下 JSON：

```
  "urlpath" : "/index.do",
  "urlargs" : {
    "uid" : "1234567890",
    "action" : "payload",
    ...
  }
```

但是，因为集群状态是存在所有节点的内存里的，一旦 URL 参数过多，ES 节点的内存就被大量用于存储字段映射内容。这是一个极大的浪费。如果碰上 URL 参数的键内容本身一直在变动，直接撑爆 ES 内存都是有可能的！

*以上是真实发生的事件，开发人员莫名的选择将一个 UUID 结果作为 key 放在 URL 参数里。直接导致 ES 集群 master 节点全部 OOM。*

如果你在 ES 日志中一直看到有新的 `updating mapping [logstash-2015.06.01]` 字样出现的话，请郑重考虑一下自己是不是用的上如此细分的字段列表吧。

好，三秒钟过去，如果你确定一定以及肯定还要这么做，下面是一个变通的解决办法。

### nested object

用 nested object 来存放 URL 参数的方法稍微复杂，但还可以接受。单从 JSON 数据层面看，新方式的数据结构如下：

```
  "urlargs": [
    { "key": "uid", "value": "1234567890" },
    { "key": "action", "value": "payload" },
    ...
  ]
```

没错，看起来就是一个数组。但是 JSON 数组在 ES 里是有两种处理方式的。

如果直接写入数组，ES 在实际索引过程中，会把所有内容都平铺开，变成 **Arrays of Inner Objects**。整条数据实际类似这样的结构：

```
{
  "urlpath" : ["/index.do"],
  "urlargs.key" : ["uid", "action", ...],
  "urlargs.value" : ["1234567890", "payload", ...]
```

这种方式最大的问题是，当你采用 `urlargs.key:"uid" AND urlargs.value:"0987654321"` 语句意图搜索一个 uid=0987654321 的请求时，实际是整个 URL 参数中任意一处 value 为 0987654321 的，都会命中。

要想达到正确搜索的目的，需要在写入数据之前，指定 urlargs 字段的映射类型为 nested object。命令如下：

```
curl -XPOST http://127.0.0.1:9200/logstash-2015.06.01/_mapping -d '{
  "accesslog" : {
    "properties" : {
      "urlargs" : {
        "type" : "nested",
        "properties" : {
            "key" : { "type" : "string", "index" : "not_analyzed", "doc_values" : true },
            "value" : { "type" : "string", "index" : "not_analyzed", "doc_values" : true }
        }
      }
    }
  } 
}'
```

这样，数据实际是类似这样的结构：

```
{
  "urlpath" : ["/index.do"],
},
{
  "urlargs.key" : ["uid"],
  "urlargs.value" : ["1234567890"],
},
{
  "urlargs.key" : ["action"],
  "urlargs.value" : ["payload"],
}
```

当然，nested object 节省字段映射的优势对应的是它在使用的复杂。Query 和 Aggs 都必须使用专门的 nested query 和 nested aggs 才能正确读取到它。

nested query 语法如下：

```
curl -XPOST http://127.0.0.1:9200/logstash-2015.06.01/accesslog/_search -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "urlpath" : "/index.do" }}, 
        {
          "nested": {
            "path": "urlargs", 
            "query": {
              "bool": {
                "must": [ 
                  { "match": { "urlargs.key": "uid" }},
                  { "match": { "urlargs.value": "1234567890" }}
                ]
        }}}}
      ]
}}}'
```

nested aggs 语法如下：

```
curl -XPOST http://127.0.0.1:9200/logstash-2015.06.01/accesslog/_search -d '
{
  "aggs": {
    "topnuid": {
      "nested": {
        "path": "urlargs"
      },
      "aggs": {
        "uid": {
          "filter": {
            "term": {
              "urlargs.key": "uid",
            }
          },
          "aggs": {
            "topn": {
              "terms": { 
                "field": "urlargs.value"
              }
            }
          }
        }
      }
    }
  }
}'
```
