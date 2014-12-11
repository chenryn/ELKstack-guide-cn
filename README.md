# 简介

Kibana 是一个使用 Apache 开源协议，基于浏览器的 Elasticsearch 分析和搜索仪表板。Kibana 非常容易安装和使用。整个项目都是用 HTML 和 Javascript 写的，所以 Kibana 不需要任何服务器端组件，一个纯文本发布服务器就够了。Kibana 和 Elasticsearch 一样，力争成为极易上手，但同样灵活而强大的软件。

## 注释

本书原始内容来源[Elasticsearch 官方指南 Kibana 部分](http://www.elasticsearch.org/guide/en/kibana/current/index.html)，并对 panel 部分加以截图注释。在有时间的前提下，将会添加更多关于 kibana 源码解析和第三方 panel 的介绍。

## 译作者的话

Kibana 因其丰富的图表类型和漂亮的前端界面，被很多人理解成一个统计工具。而我个人认为，ELK 这一套体系，不应该和 Hadoop 体系同质化。定期的离线报表，不是 Elasticsearch 专长所在(多花费分词、打分这些步骤在高负载压力环境上太奢侈了)，也不应该由 Kibana 来完成(每次刷新都是重新计算)。Kibana 的使用场景，应该集中在两方面：

* 实时监控

  通过 histogram 面板，配合不同条件的多个 queries 可以对一个事件走很多个维度组合出不同的时间序列走势。时间序列数据是最常见的监控报警了。

* 问题分析

  通过 Kibana 的交互式界面可以很快的将异常时间或者事件范围缩小到秒级别或者个位数。期望一个完美的系统可以给你自动找到问题原因并且解决是不现实的，能够让你三两下就从 TB 级的数据里看到关键数据以便做出判断就很棒了。这时候，一些非 histogram 的其他面板还可能会体现出你意想不到的价值。全局状态下看似很普通的结果，可能在你锁定某个范围的时候发生剧烈的反方向的变化，这时候你就能从这个维度去重点排查。而表格面板则最直观的显示出你最关心的字段，加上排序等功能。入库前字段切分好，对于排错分析真的至关重要。

*以上是我在和同事就 ES 跟 Hadoop 对比的谈话中形成的思路。特此留笔。2014 年 8 月 28 日*

---------------

关于 elk 的用途，我想还可以参照其对应的商业产品 splunk 的场景：

> 使用 Splunk 的意义在于使信息收集和处理智能化。而其操作智能化表现在：
>
> 1. 搜索，通过下钻数据排查问题，通过分析根本原因来解决问题；
> 2. 实时可见性，可以将对系统的检测和警报结合在一起，便> 于跟踪 SLA 和性能问题；
> 3. 历史分析，可以从中找出趋势和历史模式，行为基线和阈值，生成一致性报告。

*——2014 年 11 月 17 日摘自 Peter Zadrozny, Raghu Kodali 著/唐宏，陈健译《Splunk大数据分析》*

## 参阅

* [Elasticsearch 权威指南](http://fuxiaopang.gitbooks.io/learnelasticsearch/)
* [Logstash 最佳实践](https://www.gitbook.io/book/chenryn/logstash-best-practice)
* [The Logstash Book](http://www.logstashbook.com/)

## 进度

[![Build Status](https://www.gitbook.io/button/status/book/chenryn/kibana-guide-cn)](https://www.gitbook.io/book/chenryn/kibana-guide-cn/activity)

*欢迎捐赠，作者支付宝账号：<rao.chenlin@gmail.com>*

![ercode](img/alipay.png)
