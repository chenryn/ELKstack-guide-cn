# Kibana4

Kibana4 是 Elastic.co 一次崭新的重构产品。在操作界面上，有一定程度的对 Splunk 的模仿。为了更直观的体现 Kibana4 跟 Kibana3 的不同，先让我们看看 Kibana4 要怎么用来探索和展示数据。

配置索引模式后，默认打开 Kibana4 会出现在一个叫做 Discover 的标签页，在这个类似 Kibana3 的 logstash dashboard 的页面上，我们可以提交搜索请求，过滤结果，然后检查返回的文档里的数据。如下图示：

![](https://www.elastic.co/guide/en/kibana/current/images/tutorial-discover.png)

默认情况下，Discover 页会显示匹配搜索条件的前 500 个文档，页面下拉到底部，会自动加载后续数据。你可以修改时间过滤器，拖拽直方图下钻数据，查看部分文档的细节。Discover 页上如何探索数据，详细说明见 [Discover 功能](./discover.md)。

Visualization 标签页用来为你的搜索结果构造可视化。每个可视化都是跟一个搜索关联着的。比如，我们可以基于前面那个搜索创建一个展示区间计数的饼图。而添加一个子聚合，我们还可以看到每个区间内排名前五的年龄。

![](https://www.elastic.co/guide/en/kibana/current/images/tutorial-visualize-pie-3.png)

还可以保存并分析可视化结果，然后合并到 Dashboard 标签页上以便对比分析。比如说，我们可以创建一个展示多个可视化数据的仪表板：

![](https://www.elastic.co/guide/en/kibana/current/images/tutorial-dashboard.png)

更多关于创建和分享可视化和仪表板的内容，请阅读 [Visualize](./visualize.md) 和 [Dashboard](./dashboard.md) 章节。
