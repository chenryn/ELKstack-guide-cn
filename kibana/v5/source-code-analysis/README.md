# 源码剖析

Kibana 4 开始采用 angular.js + node.js 框架编写。其中 node.js 主要提供两部分功能，给 Elasticsearch 做搜索请求转发代理，以及 auth、ssl、setting 等操作的服务器后端。5.0 版本在这些基础构成方面没有太大变化。

本章节假设你已经对 angular 有一定程度了解。所以不会再解释其中 angular 的 route，controller，directive，service，factory 等概念。

如果打算迁移 kibana 3 的 CAS 验证功能到 Kibana 4 或 5 版本，那么可以稍微了解一下 Hapi.JS 框架的认证扩展，相信可以很快修改成功。本章主要还是集中在前端 kibana 页面功能的实现上。

在 Elastic{ON} 大会上，也有专门针对 Kibana 4 源码和二次开发入门的演讲。请参阅：<https://speakerdeck.com/elastic/the-contributors-guide-to-the-kibana-galaxy>

另外可以看专业的前端工程师怎么看Kibana 4 的代码的：<http://www.debuggerstepthrough.com/2015/04/reviewing-kibana-4s-client-side-code.html>。
