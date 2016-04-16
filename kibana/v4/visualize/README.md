# 各 Visualize 功能

Visualize 标签页用来设计可视化。你可以保存可视化，以后再用，或者加载合并到 *dashboard* 里。一个可视化可以基于以下几种数据源类型：

* 一个新的交互式搜索
* 一个已保存的搜索
* 一个已保存的可视化

可视化是基于 Elasticsearch 1.0 引入的[聚合(aggregation)](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations.html) 特性。

## 创建一个新可视化

要开始一个 **New Visualization** 向导，点击页面左上角的 **Visualize** 标签。如果你已经在创建一个可视化了。你可以在搜索栏的右侧工具栏里点击 **New Visualization** 按钮 ![New Document button](http://www.elasticsearch.org/guide/en/kibana/current/images/K4NewDocument.png) 向导会引导你继续以下几步：

### 第 1 步: 选择可视化类型

在 **New Visualization** 向导起始页可以选择以下一个可视化类型：

|------------------|:-------------------------------------------------------------------------|
|Area chart        |用区块图来可视化多个不同序列的总体贡献。                                  |
|Data table        |用数据表来显示聚合的原始数据。其他可视化可以通过点击底部的方式显示数据表。|
|Line chart        |用折线图来比较不同序列。                                                  |
|Markdown widget   |用 Markdown 显示自定义格式的信息或和你仪表盘有关的用法说明。              |
|Metric            |用指标可视化在你仪表盘上显示单个数字。                                    |
|Pie chart         |用饼图来显示每个来源对总体的贡献。                                        |
|Tile map          |用瓦片地图将聚合结果和经纬度联系起来。                                    |
|Vertical bar chart|用垂直条形图作为一个通用图形。                                            |

你也可以加载一个你之前创建好保存下来的可视化。已存可视化选择器包括一个文本框用来过滤可视化名称，以及一个指向 **对象编辑器(Object Editor)** 的链接，可以通过 **Settings > Edit Saved Objects** 来管理已存的可视化。

如果你的新可视化是一个 **Markdown** 挂件，选择这个类型会带你到一个文本内容框，你可以在框内输入打算显示在挂件里的文本。其他的可视化类型，选择后都会转到数据源选择。

### 第 2 步: 选择数据源

你可以选择新建或者读取一个已保存的搜索，作为你可视化的数据源。搜索是和一个或者一系列索引相关联的。如果你选择了在一个配置了多个索引的系统上开始你的*新搜索*，从可视化编辑器的下拉菜单里选择一个索引模式。

当你从一个已保存的搜索开始创建并保存好了可视化，这个搜索就绑定在这个可视化上。如果你修改了搜索，对应的可视化也会自动更新。

### 第 3 步: 可视化编辑器

The visualization editor enables you to configure and edit visualizations. The visualization editor has the following main elements:
可视化编辑器用来配置编辑可视化。它有下面几个主要元素：

1. 工具栏(Toolbar)
2. 聚合构建器(Aggregation Builder)
3. 预览画布(Preview Canvas)

![images/VizEditor.jpg](http://www.elasticsearch.org/guide/en/kibana/current/images/VizEditor.jpg)

#### 工具栏

工具栏上有一个用户交互式数据搜索的搜索框，用来保存和加载可视化。因为可视化是基于保存好的搜索，搜索栏会变成灰色。要编辑搜索，双击搜索框，用编辑后的版本替换已保存搜索。

搜索框右侧的工具栏有一系列按钮，用于创建新可视化，保存当前可视化，加载一个已有可视化，分享或内嵌可视化，和刷新当前可视化的数据。

#### 聚合构建器

用页面左侧的聚合构建器配置你的可视化要用的 [metric](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations.html#_metrics_aggregations) 和 [bucket](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations.html#_bucket_aggregations) 聚合。桶(Buckets) 的效果类似于 SQL `GROUP BY` 语句。想更详细的了解聚合，阅读 [Elasticsearch aggregations reference](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations.html)。

在条带图或者折线图可视化里，用 *metrics* 做 Y 轴，然后 *buckets* 做 X 轴，条带颜色，以及行/列的区分。在饼图里，*metrics* 用来做分片的大小，*buckets* 做分片的数量。

为你的可视化 Y 轴选一个 metric 聚合，包括 [count](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations-metrics-valuecount-aggregation.html), [average](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations-metrics-avg-aggregation.html), [sum](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations-metrics-sum-aggregation.html), [min](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations-metrics-min-aggregation.html), [max](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations-metrics-max-aggregation.html), or [cardinality](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cardinality-aggregation.html) (unique count). 为你的可视化 X 轴，条带颜色，以及行/列的区分选一个 bucket 聚合，常见的有 date histogram, range, terms, filters, 和 significant terms。

你可以设置 buckets 执行的顺序。在 Elasticsearch 里，第一个聚合决定了后续聚合的数据集。下面例子演示一个网页访问量前五名的文件后缀名统计的时间条带图。

要看所有相同后缀名的，设置顺序如下：

1. `Color`: 后缀名的 Terms 聚合
2. `X-Axis`: `@timestamp` 的时间条带图

Elasticsearch 收集记录，算出前 5 名后缀名，然后为每个后缀名创建一个时间条带图。

要看每个小时的前 5 名后缀名情况，设置顺序如下：

1. `X-Axis`: `@timestamp` 的时间条带图( 1 小时间隔)
2. `Color`: 后缀名的 Terms 聚合

这次，Elasticsearch 会从所有记录里创建一个时间条带图，然后在每个桶内，分组(本例中就是一个小时的间隔)计算出前 5 名的后缀名。

> 记住，每个后续的桶，都是从前一个的桶里分割数据。

要在*预览画布(preview canvas)*上渲染可视化，点击聚合构建器底部的 **Apply** 按钮。

#### 预览画布(canvas)

预览 canvas 上显示你定义在聚合构建器里的可视化的预览效果。要刷新可视化预览，点击工具栏里的 **Refresh** 按钮 ![Refresh button](http://www.elasticsearch.org/guide/en/kibana/current/images/K4Refresh.png)。
