## Logstash 的 监控 API

Logstash 5.0 开始，提供了输出自身进程的指标和状态监控的 API。这大大降低了我们监控 Logstash 的难度。

目前 API 主要有两类：

### 节点指标

node stats 接口目前支持三种类型的指标：

#### events

获取该指标的方式为：

```
curl -s localhost:9600/_node/stats/events?pretty=true
```

是的，Logstash 跟 Elasticsearch 一样也支持用 `?pretty` 参数美化 JSON 输出。此外，还支持 `?format=yaml` 来输出 YAML 格式的指标统计。Logstash 默认监听在 9600 端口提供这些 API 访问。如果需要修改，通过 `--http.port` 命令行参数，或者对应的 `logstash.yml` 设置修改。

该指标的响应结果示例如下：

```json
{
    "events" : {
        "in" : 59685,
        "filtered" : 59685,
        "out" : 59685
    }
}
```

#### jvm

获取该指标的方式为：

```
curl -s localhost:9600/_node/stats/jvm?pretty=true
```

该指标的响应结果示例如下：

```
{
    "jvm" : {
        "threads" : {
            "count" : 32,
            "peak_count" : 34
        }
    }
}
```

#### process

获取该指标的方式为：

```
curl -s localhost:9600/_node/stats/process?pretty=true
```

该指标的响应结果示例如下：

```
{
    "process" : {
            "peak_open_file_descriptors" : 64,
            "max_file_descriptors" : 10240,
            "open_file_descriptors" : 64,
        "mem" : {
            "total_virtual_in_bytes" : 5278068736
        },
        "cpu" : {
            "total_in_millis" : 103290097000,
            "percent" : 0
        }
    }
}
```

目前 beats 家族有个 [logstashbeat](https://github.com/consulthys/logstashbeat) 项目，就是专门采集这个数据的。

### 热线程统计

上面的指标值可能比较适合的是长期趋势的监控，在排障的时候，更需要的是即时的线程情况统计。获取方式如下：

```
curl -s localhost:9600/_node/stats/hot_threads?human=true
```

该接口默认返回也是 JSON 格式，在看堆栈的时候并不方便，可以用 `?human=true` 参数来改成文本换行的样式。效果上跟我们看 Elasticsearch 的 `/_nodes/_local/hot_threads` 效果就一样了。

其实节点指标 API 也有 `?human=true` 参数，其作用和 `hot_threads` 不一样，是把一些网络字节数啊，时间啊，改成人类更易懂的大单位。
