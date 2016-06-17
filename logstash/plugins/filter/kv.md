# Key-Value 切分

在很多情况下，日志内容本身都是一个类似于 key-value 的格式，但是格式具体的样式却是多种多样的。logstash 提供 `filters/kv` 插件，帮助处理不同样式的 key-value 日志，变成实际的 LogStash::Event 数据。

## 配置示例

```
filter {
    ruby {
        init => "@kname = ['method','uri','verb']"
        code => "
            new_event = LogStash::Event.new(Hash[@kname.zip(event.get('request').split('|'))])
            new_event.remove('@timestamp')
            event.append(new_event)""
        "
    }
    if [uri] {
        ruby {
            init => "@kname = ['url_path','url_args']"
            code => "
                new_event = LogStash::Event.new(Hash[@kname.zip(event.get('uri').split('?'))])
                new_event.remove('@timestamp')
                event.append(new_event)""
            "
        }
        kv {
            prefix => "url_"
            source => "url_args"
            field_split => "&"
            remove_field => [ "url_args", "uri", "request" ]
        }
    }
}
```

## 解释

Nginx 访问日志中的 `$request`，通过这段配置，可以详细切分成 `method`, `url_path`, `verb`, `url_a`, `url_b` ...

进一步的，如果 url_args 中有过多字段，可能导致 Elasticsearch 集群因为频繁 update mapping 或者消耗太多内存在 cluster state 上而宕机。所以，更优的选择，是只保留明确有用的 url_args 内容，其他部分舍去。

```
    kv {
        prefix => "url_"
        source => "url_args"
        field_split => "&"
        include_keys => [ "uid", "cip" ]
        remove_field => [ "url_args", "uri", "request" ]
    }
```

上例即表示，除了 `url_uid` 和 `url_cip` 两个字段以外，其他的 `url_*` 都不保留。
