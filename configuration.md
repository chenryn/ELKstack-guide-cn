# 配置

config.js 是 Kibana 核心配置的地方。文件里包括得参数都是必须在初次运行 kibana 之前提前设置好的。

## 参数

**elasticsearch**

你 elasticsearch 服务器的 URL 访问地址。你应该不会像写个 `http://localhost:9200` 在这，哪怕你的 Kibana 和 Elasticsearch 是在同一台服务器上。默认的时候这里会尝试访问你部署 kibana 的服务器上的 ES 服务，你可能需要设置为你 elasticsearch 服务器的主机名。

注意：如果你要传递参数给 http 客户端，这里也可以设置成对象形式，如下：

```
+elasticsearch: {server: "http://localhost:9200", withCredentials: true}+
```

**default_route**

没有指明加载哪个仪表板的时候，默认加载页路径的设置参数。你可以设置为文件，脚本或者保存的仪表板。比如，你有一个保存成 "WebLogs" 的仪表板在 elasticsearch 里，那么你就可以设置成：

```
default_route: /dashboard/elasticsearch/WebLogs,
```

**kibana-int**

默认用来保存 Kibana 相关对象，比如仪表板，的 Elasticsearch 索引名称。

**panel_name**

可用的面板模块数组。面板只有在仪表板中有定义时才会被加载，这个数组只是用在 "add panel" 界面里做下拉菜单。


