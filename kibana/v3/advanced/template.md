# 模板和脚本

Kibana 支持通过模板或者更高级的脚本来动态的创建仪表板。你先创建一个基础的仪表板，然后通过参数来改变它，比如通过 URL 插入一个新的请求或者过滤规则。

模板和脚本都必须存储在磁盘上，目前不支持存储在 Elasticsearch 里。同时它们也必须是通过编辑或创建纲要生成的。所以我们强烈建议阅读 [The Kibana Schema Explained](http://www.elasticsearch.org/guide/en/kibana/3.0/_dashboard_schema.html)

## 仪表板目录

仪表板存储在 Kibana 安装目录里的 `app/dashboards` 子目录里。你会注意到这里面有两种文件：`.json` 文件和 `.js` 文件。

## 模板化仪表板(.json)

`.json` 文件就是模板化的仪表板。模板示例可以在 `logstash.json` 仪表板的请求和过滤对象里找到。模板使用 handlebars 语法，可以让你在 json 里插入 javascript 语句。URL 参数存在 `ARGS` 对象中。下面是 `logstash.json`([on github](https://github.com/elasticsearch/kibana/blob/master/src/app/dashboards/logstash.json)) 里请求和过滤服务的代码片段：

```
  "0": {
    "query": "{{ARGS.query || '*'}}",
    "alias": "",
    "color": "#7EB26D",
    "id": 0,
    "pin": false
  }

  [...]

  "0": {
    "type": "time",
    "field": "@timestamp",
    "from": "now-{{ARGS.from || '24h'}}",
    "to": "now",
    "mandate": "must",
    "active": true,
    "alias": "",
    "id": 0
  }
```

这允许我们在 URL 里设置两个参数，`query` 和 `from`。如果没设置，默认值就是 `||` 后面的内容。比如说，下面的 URL 就会搜索过去 7 天内 `status:200` 的数据：

注意：千万注意 url `#/dashboard/file/logstash.json` 里的 `file` 字样

```
http://yourserver/index.html#/dashboard/file/logstash.json?query=status:200&from=7d
```

