# 安排、配置和运行

Kibana4 安装方式依然简单，你可以在几分钟内安装好 Kibana 然后开始探索你的 Elasticsearch 索引。只需要预备：

* Elasticsearch 1.4.4 或者更新的版本
* 一个现代浏览器 - [支持的浏览器列表](http://www.elasticsearch.com/support/matrix?_ga=1.149082614.1575542547.1409213558).
* 有关你的 Elasticsearch 集群的信息：
  * 你想要连接 Elasticsearch 实例的 URL
  * 你想搜索哪些 Elasticsearch 索引

> 如果你的 Elasticsearch 是被 [Shield](http://www.elasticsearch.org/overview/shield/) 保护着的，阅读[生产环境部署章节](./production.md)相关内容学习额外的安装说明。

## 安装并启动 kibana

要安装启动 Kibana:

1. 下载对应平台的 [Kibana 4 二进制包](http://www.elasticsearch.org/overview/kibana/installation/)
2. 解压 `.zip` 或 `tar.gz` 压缩文件
3. 在安装目录里运行: `bin/kibana` (Linux/MacOSX) 或 `bin\kibana.bat` (Windows)

完毕！Kibana 现在运行在 5601 端口了。

## 让 kibana 连接到 elasticsearch

在开始用 Kibana 之前，你需要告诉它你打算探索哪个 Elasticsearch 索引。第一次访问 Kibana 的时候，你会被要求定义一个 *index pattern* 用来匹配一个或者多个索引名。好了。这就是你需要做的全部工作。以后你还可以随时从 [Settings 标签页](./settings.md)添加更多的 index pattern。

> 默认情况下，Kibana 会连接运行在 `localhost` 的 Elasticsearch。要连接其他 Elasticsearch 实例，修改 `kibana.yml` 里的 Elasticsearch URL，然后重启 Kibana。如何在生产环境下使用 Kibana，阅读[生产环境部署章节](./production.md)。

要从 Kibana 访问的 Elasticsearch 索引的配置方法：

1. 从浏览器访问 Kibana 界面。也就是说访问比如 `localhost:5601` 或者 `http://YOURDOMAIN.com:5601`。
![](https://www.elastic.co/guide/en/kibana/current/images/Start-Page.png)
2. 指定一个可以匹配一个或者多个 Elasticsearch 索引的 index pattern 。默认情况下，Kibana 认为你要访问的是通过 Logstash 导入 Elasticsearch 的数据。这时候你可以用默认的 `logstash-*` 作为你的 index pattern。通配符(*) 匹配索引名中零到多个字符。如果你的 Elasticsearch 索引有其他命名约定，输入合适的 pattern。pattern 也开始是最简单的单个索引的名字。
3. 选择一个包含了时间戳的索引字段，可以用来做基于时间的处理。Kibana 会读取索引的映射，然后列出所有包含了时间戳的字段(译者注：实际是字段类型为 date 的字段，而不是“看起来像时间戳”的字段)。如果你的索引没有基于时间的数据，关闭 `Index contains time-based events` 参数。
4. 如果一个新索引是定期生成，而且索引名中带有时间戳，选择 `Use event times to create index names` 选项，然后再选择 `Index pattern interval`。这可以提高搜索性能，Kibana 会至搜索你指定的时间范围内的索引。在你用 Logstash 输出数据给 Elasticsearch 的情况下尤其有效。
5. 点击 `Create` 添加 index pattern。第一个被添加的 pattern 会自动被设置为默认值。如果你有多个 index pattern 的时候，你可以在 `Settings > Indices` 里设置具体哪个是默认值。

好了。Kibana 现在连接上你的 Elasticsearch 数据了。Kibana 会显示匹配上的索引里的字段名的只读列表。

## 开始探索你的数据！

你可以开始下钻你的数据了：

* 在 [Discover](./discover.md) 页搜索和浏览你的数据。
* 在 [Visualize](./visualize.md) 页转换数据成图表。
* 在 [Dashboard](./dashboard.md) 页创建定制自己的仪表板。
