# Kibana 各版本的对比

Kibana 一直是 Elastic Stack 中的版本帝，并最终带着整个技术栈的产品都提升版本号到了 5.0。和其他产品一路升级上来不同的是，Kibana 几乎每次版本升级都是革命性的重写。而且至今依然有很多 Kibana 3 的用户不愿意迁移升级。因为前后者分别基于不同接口，不同目的，采取了不同的页面设计和逻辑。在不同场景下，各有优势。这里稍作解释。

## Kibana 3 的设计思路和功能

本书一开始就提到，Kibana3 在设计之初，有另一个名字，叫 **elasticsearch dashboard**。事实上，整个 Kibana3 就是一个围绕着 dashboard 构建的单页应用。

所以，在页面逻辑上，Kibana3 异常简洁。大量的代码和逻辑，都下放到 panel 层次上。每个 panel 要独立完成自己的可视化设计、数据请求，数据处理，数据渲染。panel 和 panel 之间，则几乎毫无关联。简单一点看，整个页面就像是一堆 iframe 一样。

而 panel 的设计，则是以使用者角度来考虑的。Kibana3 尽量提供能让运维人员一步到位的使用策略。即，使用者只需要了解 panel 的配置页面能填什么参数，得到什么可视化结果。

最明显的例子，就是 trend panel。trend panel 背后，其实是针对今天和昨天，分别发起两次请求，然后再拿两次请求的结果，做一次除法，计算涨跌幅。这个除法计算，是在浏览器端完成的。

类似在浏览器端完成的，还有 histogram panel 的 hits，second 等的计算。

此外，Kibana3 还有一个非常有用的功能，setting 中的 index pattern，是可以输入多个的，比如 `accesslog-[YYYY.MM.DD],syslog-[YYYY.MM.DD]`，这样就可以在同一个面板上，看到来自不同索引的数据的情况。

## Kibana 4 的设计思路和功能

从新特性来说，Kibana4 全面支持 Aggregation 接口，还有更多的可视化选择，可以任意拖动自动对齐的挂件框架，保存在 URL 可以跨页面保持的检索条件，以及对页面请求的内部排队机制。

从页面设计来说，Kibana4 参考了 Splunk 的产品形态，将功能拆分成了搜索，可视化和仪表盘三个标签页。可视化和搜索，是一一绑定的，无法跨多个 index pattern 做搜索，勿论可视化了。而且可视化标签页中，用 d3.js 实现的可视化构建器，与请求 ES 数据的聚合选择器，又是各自独立的插件。

也就是说，Kibana4 在使用 Aggregation 接口提供更复杂功能和更高性能的同时，彻底改变了用户的使用形式。用户必须明确了解 ES 各个 aggs 接口的意义，请求和响应体的数据情况；还要想清楚可视化的展现形式，充分理解数据字段的作用。然后才能实现想要的结果。毫无疑问，这是有学习成本的。

至于像 Kibana 3 那种在浏览器端计算的功能，Kibana4 中则完全没有。

在界面美观方面。Kibana4 至今未提供类似 Kibana3 中的 Query 设置功能，包括 Query 别名这个常用功能。直接导致目前 Kibana4 的图例几乎毫无作用。

在 filter 方面，Kibana4 用 filter agg 替代了 Kibana3 使用的 facet\_filter。页面表现形式上，Kibana3 是在页面顶部添加 Query 输入框，全局生效；Kibana4 是在 Visualize 页添加 aggs，单个面板生效。依然需要多查询条件对比的用户，需要一个个面板创建，非常麻烦。

## Kibana 5 的设计思路和功能

Kibana 5 在 4.6 的基础上，做了全面的页面布局修改，采用了更现代化的左侧边栏菜单设计。单就 Kibana 本身的功能来说，其实没有太大差别。但是 Kibana 5 将重心放在了应用扩展和融合上。左侧边栏显然可以留出更多的空间来摆放各种扩展应用的 logo 图标。

ES 2.0 开始提供了一种全新的 pipeline aggregation 特性， Kibana 在这个 ES 新特性的基础上实现了 timelion 应用，专注于时序计算和分析。甚至可以将 timelion 的分析图表加入到仪表盘中和其他可视化一起展现。

此外，ES 5.0 取消了 site plugin 形式，原来的 site plugin 都要迁移成 Kibana app。目前官方已经将原先的 sense 改成了 Kibana console 应用。相信未来一段时间，会有更多的 Kibana 应用涌现出来。
