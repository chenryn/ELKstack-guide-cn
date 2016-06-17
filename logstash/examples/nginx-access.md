# Nginx 访问日志

## grok

Logstash 默认自带了 apache 标准日志的 grok 正则:

```
COMMONAPACHELOG %{IPORHOST:clientip} %{USER:ident} %{NOTSPACE:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-)
COMBINEDAPACHELOG %{COMMONAPACHELOG} %{QS:referrer} %{QS:agent}
```

对于 nginx 标准日志格式，可以发现只是最后多了一个 `$http_x_forwarded_for` 变量。所以 nginx 标准日志的 grok 正则定义是：

```
MAINNGINXLOG %{COMBINEDAPACHELOG} %{QS:x_forwarded_for}
```

自定义的日志格式，可以照此修改。

# split

nginx 日志因为部分变量中内含空格，所以很多时候只能使用 `%{QS}` 正则来做分割，性能和细度都不太好。如果能自定义一个比较少见的字符作为分隔符，那么处理起来就简单多了。假设定义的日志格式如下：

```
log_format main "$http_x_forwarded_for | $time_local | $request | $status | $body_bytes_sent | "
                "$request_body | $content_length | $http_referer | $http_user_agent | $nuid | "
                "$http_cookie | $remote_addr | $hostname | $upstream_addr | $upstream_response_time | $request_time";
```

实际日志如下：

> 117.136.9.248 | 08/Apr/2015:16:00:01 +0800 | POST /notice/newmessage?sign=cba4f614e05db285850cadc696fcdad0&token=JAGQ92Mjs3--gik_b_DsPIQHcyMKYGpD&did=4b749736ac70f12df700b18cd6d051d5&osn=android&osv=4.0.4&appv=3.0.1&net=460-02-2g&longitude=120.393006&latitude=36.178329&ch=360&lp=1&ver=1&ts=1428479998151&im=869736012353958&sw=0&sh=0&la=zh-CN&lm=weixin&dt=vivoS11t HTTP/1.1 | 200 | 132 | abcd-sign-v1://dd03c57f8cb6fcef919ab5df66f2903f:d51asq5yslwnyz5t/{\x22type\x22:4,\x22uid\x22:7567306} | 89 | - | abcd/3.0.1, Android/4.0.4, vivo S11t | nuid=0C0A0A0A01E02455EA7CF47E02FD072C1428480001.157 | - | 10.10.10.13 | bnx02.abcdprivate.com | 10.10.10.22:9999 | 0.022 | 0.022
> 59.50.44.53 | 08/Apr/2015:16:00:01 +0800 | POST /feed/pubList?appv=3.0.3&did=89da72550de488328e2aba5d97850e9f&dt=iPhone6%2C2&im=B48C21F3-487E-4071-9742-DC6D61710888&la=cn&latitude=0.000000&lm=weixin&longitude=0.000000&lp=-1.000000&net=0-0-wifi&osn=iOS&osv=8.1.3&sh=568.000000&sw=320.000000&token=7NobA7asg3Jb6n9o4ETdPXyNNiHwMs4J&ts=1428480001275 HTTP/1.1 | 200 | 983 | abcd-sign-v1://b398870a0b25b29aae65cd553addc43d:72214ee85d7cca22/{\x22nextkey\x22:\x22\x22,\x22uid\x22:\x2213062545\x22,\x22token\x22:\x227NobA7asg3Jb6n9o4ETdPXyNNiHwMs4J\x22} | 139 | - | Shopping/3.0.3 (iPhone; iOS 8.1.3; Scale/2.00) | nuid=0C0A0A0A81DF2455017D548502E48E2E1428480001.154 | nuid=CgoKDFUk34GFVH0BLo7kAg== | 10.10.10.11 | bnx02.abcdprivate.com | 10.10.10.35:9999 | 0.025 | 0.026

然后还可以针对 request 做更细致的切分。比如 URL 参数部分。很明显，URL 参数中的字段，顺序是乱的，第一行，问号之后的第一个字段是 sign，第二行问号之后的第一个字段是 appv，所以需要将字段进行切分，取出每个字段对应的值。官方自带 grok 满足不了要求。最终采用的 logstash 配置如下：

```
filter {
    ruby {
        init => "@kname = ['http_x_forwarded_for','time_local','request','status','body_bytes_sent','request_body','content_length','http_referer','http_user_agent','nuid','http_cookie','remote_addr','hostname','upstream_addr','upstream_response_time','request_time']"
        code => "
            new_event = LogStash::Event.new(Hash[@kname.zip(event.get('message').split('|'))])
            new_event.remove('@timestamp')
            event.append(new_event)""
        "
    }
    if [request] {
        ruby {
            init => "@kname = ['method','uri','verb']"
            code => "
                new_event = LogStash::Event.new(Hash[@kname.zip(event.get('request').split(' '))])
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
                field_split => "& "
                remove_field => [ "url_args","uri","request" ]
            }
        }
    }
    mutate {
        convert => [
            "body_bytes_sent" , "integer",
            "content_length", "integer",
            "upstream_response_time", "float",
            "request_time", "float"
        ]
    }
    date {
        match => [ "time_local", "dd/MMM/yyyy:hh:mm:ss Z" ]
        locale => "en"
    }
}
```

最终结果：

```
{
                   "message" => "1.43.3.188 | 08/Apr/2015:16:00:01 +0800 | POST /search/suggest?appv=3.0.3&did=dfd5629d705d400795f698055806f01d&dt=iPhone7%2C2&im=AC926907-27AA-4A10-9916-C5DC75F29399&la=cn&latitude=-33.903867&lm=sina&longitude=151.208137&lp=-1.000000&net=0-0-wifi&osn=iOS&osv=8.1.3&sh=667.000000&sw=375.000000&token=_ovaPz6Ue68ybBuhXustPbG-xf1WbsPO&ts=1428480001567 HTTP/1.1 | 200 | 353 | abcd-sign-v1://a24b478486d3bb92ed89a901541b60a5:b23e9d2c14fe6755/{\\x22key\\x22:\\x22last\\x22,\\x22offset\\x22:\\x220\\x22,\\x22token\\x22:\\x22_ovaPz6Ue68ybBuhXustPbG-xf1WbsPO\\x22,\\x22limit\\x22:\\x2220\\x22} | 148 | - | abcdShopping/3.0.3 (iPhone; iOS 8.1.3; Scale/2.00) | nuid=0B0A0A0A9A64AF54F97634640230944E1428480001.113 | nuid=CgoKC1SvZJpkNHb5TpQwAg== | 10.10.10.11 | bnx02.abcdprivate.com | 10.10.10.26:9999 | 0.070 | 0.071",
                  "@version" => "1",
                "@timestamp" => "2015-04-08T08:00:01.000Z",
                      "type" => "nginxapiaccess",
                      "host" => "blog05.abcdprivate.com",
                      "path" => "/home/nginx/logs/api.access.log",
      "http_x_forwarded_for" => "1.43.3.188",
                "time_local" => " 08/Apr/2015:16:00:01 +0800",
                    "status" => "200",
           "body_bytes_sent" => 353,
              "request_body" => "abcd-sign-v1://a24b478486d3bb92ed89a901541b60a5:b23e9d2c14fe6755/{\\x22key\\x22:\\x22last\\x22,\\x22offset\\x22:\\x220\\x22,\\x22token\\x22:\\x22_ovaPz6Ue68ybBuhXustPbG-xf1WbsPO\\x22,\\x22limit\\x22:\\x2220\\x22}",
            "content_length" => 148,
              "http_referer" => "-",
           "http_user_agent" => "abcdShopping/3.0.3 (iPhone; iOS 8.1.3; Scale/2.00)",
                      "nuid" => "nuid=0B0A0A0A9A64AF54F97634640230944E1428480001.113",
               "http_cookie" => "nuid=CgoKC1SvZJpkNHb5TpQwAg==",
               "remote_addr" => "10.10.10.11",
                  "hostname" => "bnx02.abcdprivate.com",
             "upstream_addr" => "10.10.10.26:9999",
    "upstream_response_time" => 0.070,
              "request_time" => 0.071,
                    "method" => "POST",
                      "verb" => "HTTP/1.1",
                  "url_path" => "/search/suggest",
                  "url_appv" => "3.0.3",
                   "url_did" => "dfd5629d705d400795f698055806f01d",
                    "url_dt" => "iPhone7%2C2",
                    "url_im" => "AC926907-27AA-4A10-9916-C5DC75F29399",
                    "url_la" => "cn",
              "url_latitude" => "-33.903867",
                    "url_lm" => "sina",
             "url_longitude" => "151.208137",
                    "url_lp" => "-1.000000",
                   "url_net" => "0-0-wifi",
                   "url_osn" => "iOS",
                   "url_osv" => "8.1.3",
                    "url_sh" => "667.000000",
                    "url_sw" => "375.000000",
                 "url_token" => "_ovaPz6Ue68ybBuhXustPbG-xf1WbsPO",
                    "url_ts" => "1428480001567"
}
```

如果 url 参数过多，可以不使用 kv 切割，或者预先定义成 nested object 后，改成数组形式：

```
        if [uri] {
            ruby {
                init => "@kname = ['url_path','url_args']"
                code => "
                    new_event = LogStash::Event.new(Hash[@kname.zip(event.get('request').split('?'))])
                    new_event.remove('@timestamp')
                    event.append(new_event)""
                "
            }
            if [url_args] {
                ruby {
                    init => "@kname = ['key','value']"
                    code => "event.set('nested_args', event.get('url_args').split('&').collect {|i| Hash[@kname.zip(i.split('='))]})"
                    remove_field => [ "url_args","uri","request" ]
                }
            }
        }
```

采用 nested object 的优化原理和 nested object 的使用方式，请阅读稍后 Elasticsearch 调优章节。

## json format

自定义分隔符虽好，但是配置写起来毕竟复杂很多。其实对 logstash 来说，nginx 日志还有另一种更简便的处理方式。就是自定义日志格式时，通过手工拼写，直接输出成 JSON 格式：

```
log_format json '{"@timestamp":"$time_iso8601",'
                 '"host":"$server_addr",'
                 '"clientip":"$remote_addr",'
                 '"size":$body_bytes_sent,'
                 '"responsetime":$request_time,'
                 '"upstreamtime":"$upstream_response_time",'
                 '"upstreamhost":"$upstream_addr",'
                 '"http_host":"$host",'
                 '"url":"$uri",'
                 '"xff":"$http_x_forwarded_for",'
                 '"referer":"$http_referer",'
                 '"agent":"$http_user_agent",'
                 '"status":"$status"}';
```

然后采用下面的 logstash 配置即可：

```
input {
    file {
        path => "/var/log/nginx/access.log"
        codec => json
    }
}
filter {
    mutate {
        split => [ "upstreamtime", "," ]
    }
    mutate {
        convert => [ "upstreamtime", "float" ]
    }
}
```

这里采用多个 mutate 插件，因为 upstreamtime 可能有多个数值，所以先切割成数组以后，再分别转换成浮点型数值。而在 mutate 中，convert 函数执行优先级高于 split 函数，所以只能分开两步写。mutate 内各函数优先级顺序，之前插件介绍章节有详细说明，读者可以返回去加强阅读。

## syslog

Nginx 从 1.7 版开始，加入了 syslog 支持。Tengine 则更早。这样我们可以通过 syslog 直接发送日志出来。Nginx 上的配置如下：

```
access_log syslog:server=unix:/data0/rsyslog/nginx.sock locallog;
```

或者直接发送给远程 logstash 机器：

```
access_log syslog:server=192.168.0.2:5140,facility=local6,tag=nginx-access,severity=info logstashlog;
```

默认情况下，Nginx 将使用 `local7.info` 等级，`nginx` 为标签，发送数据。注意，采用 syslog 发送日志的时候，无法配置 `buffer=16k` 选项。
