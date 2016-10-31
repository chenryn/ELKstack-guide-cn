# 扩展和测试方案

在体验完 Elasticsearch 便捷的操作后，下一步一定会碰到的问题是：数据写入变慢了，机器变卡了，是需要做优化呢？还是需要扩容设备了？如果做扩容，索引的分片和副本设置多少才合适？如果做优化，某个参数能造成什么样的影响？

而 ES 集群性能，受服务器硬件、数据结构和长度、请求接口复杂度等各种环节影响颇大。这些问题，都需要有一个标准的测试流程给出答案。

由于 ES 是近乎线性扩展的分布式系统，所以对上述需求我们都可以总结成同一个测试模式：

1. 使用和线上集群相同硬件配置的服务器搭建一个单节点集群。
2. 使用和线上集群相同的映射创建一个 0 副本，1 分片的测试索引。
3. 使用和线上集群相同的数据写入进行压测。
4. 观察写入性能，或者运行查询请求观察搜索聚合性能。
5. 持续压测数小时，使用监控系统记录 eps、requesttime、fielddata cache、GC count 等关键数据。

测试完成后，根据监控系统数据，确定单分片的性能拐点，或者适合自己预期值的临界点。这个数据，就是一个基准数据。之后的扩容计划，都可以以这个基准单位进行。

需要注意的是，测试是以分片为单位的，在实际使用中，因为主分片和副本分片都是在各自节点做 indexing 和 merge 操作，需要消耗同样的写入性能。所以，实际集群的容量预估中，要考虑副本数的影响。也就是说，假如你在基准测试中得到单机写入性能在 10000 eps，那么开启一个副本后所能达到的 eps 就只有 5000 了。还想写入 10000 eps 的话，就需要加一倍机器。

另外，测试中我们使用的配置都尽量贴合当前现状。事实上，很多配置可能其实并不合理。在确定基准线并开始扩容之前，还是要认真调节配置，审核请求使用的接口是否最优，然后反复测试。然后取一个最终的基准值。

审核请求，更是一个长期的过程，就像 DBA 永远需要关注慢查询一样。ES 的慢查询请求处理，请阅读稍后[性能日志](./monitor/logging.md)一节。

## esrally 测试工具

esrally 是 Elastic.co 开源的专门做 Elasticsearch 压测的工具。我们在官网上看到的 nightly benchmark 结果就是用这个工具每晚运行生成的报告。用这个工具，可以很方便的验证自己的代码修改、配置调整对性能的影响效果。

### 安装运行

esrally 依赖 python3.4+，所以需要先安装好 python 3.5。然后直接 `pip3 install esrally` 即可。

被压测的 Elasticsearch 有两种来源：

* 本机有 gradle 工具的，可以从最新的 GitHub master 代码编译
* 没有 gradle 工具的，可以按官方提供的标签，下载对应版本的二进制分发包。

esrally 在压测完毕后，可以把指标数据写入到另一个 ES 索引中，可以很方便的用 Kibana 做图表可视化。这就需要另外配置一下 `~/.rally/rally.ini` 里 reporting 部分的参数：

```
[reporting]
datastore.type = elasticsearch
datastore.host = localhost
datastore.port = 9200
datastore.secure = False
datastore.user =
datastore.password =

```

不用担心这个 ES 会跟一会儿压测运行的 ES 冲突，因为压测启动的 ES 会监听在其他端口上。

我们先简单测试一下标准的运行：：

```
/opt/local/Library/Frameworks/Python.framework/Versions/3.5/bin/esrally --pipeline=from-distribution --distribution-version=5.0.0
```

默认情况下压测采用的数据集叫 geonames，是一个 2.8GB 大的 JSON 数据。ES 也提供了一系列其他类型的压测数据集。如果要切换数据集采用 --track 参数：

```
/opt/local/Library/Frameworks/Python.framework/Versions/3.5/bin/esrally --pipeline=from-distribution --distribution-version=5.0.0 --track=geonames
```

重复运行的时候可以修改 `~/.rally/rally.ini` 里的 `tracks[default.url]`，改为第一次运行时下载的地址：~/.rally/benchmarks/tracks/default 。然后采用离线参数重复运行：

```
/opt/local/Library/Frameworks/Python.framework/Versions/3.5/bin/esrally --offline --pipeline=from-distribution --distribution-version=5.0.0 --track=geonames
```

静静等待程序运行完毕，就会给出一个漂亮的输出结果了。

### 调整压测任务

默认一次压测运行会是一个很漫长的时间，如果你其实只关心部分的性能，比如只关心写入，不关心搜索。其实可以自己去修改一下 track 的任务定义。

track 的定义文件在 `~/.rally/benchmarks/tracks/default/geonames/track.json`。如果你改动较大，建议直接新建一个 track 目录，比如叫 `mytest/track.json`。

对照 geonames 里的定义，一个 track 包括以下部分：

* meta：定义数据来源 URL。
* indices：定义索引名称、索引 mapping 的文件位置、数据的存放位置和校验信息。
* operations：定义一个个操作的名称、类型、索引和请求参数。如果操作类型是 index，可用的索引参数有：client 并发量、bulk 大小、是否强制 merge 等；如果操作类型是 search，可用的请求参数就是一个 queries 数组，按序放好一个个 queryDSL。
* challenges：定义好名称和调用哪些 operation，调用顺序如何。

最后运行命令的时候通过 --challenge= 参数来指定执行哪个任务。

比如我们只关心写入不关心搜索，打开 track.json 可以看到有这么几个 challenges：

```JSON
"challenges": [
    {
        "name": "append-no-conflicts",
        "description": "",
        "schedule": [
            "index-append-default-settings",
            "stats",
            "search"
        ]
    },
    {
        "name": "append-fast-no-conflicts",
        "description": "",
        "schedule": [
            "index-append-fast-settings"
        ]
    },
```

我们就知道了，默认的 `append-no-conflicts` 是要测完写入再测搜索的，而 `append-fast-no-conflicts` 是只测写入的。那么我们这么运行就行：

```
/opt/local/Library/Frameworks/Python.framework/Versions/3.5/bin/esrally --offline --pipeline=from-distribution --distribution-version=5.0.0 --track=geonames --challenge=append-fast-no-conflicts
```

### 调整压测数据

如果要用自己的数据集呢，也一样是在自己的 track.json 里定义，比如：

```
{
    "meta": {
        "data-url": "/Users/raochenlin/.rally/benchmarks/data/splunklog/1468766825_10.json.bz2"
    },
    "indices": [
        {
            "name": "splunklog",
            "types": [
                {
                    "name": "type",
                    "mapping": "mappings.json",
                    "documents": "1468766825_10.json.bz2",
                    "document-count":  924645,
                    "compressed-bytes": 19149532,
                    "uncompressed-bytes": 938012996
                }
            ]
        }
    ],
```

这里就是用的一份 splunkd 的 internal 日志，JSON 导出。字节数大小为 938012996。压缩后为 19149532。
