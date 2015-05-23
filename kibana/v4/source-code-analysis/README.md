# 源码剖析

Kibana 4 采用 angular.js + node.js 框架编写。其中 node.js 主要提供两部分功能，给 Elasticsearch 做搜索请求转发代理，以及 auth、ssl、setting 等操作的服务器后端。

本章节假设你已经对 angular 有一定程度了解 —— 至少是阅读并理解了 kibana 3 源码剖析章节内容的程度。所以不会再解释其中 angular 的 route，controller，directive，service，factory 等概念。

如果打算迁移 kibana 3 的 CAS 验证功能到 kibana 4，那么可以稍微了解一下 `index.js`, `app.js`, `lib/auth.js` 里的 htpasswd 简单实现，相信可以很快修改成功。本章主要还是集中在前端 kibana 页面功能的实现上。

在 Elastic{ON} 大会上，也有专门针对 Kibana 4 源码和二次开发入门的演讲。请参阅：<https://speakerdeck.com/elastic/the-contributors-guide-to-the-kibana-galaxy>

另外可以看专业的前端工程师怎么看Kibana4的代码的：<http://www.debuggerstepthrough.com/2015/04/reviewing-kibana-4s-client-side-code.html>。
