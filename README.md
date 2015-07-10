# 介绍

## 历史沿革

我们可以首先了解一下 ELK stack 中三个软件各自的发展历程：

### Elasticsearch 历史

Elasticsearch 来源于作者 Shay Banon 的第一个开源项目 Compass 库，而这个 Java 库最初的目的只是为了给 Shay 当时正在学厨师的妻子做一个菜谱的搜索引擎。2010 年，Elasticsearch 正式发布。至今已经成为 GitHub 上最流行的 Java 项目，不过 Shay 承诺给妻子的菜谱搜索依然没有面世……

2015 年初，Elasticsearch 公司召开了第一次全球用户大会 Elastic{ON}15。诸多 IT 巨头纷纷赞助，参会，演讲。会后，Elasticsearch 公司宣布改名 Elastic，公司官网也变成 <http://elastic.co/>。这意味着 Elasticsearch 的发展方向，不再限于搜索业务，也就是说，ELKstack 等机器数据和 IT 服务领域成为官方更加注意的方向。随后几个月，专注监控报警的 Watcher 发布 beta 版，社区有名的网络抓包工具 Packetbeat 也在前几天被 Elastic 公司收购。

### Logstash 历史

Logstash 项目诞生于 2009 年 8 月 2 日。其作者是世界著名的运维工程师乔丹西塞(JordanSissel)，乔丹西塞当时是著名虚拟主机托管商 DreamHost 的员工，还发布过非常棒的软件打包工具 fpm，并主办着一年一度的 sysadmin advent calendar(advent calendar 文化源自基督教氛围浓厚的 Perl 社区，在每年圣诞来临的 12 月举办，从 12 月 1 日起至 12 月 24 日止，每天发布一篇小短文介绍主题相关技术)。

*小贴士：Logstash 动手很早，对比一下，scribed 诞生于 2008 年，flume 诞生于 2010 年，Graylog2 诞生于 2010 年，Fluentd 诞生于 2011 年。*

scribed 在 2011 年进入半死不活的状态，大大激发了其他各种开源日志收集处理框架的蓬勃发展，Logstash 也从 2011 年开始进入 commit 密集期并延续至今。

作为一个系出名门的产品，Logstash 的身影多次出现在 Sysadmin Weekly 上，它和它的小伙伴们 Elasticsearch、Kibana 直接成为了和商业产品 Splunk 做比较的开源项目(乔丹西塞曾经在博客上承认设计想法来自 AWS 平台上最大的第三方日志服务商 Loggy，而 Loggy 两位创始人都曾是 Splunk 员工)。

2013 年，Logstash 被 Elasticsearch 公司收购，ELK stack 正式成为官方用语(虽然还没正式命名)。Elasticsearch 本身 也是近两年最受关注的大数据项目之一，三次融资已经超过一亿美元。在 Elasticsearch 开发人员的共同努力下，Logstash 的发布机制，插件架构也愈发科学和合理。

### Kibana 历史

Logstash 早期曾经自带了一个特别简单的 logstash-web 用来查看 ES 中的数据。其功能太过简单，于是 Rashid Khan 用 PHP 写了一个更好用的 web，取名叫 Kibana。这个 PHP 版本的 Kibana 发布时间是 2011 年 12 月 11 日。

Kibana 迅速流行起来，不久的 2012 年 8 月 19 日，Rashid Khan 用 Ruby 重写了 Kibana，也被叫做 Kibana2。因为 Logstash 也是用 Ruby 写的，这样 Kibana 就可以替代原先那个简陋的 logstash-web 页面了。

目前我们看到的 angularjs 版本 kibana 其实原名叫 elasticsearch-dashboard，kibana 原先是 RoR 框架的另一个项目，但作者是同一个人，换句话说，kibana 比 logstash 还早就进了 elasticsearch 名下。这个项目改名 Kibana 是在 2014 年 2 月，也被叫做 Kibana3。全新的设计一下子风靡 DevOps 界。随后其他社区纷纷借鉴，Graphite 目前最流行的 Grafana 界面就是由此而来，至今代码中还留存有十余处 kbn 字样。

2014 年 4 月，Kibana3 停止开发，ES 公司集中人力开始 Kibana4 的重构，在 2015 年初发布了使用 JRuby 做后端的 beta 版后，于 3 月正式推出使用 node.js 做后端的正式版。由于设计思路上的差别，一些 K3 适宜的场景并不在 K4 考虑范围内，所以，至今 K3 和 K4 并存使用。本书也会分别讲解两者。

ELKstack 已经成为最近一年 IT 服务领域最为瞩目的大数据分析套件。

—— 2015 年 5 月 28 日


## 社区文化

日志收集处理框架这么多，像 scribe 是 facebook 出品，flume 是 apache 基金会项目，都算声名赫赫。但 logstash 因乔丹西塞的个人性格，形成了一套独特的社区文化。每一个在 google groups 的 logstash-users 组里问答的人都会看到这么一句话：

**Remember: if a new user has a bad time, it's a bug in logstash.**

所以，logstash 是一个开放的，极其互助和友好的大家庭。有任何问题，尽管在 github issue，Google groups，Freenode#logstash channel 上发问就好！

## ELKstack 与 Hadoop 体系的区别

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
* [精通 Elasticsearch](http://shgy.gitbooks.io/mastering-elasticsearch/)
* [The Logstash Book](http://www.logstashbook.com/)

## 进度

[![Build Status](https://www.gitbook.io/button/status/book/chenryn/kibana-guide-cn)](https://www.gitbook.io/book/chenryn/kibana-guide-cn/activity)

*欢迎捐赠，作者支付宝账号：<rao.chenlin@gmail.com>*

![ercode](kibana/v3/img/alipay.png)
