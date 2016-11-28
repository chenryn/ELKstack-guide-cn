# discover 功能

Discover 标签页用于交互式探索你的数据。你可以访问到匹配得上你选择的索引模式的每个索引的每条记录。你可以提交搜索请求，过滤搜索结果，然后查看文档数据。你还可以看到匹配搜索请求的文档总数，获取字段值的统计情况。如果索引模式配置了时间字段，文档的时序分布情况会在页面顶部以柱状图的形式展示出来。

![](http://www.elasticsearch.org/guide/en/kibana/current/images/Discover-Start-Annotated.jpg)

## 设置时间过滤器

时间过滤器(Time Filter)限制搜索结果在一个特定的时间周期内。如果你的索引包含的是时序诗句，而且你为所选的索引模式配置了时间字段，那么就就可以设置时间过滤器。

默认的时间过滤器设置为最近 15 分钟。你可以用页面顶部的时间选择器(Time Picker)来修改时间过滤器，或者选择一个特定的时间间隔，或者直方图的时间范围。

要用时间选择器来修改时间过滤器：

1. 点击菜单栏右上角显示的 Time Filter 打开时间选择器。
2. 快速过滤，直接选择一个短链接即可。
3. 要指定相对时间过滤，点击 Relative 然后输入一个相对的开始时间。可以是任意数字的秒、分、小时、天、月甚至年之前。
4. 要指定绝对时间过滤，点击 Absolute 然后在 From 框内输入开始日期，To 框内输入结束日期。
5. 点击时间选择器底部的箭头隐藏选择器。

要从柱状图上设置时间过滤器，有以下几种方式：

* 想要放大那个时间间隔，点击对应的柱体。
* 单击并拖拽一个时间区域。注意需要等到光标变成加号，才意味着这是一个有效的起始点。

你可以用浏览器的后退键来回退你的操作。

## 搜索数据

在 Discover 页提交一个搜索，你就可以搜索匹配当前索引模式的索引数据了。你可以直接输入简单的请求字符串，也就是用 Lucene [query syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html)，也可以用完整的基于 JSON 的 [Elasticsearch Query DSL](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl.html)。

当你提交搜索的时候，直方图，文档表格，字段列表，都会自动反映成搜索的结果。hits(匹配的文档)总数会在直方图的右上角显示。文档表格显示前 500 个匹配文档。默认的，文档倒序排列，最新的文档最先显示。你可以通过点击时间列的头部来反转排序。事实上，所有建了索引的字段，都可以用来排序，稍后会详细说明。

要搜索你的数据：

1. 在搜索框内输入请求字符串：
  * 简单的文本搜索，直接输入文本字符串。比如，如果你在搜索网站服务器日志，你可以输入 `safari` 来搜索各字段中的 `safari` 单词。
  * 要搜索特定字段中的值，则在值前加上字段名。比如，你可以输入 `status:200` 来限制搜索结果都是在 `status` 字段里有 `200` 内容。
  * 要搜索一个值的范围，你可以用范围查询语法，`[START_VALUE TO END_VALUE]`。比如，要查找 4xx 的状态码，你可以输入 `status:[400 TO 499]`。
  * 要指定更复杂的搜索标准，你可以用布尔操作符 `AND`, `OR`, 和 `NOT`。比如，要查找 4xx 的状态码，还是 `php` 或 `html` 结尾的数据，你可以输入 `status:[400 TO 499] AND (extension:php OR extension:html)`。

> 这些例子都用了 Lucene query syntax。你也可以提交 Elasticsearch Query DSL 式的请求。更多示例，参见之前 [Elasticsearch 章节](../../elasticsearch/api/search.md)。

2. 点击回车键，或者点击 `Search` 按钮提交你的搜索请求。

## 开始一个新的搜索

要清除当前搜索或开始一个新搜索，点击 Discover 工具栏的 New Search 按钮。

## 保存搜索

你可以在 Discover 页加载已保存的搜索，也可以用作 [visualizations](./visualize.md) 的基础。保存一个搜索，意味着同时保存下了搜索请求字符串和当前选择的索引模式。

要保存当前搜索：

1. 点击 Discover 工具栏的 `Save Search` 按钮。
2. 输入一个名称，点击 `Save`。

## 加载一个已存搜索

要加载一个已保存的搜索：

1. 点击 Discover 工具栏的 `Load Search` 按钮。
2. 选择你要加载的搜索。

如果已保存的搜索关联到跟你当前选择的索引模式不一样的其他索引上，加载这个搜索也会切换当前的已选索引模式。

## 改变你搜索的索引

当你提交一个搜索请求，匹配当前的已选索引模式的索引都会被搜索。当前模式模式会显示在搜索栏下方。要改变搜索的索引，需要选择另外的模式模式。

要选择另外的索引模式：

1. 点击 Discover 工具栏的 `Settings` 按钮。
2. 从索引模式列表中选取你打算采用的模式。

关于索引模式的更多细节，请阅读稍后 [Setting 功能小节](./settings.md)。

## 自动刷新页面

亦可以配置一个刷新间隔来自动刷新 Discover 页面的最新索引数据。这会定期重新提交一次搜索请求。

设置刷新间隔后，会显示在菜单栏时间过滤器的左边。

要设置刷新间隔：

1. 点击菜单栏右上角的 `Time Picker` ![Time Picker](https://www.elastic.co/guide/en/kibana/current/images/time-picker.jpg)。
2. 点击 `Auto refresh` 标签。
3. 从列表中选择一个刷新间隔。

![images/autorefresh-intervals.png](https://www.elastic.co/guide/en/kibana/current/images/autorefresh-intervals.png)

开启自动刷新后，Kibana 的顶部栏会出现一个暂停按钮和自动刷新的间隔，点击 **Pause** 按钮可以暂停自动刷新。

## 按字段过滤

你可以过滤搜索结果，只显示在某字段中包含了特定值的文档。也可以创建反向过滤器，排除掉包含特定字段值的文档。

你可以从字段列表或者文档表格里添加过滤器。当你添加好一个过滤器后，它会显示在搜索请求下方的过滤栏里。从过滤栏里你可以编辑或者关闭一个过滤器，转换过滤器(从正向改成反向，反之亦然)，切换过滤器开关，或者完全移除掉它。

要从字段列表添加过滤器：

1. 点击你想要过滤的字段名。会显示这个字段的前 5 名数据。每个数据的右侧，有两个小按钮 —— 一个用来添加常规(正向)过滤器，一个用来添加反向过滤器。
2. 要添加正向过滤器，点击 `Positive Filter` 按钮 ![Positive Filter Button](http://www.elasticsearch.org/guide/en/kibana/current/images/PositiveFilter.jpg)。这个会过滤掉在本字段不包含这个数据的文档。
3. 要添加反向过滤器，点击 `Negative Filter` 按钮 ![Negative Filter Button](http://www.elasticsearch.org/guide/en/kibana/current/images/NegativeFilter.jpg)。这个会过滤掉在本字段包含这个数据的文档。

要从文档表格添加过滤器：

1. 点击表格第一列(通常都是时间)文档内容左侧的 `Expand` 按钮 ![Expand Button](http://www.elasticsearch.org/guide/en/kibana/current/images/ExpandButton.jpg) 展开文档表格中的文档。每个字段名的右侧，有两个小按钮 —— 一个用来添加常规(正向)过滤器，一个用来添加反向过滤器。
2. 要添加正向过滤器，点击 `Positive Filter` 按钮 ![Positive Filter Button](http://www.elasticsearch.org/guide/en/kibana/current/images/PositiveFilter.jpg)。这个会过滤掉在本字段不包含这个数据的文档。
3. 要添加反向过滤器，点击 `Negative Filter` 按钮 ![Negative Filter Button](http://www.elasticsearch.org/guide/en/kibana/current/images/NegativeFilter.jpg)。这个会过滤掉在本字段包含这个数据的文档。

## 过滤器(Filter)的协同工作方式

在 Kibana 的任意页面创建过滤器后，就会在搜索输入框的下方，出现椭圆形的过滤条件。

鼠标移动到过滤条件上，会显示下面几个图标：

![images/filter-allbuttons.png](https://www.elastic.co/guide/en/kibana/current/images/filter-allbuttons.png)

* 过滤器开关 ![images/filter-enable.png](https://www.elastic.co/guide/en/kibana/current/images/filter-enable.png)
  点击这个图标，可以在不移除过滤器的情况下关闭过滤条件。再次点击则重新打开。被禁用的过滤器是条纹状的灰色，要求包含(相当于 Kibana3 里的 must)的过滤条件显示为绿色，要求排除(相当于 Kibana3 里的 mustNot)的过滤条件显示为红色。
* 过滤器图钉 ![images/filter-pin.png](https://www.elastic.co/guide/en/kibana/current/images/filter-pin.png)
  点击这个图标*钉住*过滤器。被钉住的过滤器，可以横贯 Kibana 各个标签生效。比如在 *Visualize* 标签页钉住一个过滤器，然后切换到 *Discover* 或者 *Dashboard* 标签页，过滤器依然还在。注意：如果你钉住了过滤器，然后发现检索结果为空，注意查看当前标签页的索引模式是不是跟过滤器匹配。
* 过滤器反转 ![images/filter-toggle.png](https://www.elastic.co/guide/en/kibana/current/images/filter-toggle.png)
  点击这个图标*反转*过滤器。默认情况下，过滤器都是包含型，显示为绿色，只有匹配过滤条件的结果才会显示。反转成排除型过滤器后，显示的是*不*匹配过滤器的检索项，显示为红色。
* 移除过滤器 ![images/filter-delete.png](https://www.elastic.co/guide/en/kibana/current/images/filter-delete.png)
  点击这个图标删除过滤器。
* 自定义过滤器 ![](https://www.elastic.co/guide/en/kibana/current/images/filter-custom.png)
  点击这个图标会打开一个文本编辑框。编辑框内可以修改 JSON 形式的过滤器内容，并起一个 alias 别名：
  ![](https://www.elastic.co/guide/en/kibana/current/images/filter-custom-json.png)
  JSON 中可以灵活应用 bool query 组合各种 `should`、`must`、`must_not` 条件。一个用 `should` 表达的 OR 关系过滤如下:
```
{
    "bool": {
        "should": [
            {
                "term": {
                    "geoip.country_name.raw": "Canada"
                }
            },
            {
                "term": {
                    "geoip.country_name.raw": "China"
                }
            }
        ]
    }
}
```

想要对当前页所有过滤器统一执行上面的某个操作，点击 **Actions** 按钮，然后再执行操作即可。

## 查看文档数据

当你提交一个搜索请求，最近的 500 个搜索结果会显示在文档表格里。你可以在 **Advanced Settings** 里通过 `discover:sampleSize` 属性配置表格里具体的文档数量。默认的，表格会显示当前选择的索引模式中定义的时间字段内容(转换成本地时区)以及 `_source` 文档。你可以从字段列表*添加字段到文档表格*。还可以用表格里包含的任意已建索引的字段来*排序列出的文档*。

要查看一个文档的字段数据，点击表格第一列(通常都是时间)文档内容左侧的 `Expand` 按钮 ![Expand Button](http://www.elasticsearch.org/guide/en/kibana/current/images/ExpandButton.jpg)。Kibana 从 Elasticsearch 读取数据然后在表格中显示文档字段。这个表格每行是一个字段的名字、过滤器按钮和字段的值。

![](https://www.elastic.co/guide/en/kibana/current/images/Expanded-Document.png)

1. 要查看原始 JSON 文档(格式美化过的)，点击 **JSON** 标签。
2. 要在单独的页面上查看文档内容，点击链接。你可以添加书签或者分享这个链接，以直接访问这条特定文档。
3. 收回文档细节，点击 **Collapse** 按钮 ![Collapse Button](http://www.elasticsearch.org/guide/en/kibana/current/images/CollapseButton.jpg)。
4. To toggle a particular field’s column in the Documents table, click the ![Add Column](https://www.elastic.co/guide/en/kibana/current/images/add-column-button.png) **Toggle column in table** button.

## 文档列表排序

你可以用任意已建索引的字段排序文档表格中的数据。如果当前索引模式配置了时间字段，默认会使用该字段倒序排列文档。

要改变排序方式：

* 点击想要用来排序的字段名。能用来排序的字段在字段名右侧都有一个排序按钮。再次点击字段名，就会反向调整排序方式。

## 给文档表格添加字段列
默认的，文档表格会显示当前选择的索引模式中定义的时间字段内容(转换成本地时区)以及 `_source` 文档。你可以从字段列表添加字段到文档表格。

要添加字段列到文档表格：

1. 从字段列表中添加一列，移动鼠标到字段列表的字段上，点击它的 `add` 按钮
2. 从文档的字段数据添加一列，展开对应的文档后点击字段的![](https://www.elastic.co/guide/en/kibana/current/images/add-column-button.png)`Toggle column in table`按钮

添加的字段会替换掉文档表格里的 `_source` 列。同时还会显示在字段列表顶部的 `Selected Fields` 区域里。

要重排表格中的字段列，移动鼠标到你要移动的列顶部，点击移动过按钮。

![](http://www.elasticsearch.org/guide/en/kibana/current/images/Discover-MoveColumn.jpg)

## 从文档表格删除字段列

要从文档表格删除字段列：

1. 移动鼠标到字段列表的 `Selected Fields` 区域里你想要移除的字段上，然后点击它的 `remove` 按钮 ![Remove Field Button](http://www.elasticsearch.org/guide/en/kibana/current/images/RemoveFieldButton.jpg)。
2. 重复操作直到你移除完所有你想移除的字段。

## 查看字段数据统计

从字段列表，你可以看到文档表格里有多少数据包含了这个字段，排名前 5 的值是什么，以及包含各个值的文档的占比。

要查看字段数据统计，在字段列表中点击对应的字段名即可。

![](https://www.elastic.co/guide/en/kibana/current/images/filter-field.jpg)

> 要基于这个字段创建可视化，点击字段统计下方的 Visualize 按钮。
