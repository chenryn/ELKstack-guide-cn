# 源码剖析与二次开发

Kibana 3 作为 Elastic Stack 风靡世界的最大推动力，其与优美的界面配套的简洁的代码同样功不可没。事实上，graphite 社区就通过移植 kibana 3 代码框架的方式，启动了 [grafana 项目](http://grafana.org/)。至今你还能在 grafana 源码找到二十多处 "kbn" 字样。

*巧合的是，在 Kibana 重构 v4 版的同时，grafana 的 v2 版也到了 Alpha 阶段，从目前的预览效果看，主体 dashboard 沿用了 Kibana 3 的风格，不过添加了额外的菜单栏，供用户权限设置等使用 —— 这意味着 grafana 2 跟 kibana 4 一样需要一个单独的 server 端。*

笔者并非专业的前端工程师，对 angularjs 也处于一本入门指南都没看过的水准。所以本节内容，只会抽取一些个人经验中会有涉及到的地方提出一些"私货"。欢迎方家指正。

