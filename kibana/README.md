# 简介

Logstash 早期曾经自带了一个特别简单的 logstash-web 用来查看 ES 中的数据。其功能太过简单，于是 Rashid Khan 用 PHP 写了一个更好用的 web，取名叫 Kibana。这个 PHP 版本的 Kibana 发布时间是 2011 年 12 月 11 日。

Kibana 迅速流行起来，不久的 2012 年 8 月 19 日，Rashid Khan 用 Ruby 重写了 Kibana，也被叫做 Kibana2。因为 Logstash 也是用 Ruby 写的，这样 Kibana 就可以替代原先那个简陋的 logstash-web 页面了。

目前我们看到的 angularjs 版本 kibana 其实原名叫 elasticsearch-dashboard，但跟 Kibana2 作者是同一个人，换句话说，kibana 比 logstash 还早就进了 elasticsearch 名下。这个项目改名 Kibana 是在 2014 年 2 月，也被叫做 Kibana3。全新的设计一下子风靡 DevOps 界。随后其他社区纷纷借鉴，Graphite 目前最流行的 Grafana 界面就是由此而来，至今代码中还留存有十余处 kbn 字样。

2014 年 4 月，Kibana3 停止开发，ES 公司集中人力开始 Kibana4 的重构，在 2015 年初发布了使用 JRuby 做后端的 beta 版后，于 3 月正式推出使用 node.js 做后端的正式版。由于设计思路上的差别，一些 K3 适宜的场景并不在 K4 考虑范围内，所以，至今 K3 和 K4 并存使用。本书也会分别讲解两者。

## 注释

本章配置参数内容部分译自[Elasticsearch 官方指南 Kibana 部分](http://www.elasticsearch.org/guide/en/kibana/current/index.html)，其中 [v3](http://www.elasticsearch.org/guide/en/kibana/3.0/index.html) 的 panel 部分额外添加了截图注释。然后在使用的基础上，添加了原创的源码解析，场景示例，二次开发入门等内容。

