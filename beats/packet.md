# packetbeat

目前 packetbeat 支持的网络协议有：HTTP，MySQL，PostgreSQL，Redis，Thrift，DNS，MongoDB，Memcache。

对于很多 Elastic Stack 新手来说，面对的很可能就是几种常用数据流，而书写 logstash 正则是一个耗时耗力的重复劳动，文件落地本身又是多余操作，packetbeat 的运行方式，无疑是对新手入门极大的帮助。

## 安装部署

packetbeat 同样有已经编译完成的软件包可以直接安装使用。需要注意的是，packetbeat 支持不同的抓包方式，也就有不同的依赖。比如最通用的 *pcap*，就要求安装有 `libpcap` 包，*pf_ring* 就要求有 `pfring` 包，而在 windows 平台上，则需要下载安装 [WinPcap 软件](http://www.winpcap.org/install/default.htm)。

```
# yum install libpcap
# rpm -ivh http://www.nmon.net/packages/rpm6/x86_64/PF_RING/pfring-6.1.1-83.x86_64.rpm
# rpm -ivh https://download.elasticsearch.org/beats/packetbeat/packetbeat-5.0.0-x86_64.rpm
```

packetbeat 还附带了一个定制的 Elasticsearch 模板，要在正式使用前导入 ES 中。

```
# curl -XPUT 'http://localhost:9200/_template/packetbeat' -d@/etc/packetbeat/packetbeat.template.json
```

## 配置示例

通过 RPM 安装的 packetbeat 配置文件位于 `/etc/packetbeat/packetbeat.yml`。其示例如下：

```
interfaces:
    device: any
    type: af_packet
    snaplen: 65536                               # 保证大过 MTU 即可，所以公网抓包的话可以改成 1514
    buffer_size_mb: 30                           # 该参数仅在 af_packet 时有效，表示在 linux 内核空间和用户空间之间开启多大共享内存，可以减低 CPU 消耗
    with_vlans: false                            # 在生成抓包语句的时候，是否带上 vlan 标示。示例："port 80 or port 3306 or (vlan and (port 80 or port 3306))"
protocols:
    dns:
        ports: [53]
        send_request: false                      # 通用配置：是否将原始的 request 字段内容都发给 ES
        send_response: false                     # 通用配置：是否将原始的 response 字段内容都发给 ES
        transaction_timeout: "10s"               # 通用配置：超过 10 秒的不再等待响应，直接认定为一个事务，发送给 ES
        include_additionals: false               # DNS 专属配置：是否带上 dns.additionals 字段
        include_authorities: false               # DNS 专属配置：是否带上 dns.authority 字段
    http:
        ports: [80, 8080]                        # 抓包的监听端口
        hide_keywords: ["password","passwd"]     # 对 GET 方法的请求参数，POST 方法的顶层参数里的指定关键信息脱敏。注意：如果你又开启了 send_request，request 字段里的并不会脱敏"
        redact_authorization: true               # 对 header 中的 Authorization 和 Proxy-Authorization 内容做模糊处理。道理和上一条类似
        send_headers: ["User-Agent"]             # 只在 headers 对象里记录指定的 header 字段内容
        send_all_headers: true                   # 在 headers 对象里记录所有 header 字段内容
        include_body_for: ["text/html"]          # 在开启 send_response 的前提下，只记录某些类别的响应体内容
        split_cookie: true                       # 将 headers 对象里的 Cookie 或 Set-Cookie 字段内容再继续做 KV 切割
        real_ip_header: "X-Forwarded-For"        # 将 headers 对象里的指定字段内容设为 client_location 和 real_ip 字段
    memcache:
        ports: [11211]
        parseunknown: false
        maxvalues: 0
        maxbytespervalue: 100
        udptransactiontimeout: 10000
    mysql:
        ports: [3306]
        max_rows: 10                             # 发送给 ES 的 SQL 内容最多为 10 行
        max_row_length: 1024                     # 发送给 ES 的 SQL 内容每行最长为 1024 字节
    cassandra:
        send_request_header: true                # 在开启了 send_request 的前提下，记录 cassandra_request.request_headers 字段
        send_response_header: true
        compressor: "snappy"
        ignored_ops: ["SUPPORTED","OPTIONS"]     # 不记录部分操作
    amqp:
        ports: [5672]
        max_body_length: 1000                    # 超长的都截断掉
        parse_headers: true
        parse_arguments: false
        hide_connection_information: true        # 不记录打开、关闭连接之类的信息
    thrift:
        transport_type: socket                   # Thrift 传输类型，"socket" 或 "framed"
        protocol_type: binary
        idl_files: ["shared.thrift"]             # 一般来说默认内置的 IDL 文件已经足够了
        string_max_size: 200                     # 参数或返回值中字符串的最大长度，超长的都截断掉
        collection_max_size: 15                  # Thrift 的 list, map, set, structure 结构中的元素个数上限，超过的元素丢弃
        capture_reply: true                      # 为了节省资源，可以设置 false，只解析响应中的方法名称，其余信息丢弃
        obfuscate_strings: true                  # 把所有方法参数、返回码、异常结构中的字符串都用 * 代替
        drop_after_n_struct_fields: 500          # 在 packetbeat 结束整个事务之前，一个结构体最多能存多少个字段。该配置主要考虑节约内存
    monogodb:
        max_docs: 10                             # 设置 send_response 前提下，response 字段最多保存多少个文档。超长的都截断掉。不限则设为 0
        max_doc_length: 5000                     # response 字段里每个文档的最大字节数。超长截断。不限则设为 0
procs:
    enabled: true
    monitored:
        - process: mysqld
          cmdline_grep: mysqld
        - process: app                           # 设置发送给 ES 的数据中进程名字段的内容
          cmdline_grep: gunicorn                 # 从 /proc/<pid>/cmdline 中过滤实际进程名的字符串

output:
    ...
```

在 windows 上，网卡设备名称会比较长。所以 packetbeat 单独提供了一个参数：`packetbeat -device`，返回整个可用网卡设备列表数组，你可以直接写数组下标来代表这个设备。比如：`device: 0`。

## 字段

* `@timestamp` 事件时间
* `type` 事件类型，比如 HTTP, MySQL, Redis, RUM 等
* `count` 事件代表的实际事务数。也就是采样率的倒数
* `direction` 事务流向。比如 "in" 或者 "out"
* `status` 事务状态。不同协议取值方式可能不一致
* `method` 事务方法。对 HTTP 是 GET/POST/PUT 等，对 SQL 是 SELECT/UPDATE/INSERT 等
* `resource` 事务关联的逻辑资源。对 HTTP 是 URL 路径的第一个层级，对 SQL 是表名
* `path` 事务关联的路径。对 HTTP 是 URL 路径，对 SQL 是表名，对 KV 存储是 key
* `query` 人类可读的请求字符串。对 HTTP 是 GET /resource/path?key=value，对 SQL 是 SELECT id FROM table WHERE name=test
* `params` 请求参数。对 HTTP 就是 GET 或者 POST 的请求参数，对 Thrift 是 request 里的参数
* `notes` packetbeat 解析数据出错时的日志记录

其余根据采用协议不同，各自有自己的字段。

## dashboard 效果

针对 packetbeat 自动识别的不同协议，packetbeat 还自带了几个预定义好的 Kibana dashboard 方便使用和查看。包括：

* Packetbeat Statistics: 针对 HTTP 和标准流量事件的性能统计仪表盘
* Packetbeat Search: 用来搜索关键字的仪表盘
* MySQL Performance: MySQL 性能分析仪表盘
* PgSQL Performance: PgSQL 性能分析仪表盘

预定义仪表盘的导入方式如下：

```
# git clone https://github.com/elastic/packetbeat-dashboards
# cd packetbeat-dashboards
# ./load.sh http://192.168.0.2:9200
```

效果如下：
![statistics](https://github.com/elastic/packetbeat-dashboards/raw/master/screenshots/Packetbeat-statistics.png)
![mysql performance](https://github.com/elastic/packetbeat-dashboards/raw/master/screenshots/MySql-performance.png)

### Kibana3 topology

其实在加入 Elastic.co 之前，packetbeat 曾经自己 fork 了一个 Kibana3 的分支，并在此基础上二次开发了一个专门用来展示网络拓扑结构的面板，叫 force panel。该特性至今依然只能运行在 Kibana3 上。所以，需要网络拓扑展现的用户，还得继续使用 Kibana3。部署方式如下：

```
curl -L -O https://github.com/packetbeat/kibana/releases/download/v3.1.2-pb/kibana-3.1.2-packetbeat.tar.gz
tar xzvf kibana-3.1.2-packetbeat.tar.gz
curl -L -O https://download.elasticsearch.org/beats/packetbeat/packetbeat-dashboards-k3-1.0.0~Beta1.tar.gz
tar xzvf packetbeat-dashboards-k3-1.0.0~Beta1.tar.gz
cd packetbeat-dashboards-k3-1.0.0~Beta1/
./load.sh 192.169.0.2
```

force panel 示例如下图。注意，force panel 用到的数据，其实质是对各来源 IP 分别请求目的 IP，对 ES 的计算量要求较大，并不适合在高流量高负载的条件下使用。

![](https://www.elastic.co/guide/en/beats/packetbeat/current/images/topology_map.png)

### 小贴士

pfring 抓包模式的原厂，ntop 公司，也有类似 packetbeat 的计划。ntopng/nProbe 除了储存到 SQLite 以外，也开始支持存储到 Elasticsearch 中。不过它们推荐采用的 dashboard，是 Kibana3 的另一个 fork 分支，叫 Qbana。

![](http://www.ntop.org/wp-content/uploads/2015/06/687474703a2f2f692e696d6775722e636f6d2f396758544b43642e706e67.png)

有兴趣的读者可以参考 ntop 官方文档：<http://www.ntop.org/ntopng/exploring-your-traffic-using-ntopng-with-elasticsearchkibana/>
