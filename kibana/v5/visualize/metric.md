# Metric

metric 可视化为你选择的聚合显示一个单独的数字：

* Count
    [count](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-valuecount-aggregation.html) 聚合返回选中索引模式中元素的原始计数。
* Average
    这个聚合返回一个数值字段的 [average](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-avg-aggregation.html) 。从下拉菜单选择一个字段。
* Sum
    [sum](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-sum-aggregation.html) 聚合返回一个数值字段的总和。从下拉菜单选择一个字段。
* Min
    [min](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-min-aggregation.html) 聚合返回一个数值字段的最小值。从下拉菜单选择一个字段。
* Max
    [max](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-max-aggregation.html) 聚合返回一个数值字段的最大值。从下拉菜单选择一个字段。
* Unique Count
    [cardinality](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cardinality-aggregation.html) 聚合返回一个字段的去重数据值。从下拉菜单选择一个字段。
* Standard Deviation
    [extended stats](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-extendedstats-aggregation.html) 聚合返回一个数值字段数据的标准差。从下拉菜单选择一个字段。
* Percentile
    [percentile](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-percentile-rank-aggregation.html) 聚合返回一个数值字段中值的百分比分布。从下拉菜单选择一个字段，然后在 **Percentiles** 框内指定范围。点击 **X** 移除一个百分比框，点击 **+ Add Percent** 添加一个百分比框。
* Percentile Rank
    [percentile ranks](http://www.elastic.co/guide/en/elasticsearch/reference/current//search-aggregations-metrics-percentile-rank-aggregation.html) 聚合返回一个数值字段中你指定值的百分位排名。从下拉菜单选择一个字段，然后在 **Values** 框内指定一到多个百分位排名值。点击 **X** 移除一个百分比框，点击 **+Add** 添加一个数值框。

你可以点击 **+ Add Aggregation** 按键添加一个聚合。你可以点击 **Advanced** 链接显示更多有关聚合的自定义参数：

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

点击 **view options** 修改显示 metric 的字体大小。

