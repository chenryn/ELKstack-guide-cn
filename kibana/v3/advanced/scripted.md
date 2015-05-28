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
