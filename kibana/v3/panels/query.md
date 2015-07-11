# query

query 面板和 filter 面板都是特殊类型的面板，在 dashboard 上有且仅有一个。不能删除不能添加。

query 和 filter 的普通样式和基本操作，在之前[ Query 和 Filtering](../dashboard/queries-and-filters.md) 章节已经讲述过。这里，额外讲一下一些高阶功能。

## 请求类型

query 搜索框支持三种请求类型：

* lucene

这也是默认的类型，使用要点就是请求语法。语法说明在之前 [Elasticsearch 一章](../../elasticsearch/api/search.md)已经讲过。

* regex

* topN

topN 是一个方便大家进行多项对比搜索的功能。其配置界面如下：

![](../img/query-topn.png)

**注意 topN 后颜色选择器的小圆点变成了齿轮状！**

其运行实质，是先根据你填写的 field 和 size，发起一次 termsFacet 查询，获取 topN 的 term 结果；然后拿着这个列表，逐一发起附加了 term 条件的其他请求(比如绑定在 histogram 面板就是 `date_histogram` 请求，stats 面板就是 `termStats` 请求)，也就获得了 topN 结果。

![](../img/topn-histogram.png)

*小贴士：如果 ES 响应较慢的时候，你甚至可以很明显的看到 histogram 面板上的多条曲线是一条一条出来数据绘制的。*

## 别名

query 还可以设置别名(alias)。默认没有别名的时候，各 panel 上显示对应 query 时，会使用具体的 query 语句。在查询比较复杂的时候，不便观看。而设置别名后，pinned queries，panel 图例等处，都会只显示设置好的别名，而不再显示复杂的查询语句，这样一目了然。
