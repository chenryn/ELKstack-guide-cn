# timelion 介绍

Elasticsearch 2.0 开始提供了一个崭新的 pipeline aggregation 特性，但是 Kibana 并没有立刻跟进这方面的意思，相反，Elastic 公司推出了另一个实验室产品：[Timelion](https://github.com/elastic/timelion)。最后在 5.0 版中，timelion 成为 Kibana 5 默认分发的一个插件。

timelion 的用法在[官博](https://www.elastic.co/blog/timelion-timeline)里已经有介绍。尤其是最近两篇如何用 timelion 实现异常告警的[文章](https://www.elastic.co/blog/implementing-a-statistical-anomaly-detector-part-2)，更是从 ES 的 pipeline aggregation 细节和场景一路讲到 timelion 具体操作，我这里几乎没有再重新讲一遍 timelion 操作入门的必要了。不过，官方却一直没有列出来 timelion 支持的请求语法的文档，而是在页面上通过点击图标的方式下拉帮助。

![](http://logstash.es/images/timelion.png)

*timelion 页面设计上，更接近 Kibana3 而不是 Kibana4。比如 panel 分布是通过设置几行几列的数目来固化的；query 框是唯一的，要修改哪个 panel 的 query，鼠标点选一下 panel，query 就自动切换成这个 panel 的了。*

为了方便大家在上手之前了解 timelion 能做到什么，今天特意把 timelion 的请求语法所支持的函数分为几类，罗列如下：

## 可视化效果类

* `.bars($width)`: 用柱状图展示数组
* `.lines($width, $fill, $show, $steps)`: 用折线图展示数组
* `.points()`: 用散点图展示数组
* `.color("#c6c6c6")`: 改变颜色
* `.hide()`: 隐藏该数组
* `.label("change from %s")`: 标签
* `.legend($position, $column)`: 图例位置
* `.yaxis($yaxis_number, $min, $max, $position)`: 设置 Y 轴属性，.yaxis(2) 表示第二根 Y 轴

## 数据运算类

* `.abs()`: 对整个数组元素求绝对值
* `.precision($number)`: 浮点数精度
* `.testcast($count, $alpha, $beta, $gamma)`: holt-winters 预测
* `.cusum($base)`: 数组元素之和，再加上 $base
* `.derivative()`: 对数组求导数
* `.divide($divisor)`: 数组元素除法
* `.multiply($multiplier)`: 数组元素乘法
* `.subtract($term)`: 数组元素减法
* `.sum($term)`: 数组元素加法
* `.add()`: 同 .sum()
* `.plus()`: 同 .sum()
* `.first()`: 返回第一个元素
* `.movingaverage($window)`: 用指定的窗口大小计算移动平均值
* `.mvavg()`: `.movingaverage()` 的简写
* `.movingstd($window)`: 用指定的窗口大小计算移动标准差
* `.mvstd()`: `.movingstd()` 的简写

## 数据源设定类

* `.elasticsearch()`: 从 ES 读取数据
* `.es(q="querystring", metric="cardinality:uid", index="logstash-*", offset="-1d")`: .elasticsearch() 的简写
* `.graphite(metric="path.to.*.data", offset="-1d")`: 从 graphite 读取数据
* `.quandl()`: 从 quandl.com 读取 quandl 码
* `.worldbank_indicators()`: 从 worldbank.org 读取国家数据
* `.wbi()`: `.worldbank_indicators()` 的简写
* `.worldbank()`: 从 worldbank.org 读取数据
* `.wb()`: `.worldbanck()` 的简写

以上所有函数，都在 [series_functions](https://github.com/elastic/timelion/tree/master/series_functions) 目录下实现，每个 js 文件实现一个 `TimelionFunction` 功能。



