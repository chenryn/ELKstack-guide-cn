# 瓦片地图

瓦片地图显示一个由圆圈覆盖着的地理区域。这些圆圈则是由你指定的 buckets 控制的。

瓦片地图的默认 metrics 聚合是 **Count** 聚合。你可以选择下列聚合中的任意一个作为 metrics 聚合：

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

*buckets* 聚合指明从你的数据集中将要检索什么信息。

在你选择 buckets 聚合前，指定你是打算分割图形，还是在单个图形上显示 buckets 为 **Geo Coordinates**。多图切割的聚合必须在最先运行。

瓦片地图使用 **Geohash** 聚合作为他们的初始化聚合。从下拉菜单中选择一个坐标字段。**Precision** 滑动条设置圆圈在地图上显示的颗粒度大小。阅读 [geohash grid](http://www.elastic.co/guide/en/elasticsearch/reference/current//search-aggregations-bucket-geohashgrid-aggregation.html#_cell_dimensions_at_the_equator) 聚合的文档，了解每个精度级别的区域细节。Kibana 4.1 目前支持的最大 geohash 长度为 7。

### 注意

更高的精度意味着同时消耗了浏览器和 ES 集群更多的内存。

一旦你定义好了一个 X 轴聚合。你可以继续定义子聚合来完善可视化效果。点击 **+ Add Sub Aggregation** 添加子聚合，然后选择 **Split Area** 或者 **Split Chart**，然后从类型菜单中选择一个子聚合。

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
* Geohash
    The [geohash](http://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-geohashgrid-aggregation.html) aggregation displays points based on the geohash coordinates.

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

选择 **Options** 改变表格的如下方面：

### Map type

从下拉框选择下面一个选项。

* Shaded Circle Markers
    根据 metric 聚合的值显示不同的颜色。
* Scaled Circle Markers
    根据 metric 聚合的值显示不同的大小。
* Shaded Geohash Grid
    用矩形替换默认的圆形显示 geohash，根据 metric 聚合的值显示不同的颜色。
* Heatmap
    热力图可以模糊化圆标而且层叠显示颜色。热力图本身还有如下选项可用：
  * Radius: 设置单个热力图点的大小。
  * Blur: 设置热力图点的模糊度。
  * Maximum zoom: Kibana 的 Tilemap 支持 18 级缩放。该选项设置热力图最大强度下的最高缩放级别。
  * Minimum opacity: 设置数据点的不透明截止位置。
  * Show Tooltip: 勾选该项，让鼠标放在数据点上时显示该点的数据。

### Desaturate map tiles

淡化地图颜色，凸显标记的清晰度。

### WMS compliant map server

勾选该项，可以配置使用符合 Web Map Service (WMS) 标准的其他第三方地图服务。需要指定一下参数：

* WMS url: WMS 地图服务的 URL；
* WMS layers: 用于可视化的图层列表，逗号分隔。每个地图服务商都会提供自己的图层。
* WMS version: 该服务商采用的 WMS 版本。
* WMS format: 该服务商使用的图片格式。通常来说是 image/png 或 image/jpeg。
* WMS attribution: 可选项。用户自定义字符串，用来显示在图表右下角的属性说明。
* WMS styles: 逗号分隔的风格列表。每个地图服务商都会提供自己的风格选项。

更新选项后，点击绿色 **Apply changes** 按钮更新你的可视化界面，或者灰色 **Discard changes** 按钮保持原状。

## Navigating the Map

可视化地图就绪后，你可以通过一下方式探索地图：

* 在地图任意位置点击并按住鼠标后，拖动即可转移地图中心。按住鼠标左键拖拽绘制方框则可以放大选定区域。
* 点击 **Zoom In/Out** ![images/viz-zoom.png](https://www.elastic.co/guide/en/kibana/current/images/viz-zoom.png) 按钮手动修改缩放级别。
* 点击 **Fit Data Bounds** ![images/viz-fit-bounds.png](https://www.elastic.co/guide/en/kibana/current/images/viz-fit-bounds.png) 按钮让地图自动聚焦到至少有一个数据点的地区。
* 点击 **Latitude/Longitude Filter** ![images/viz-lat-long-filter.png](https://www.elastic.co/guide/en/kibana/current/images/viz-lat-long-filter.png) 按钮，然后在地图上拖拽绘制一个方框，自动生成这个框范围的经纬度过滤器。
