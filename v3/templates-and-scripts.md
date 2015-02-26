# 模板和脚本

Kibana 支持通过模板或者更高级的脚本来动态的创建仪表板。你先创建一个基础的仪表板，然后通过参数来改变它，比如通过 URL 插入一个新的请求或者过滤规则。

模板和脚本都必须存储在磁盘上，目前不支持存储在 Elasticsearch 里。同时它们也必须是通过编辑或创建纲要生成的。所以我们强烈建议阅读 [The Kibana Schema Explained](http://www.elasticsearch.org/guide/en/kibana/current/_dashboard_schema.html)

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

## 脚本化仪表板(.js)

脚本化仪表板比模板化仪表板更加强大。当然，功能强大随之而来的就是构建起来也更复杂。脚本化仪表板的目的就是构建并返回一个描述了完整的仪表板纲要的 javascript 对象。`app/dashboards/logstash.js`([on github](https://github.com/elasticsearch/kibana/blob/master/src/app/dashboards/logstash.js)) 就是一个有着详细注释的脚本化仪表板示例。这个文件的最终结果和 `logstash.json` 一致，但提供了更强大的功能，比如我们可以以逗号分割多个请求：

注意：千万注意 URL `#/dashboard/script/logstash.js` 里的 `script` 字样。这让 kibana 解析对应的文件为 javascript 脚本。

```
http://yourserver/index.html#/dashboard/script/logstash.js?query=status:403,status:404&from=7d
```

这会创建 2 个请求对象，`status:403` 和 `status:404` 并分别绘图。事实上这个仪表板还能接收另一个参数 `split`，用于指定用什么字符串切分。

```
http://yourserver/index.html#/dashboard/script/logstash.js?query=status:403!status:404&from=7d&split=!
```

我们可以看到 `logstash.js` ([on github](https://github.com/elasticsearch/kibana/blob/master/src/app/dashboards/logstash.js)) 里是这么做的：

```javascript
// In this dashboard we let users pass queries as comma separated list to the query parameter.
// Or they can specify a split character using the split aparameter
// If query is defined, split it into a list of query objects
// NOTE: ids must be integers, hence the parseInt()s
if(!_.isUndefined(ARGS.query)) {
  queries = _.object(_.map(ARGS.query.split(ARGS.split||','), function(v,k) {
    return [k,{
      query: v,
      id: parseInt(k,10),
      alias: v
    }];
  }));
} else {
  // No queries passed? Initialize a single query to match everything
  queries = {
    0: {
      query: '*',
      id: 0,
    }
  };
}
```

该仪表板可用参数比上面讲述的还要多，全部参数都在 `logstash.js`([on github](https://github.com/elasticsearch/kibana/blob/master/src/app/dashboards/logstash.js)) 文件里开始的注释中有讲解。
