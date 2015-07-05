# packetbeat

之前已经提到过，packetbeat 已经被 Elastic 公司收归旗下，未来就会替代 logstash-forwarder 成为 ELKstack 套件的一部分。那么，为什么 Elastic 公司如此看好 packetbeat 项目，不惜废掉 logstash-forwarder 呢？

packetbeat 和 logstash-forwarder 一样，也是一个 golang 写的开源项目。其特色在于，logstash-forwarder 的数据来源是文件或者标准输入，都是文本流。而packetbeat 采用 libpcap 库，抓取网络流量，识别其中的特定网络协议，自动按照协议规范，将网络流量包划分成事件字段，写入到 Elasticsearch 中。

目前 packetbeat 支持的网络协议有：HTTP，MySQL，PostgreSQL，Redis，Thrift。

对于很多 ELKstack 新手来说，面对的很可能就是几种常用数据流，而书写 logstash 正则是一个耗时耗力的重复劳动，文件落地本身又是多余操作，packetbeat 的运行方式，无疑是对新手入门极大的帮助。

目前 packetbeat 项目地址为：<https://github.com/elastic/packetbeat>。

## 安装部署

packetbeat 同样有已经编译完成的软件包可以直接安装使用。需要注意的是，packetbeat 支持不同的抓包方式，也就有不同的依赖。比如最通用的 *pcap*，就要求安装有 `libpcap` 包，*pf_ring* 就要求有 `pfring` 包。

```
# yum install libpcap
# rpm -ivh http://www.nmon.net/packages/rpm6/x86_64/PF_RING/pfring-6.1.1-83.x86_64.rpm
# rpm -ivh https://download.elasticsearch.org/beats/packetbeat/packetbeat-1.0.0~Beta1-x86_64.rpm
```

packetbeat 还附带了一个定制的 Elasticsearch 模板，要在正式使用前导入 ES 中。

```
# curl -XPUT 'http://localhost:9200/_template/packetbeat' -d@/etc/packetbeat/packetbeat.template.json
```

## 配置示例

通过 RPM 安装的 packetbeat 配置文件位于 `/etc/packetbeat/packetbeat.yml`。其基础示例如下：

```
shipper:
  tags: ["web"]
interfaces:
  device: any
  type: af_packet
  buffer_size_mb: 100
protocols:
  http:
    ports: [80, 8080]
    send_headers: ["User-Agent"]
    real_ip_header: "X-Forwarded-For"
  mysql:
    ports: [3306]
output:
  elasticsearch:
    enabled: true
    host: "192.168.0.2"
```

shipper 默认会以本机 IP 地址作为 name，interfaces 支持 pcap，af_packet 和 pf_ring 三种模式。output 除了直接给 ES，以外，还可以给 Redis，再用 logstash-input-redis 接收数据写 ES。

```
output:
  elasticsearch:
    enabled: false
  redis:
    enabled: true
    host: "192.168.0.3"
    port: 6379
    save_topology: true
```

然后 logstash 配置如下，注意因为 packetbeat 自带的 template 是匹配 `packetbeat-*` 索引的：

```
input {
    redis {
        codec => "json"
        host => "192.168.0.3"
        port => 6379
        data_type => "list"
        key => "packetbeat"
    }
}

output {
    elasticsearch {
        protocol => "http"
        host => "127.0.0.1"
        sniffing => true
        manage_template => false
        index => "packetbeat-%{+YYYY.MM.dd}"
    }
}
```

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

其实在 Kibana4 推出之前，packetbeat 曾经自己 fork 了一个 Kibana3 的分支，并在此基础上二次开发了一个专门用来展示网络拓扑结构的面板，叫 force panel。该特性至今依然只能运行在 Kibana3 上。所以，需要网络拓扑展现的用户，还得继续使用 Kibana3。部署方式如下：

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
