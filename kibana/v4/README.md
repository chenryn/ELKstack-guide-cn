# README

Kibana 是为 Elasticsearch 设计的开源分析和可视化平台。你可以使用 Kibana 来搜索，查看存储在 Elasticsearch 索引中的数据并与之交互。你可以很容易实现高级的数据分析和可视化，以图标的形式展现出来。

Kibana 让海量数据变得更容易理解。简单的基于浏览器的界面让你可以快速创建并分享动态的仪表板，用以实时修改 Elasticsearch 请求。

安装 Kibana 非常简单。你可以在几分钟内安装好 Kibana 然后开始探索你的 Elasticsearch 索引 —— 不需要写代码，不需要额外的架构。

> 本指南讲述的是如何使用 Kibana 4。想了解 Kibana 4 里有什么新特性，请阅读 [What’s New in Kibana 4](../difference.md)。想了解 Kibana 3 的内容，请阅读 [Kibana 3 用户指南](../v3/README.md)。

## 数据发现和可视化

让我们看看你可能要怎么用 Kibana 来探索和展示数据。我们会从伦敦交通局的交通运输卡的一周使用情况里导入一些数据。

在 Kibana 的 Discover 页，我们可以提交搜索请求，过滤结果，然后检查返回的文档里的数据。比如，我们可以通过排除公交出行，获取地铁出行的情况。

![](http://www.elasticsearch.org/guide/en/kibana/current/images/TFL-CompletedTrips.jpg)

现在，我们可以看到早晚上下班高峰期的直方图。默认情况下，Discover 页会显示匹配搜索条件的前 500 个文档。你可以修改时间过滤器，拖拽直方图下钻数据，查看部分文档的细节。Discover 页上如何探索数据，详细说明见 [Discover](./discover.md)。

你可以在 Visualization 页为你的搜索结构构造可视化。每个可视化都是跟一个搜索关联着的。比如，我们可以基于前面那个搜索创建一个战士每周伦敦地铁交通流量的直方图。Y 轴显示交通流量。X 轴显示时间。而添加一个子聚合，我们还可以看到每小时排名前三的地铁站。

![](http://www.elasticsearch.org/guide/en/kibana/current/images/TFL-CommuteHistogram.jpg)

你可以保存并分析可视化结果，然后合并到仪表板上以便对比分析。比如说，我们可以创建一个展示多个伦敦交通数据的仪表板：

![](http://www.elasticsearch.org/guide/en/kibana/current/images/TFL-Dashboard.jpg)

更多关于创建和分享可视化和仪表板的内容，请阅读 [Visualize](./visualize.md) 和 [Dashboard](./dashboard.md) 章节。
