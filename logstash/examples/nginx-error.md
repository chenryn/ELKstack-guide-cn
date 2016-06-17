# Nginx 错误日志

Nginx 错误日志是运维人员最常见但又极其容易忽略的日志类型之一。Nginx 错误日志即没有统一明确的分隔符，也没有特别方便的正则模式，但通过 logstash 不同插件的组合，还是可以轻松做到数据处理。

值得注意的是，Nginx 错误日志中，有一类数据是接收过大请求体时的报错，默认信息会把请求体的具体字节数记录下来。每次请求的字节数基本都是在变化的，这意味着常用的 topN 等聚合函数对该字段都没有明显效果。所以，对此需要做一下特殊处理。

最后形成的 logstash 配置如下所示：

```
filter {
    grok {
        match => { "message" => "(?<datetime>\d\d\d\d/\d\d/\d\d \d\d:\d\d:\d\d) \[(?<errtype>\w+)\] \S+: \*\d+ (?<errmsg>[^,]+), (?<errinfo>.*)$" }
    }
    mutate {
        rename => [ "host", "fromhost" ]
        gsub => [ "errmsg", "too large body: \d+ bytes", "too large body" ]
    }
    if [errinfo]
    {
        ruby {
            code => "
                new_event = LogStash::Event.new(Hash[event.get('errinfo').split(', ').map{|l| l.split(': ')}])
                new_event.remove('@timestamp')
                event.append(new_event)""
            "
        }
    }
    grok {
        match => { "request" => '"%{WORD:verb} %{URIPATH:urlpath}(?:\?%{NGX_URIPARAM:urlparam})?(?: HTTP/%{NUMBER:httpversion})"' }
        patterns_dir => "/etc/logstash/patterns"
        remove_field => [ "message", "errinfo", "request" ]
    }
}
```

经过这段 logstash 配置的 Nginx 错误日志生成的事件如下所示：

```
{
    "@version": "1",
    "@timestamp": "2015-07-02T01:26:40.000Z",
    "type": "nginx-error",
    "errtype": "error",
    "errmsg": "client intended to send too large body",
    "fromhost": "web033.mweibo.yf.sinanode.com",
    "client": "36.16.7.17",
    "server": "api.v5.weibo.cn",
    "host": "\"api.weibo.cn\"",
    "verb": "POST",
    "urlpath": "/2/client/addlog_batch",
    "urlparam": "gsid=_2A254UNaSDeTxGeRI7FMX9CrEyj2IHXVZRG1arDV6PUJbrdANLROskWp9bXakjUZM5792FW9A5S9EU4jxqQ..&wm=3333_2001&i=0c6f156&b=1&from=1053093010&c=iphone&v_p=21&skin=default&v_f=1&s=8f14e573&lang=zh_CN&ua=iPhone7,1__weibo__5.3.0__iphone__os8.3",
    "httpversion": "1.1"
}
```
