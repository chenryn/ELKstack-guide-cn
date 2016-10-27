# 饼图

饼图的分片大小通过 *metrics* 聚合定义。这个维度可以支持以下聚合：

* Count
    [count](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-valuecount-aggregation.html) 聚合返回选中索引模式中元素的原始计数。
* Sum
    [sum](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-sum-aggregation.html) 聚合返回一个数值字段的总和。从下拉菜单选择一个字段。
* Unique Count
    [cardinality](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cardinality-aggregation.html) 聚合返回一个字段的去重数据值。从下拉菜单选择一个字段。

*buckets* 聚合指明从你的数据集中将要检索什么信息。

在你选定一个 buckets 聚合之前，先指定你是要切割单个图的分片，还是切割成多个图形。多图切分必须在其他任何聚合之前要运行。如果你切分图形，你可以选择切分结果是展示成行还是列的形式，点击 **Rows | Columns** 选择器即可。

你可以为你的饼图指定以下任意 bucket 聚合：

* Date Histogram
    [date histogram](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-datehistogram-aggregation.html) 基于数值字段创建，由时间组织起来。你可以指定时间片的间隔，单位包括秒，分，小时，天，星期，月，年。
* Histogram
    标准 [histogram](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-histogram-aggregation.html) 基于数值字段创建。为这个字段指定一个整数间隔。勾选 **Show empty buckets** 让直方图中包含空的间隔。
* Range
    通过 [range](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-range-aggregation.html) 聚合。你可以为一个数值字段指定一系列区间。点击 **Add Range** 添加一堆区间端点。点击红色 **(x)** 符号移除一个区间。
* Date Range
    [date range](http://www.elastic.co/guide/en/elasticsearch/reference/current//search-aggregations-bucket-daterange-aggregation.html) 聚合计算你指定的时间区间内的值。你可以使用 [date math](http://www.elastic.co/guide/en/elasticsearch/reference/current//mapping-date-format.html#date-math) 表达式指定区间。点击 **Add Range** 添加新的区间端点。点击红色 **(/)** 符号移除区间。
* IPv4 Range
    [IPv4 range](http://www.elastic.co/guide/en/elasticsearch/reference/current//search-aggregations-bucket-iprange-aggregation.html) 聚合用来指定 IPv4 地址的区间。点击 **Add Range** 添加新的区间端点。点击红色 **(/)** 符号移除区间。
* Terms
    [terms](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html) 聚合允许你指定展示一个字段的首尾几个元素，排序方式可以是计数或者其他自定义的metric。
* Filters
    你可以为数据指定一组 [filters](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-filters-aggregation.html)。你可以用 query string，也可以用 JSON 格式来指定过滤器，就像在 Discover 页的搜索栏里一样。点击 **Add Filter** 添加下一个过滤器。
* Significant Terms
    展示实验性的 [significant terms](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-significantterms-aggregation.html) 聚合的结果。

一旦你定义好了一个 X 轴聚合。你可以继续定义子聚合来完善可视化效果。点击 **+ Add Sub Aggregation** 添加子聚合，然后选择 **Split Area** 或者 **Split Chart**，然后从类型菜单中选择一个子聚合。

当一个图形中定义了多个聚合，你可以使用聚合类型右侧的上下箭头来改变聚合的优先级。

你可以点击 **Advanced** 链接显示更多有关聚合的自定义参数：

* Exclude Pattern
    指定一个从结果集中排除掉的模式。
* Exclude Pattern Flags
    排除模式的 Java flags 标准集。
* Include Pattern
    指定一个从结果集中要包含的模式。
* Include Pattern Flags
    包含模式的 Java flags 标准集。
* JSON Input
    一个用来添加 JSON 格式属性的文本框，内容会合并进聚合的定义中，格式如下例：

```
{ "script" : "doc['grade'].value * 1.2" }
```

### 注意

> Elasticsearch 1.4.3 及以后版本，这个函数需要你开启 [dynamic Groovy scripting](http://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)。

这些参数是否可用，依赖于你选择的聚合函数。

选择 **view options** 更改表格中如下方面：

* Donut
    Display the chart as a sliced ring instead of a sliced pie.
* Show Tooltip
    勾选该项显示工具栏。
* Show Legend
    勾选该项在图形右侧显示图例。
