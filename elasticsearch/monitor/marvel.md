# marvel

marvel 是 Elastic.co 公司推出的官方监控方案，在非商业场合是可以免费使用的。如果在商业生产上使用，Elastic.co 公司的收费标准是：

* 前 5 个节点，每年 1000 美元
* 之后每增加 5 个节点，每年加收 250 美元

## 安装

marvel 是以 elasticsearch 的插件形式存在的，直接通过插件安装即可：

```
./bin/plugin -i elasticsearch/marvel/latest
```

安装之后，插件自动运行，并将定期获取到的集群状态数据，存储在 `.marvel-YYYY.MM.DD` 索引中，以单台 ES 计算，该索引的大小在 500MB 左右。所以，如果在小规模环境下运行，首先请注意，不要让你宝贵的内存都花在 marvel 的数据索引上了。

## 配置

如果不想让 marvel 数据索引影响到生产环境 ES 的运行，可以搭建单独的 marvel 数据集群，而生产数据集群上通过主动汇报的方式把数据发送过去。

在两个集群都安装好 marvel 插件后，生产集群的 `elasticsearch.yml` 上添加如下配置：

```
marvel.agent.exporter.es.hosts: ["marvel-cluster-ip:9200"]
```

数据接收端的 marvel 集群(即上一行写的 marvel-cluster-ip 代表的主机)则添加如下配置：

```
marvel.agent.enabled: false
```

即本身不启用 marvel，以免数据有混淆。

marvel 的监控页面是在 Kibana3 基础上稍有改造。如下图所示，其顶部菜单栏设计了一个下拉选择框，可以切换几个不同纬度的仪表板：

![](https://www.elastic.co/guide/en/marvel/current/images/overview_thumb.png)
