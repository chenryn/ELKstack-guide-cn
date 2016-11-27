# Kibana 服务器端插件开发

上一节介绍了如何给 Kibana 开发浏览器端的可视化插件。新版 Kibana 跟 Kibana3 比，最大的一个变化是有了独立的 node.js 服务器端。那么同样的，也就有了服务器端的 Kibana 插件。最明显的一个场景：我们可以在 node.js 里跑定时器做 Elasticsearch 的告警逻辑了！

本节示例一个最基础的 Kibana 告警插件开发。只演示基础的定时器和 Kibana 插件规范，实际运用中，肯定还涉及历史记录，告警项配置更新等。请读者不要直接 copy-paste。

首先，我们尽量沿袭 Elastic 官方的 watcher 产品的告警配置设计。也新建一个索引，里面是具体的配置内容：

```json
# curl -XPUT http://127.0.0.1:9200/watcher/watch/error_status -d'
{
  "trigger": {
    "schedule" : { "interval" : "60"  }
  },
  "input" : {
    "search" : {
      "request" : {
        "indices" : [ "<logstash-{now/d}>", "<logstash-{now/d-1d}>"  ],
        "body" : {
          "query" : {
            "filtered" : {
              "query" : { "match" : { "host" : "MacBook-Pro"  } },
              "filter" : { "range" : { "@timestamp" : { "from" : "now-5m"  } } }
            }
          }
        }
      }
    }
  },
  "condition" : {
    "script" : {
      "script" : "payload.hits.total > 0"
    }
  },
  "transform" : {
    "search" : {
      "request" : {
        "indices" : [ "<logstash-{now/d}>", "<logstash-{now/d-1d}>"  ],
        "body" : {
          "query" : {
            "filtered" : {
              "query" : { "match" : { "host" : "MacBook-Pro"  } },
              "filter" : { "range" : { "@timestamp" : { "from" : "now-5m"  } } }
            }
          },
          "aggs" : {
            "topn" : {
              "terms" : {
                "field" : "path.raw"
              }
            }
          }
        }
      }
    }
  },
  "actions" : {
    "email_admin" : {
    "throttle_period" : "15m",
    "email" : {
      "to" : "admin@domain",
      "subject" : "Found {{payload.hits.total}} Error Events",
      "priority" : "high",
      "body" : "Top10 paths:\n{{#payload.aggregations.topn.buckets}}\t{{key}} {{doc_count}}\n{{/payload.aggregations.topn.buckets}}"
    }
    }
  }
}'
```

我们可以看到，跟原版的相比，只改动了很小的一些地方：

1. 为了简便，`interval` 固定写数值，没带 `s/m/d/H` 之类的单位；
2. `condition` 里直接使用了 JavaScript，这点也是 ES 2.x 的 mapping 要求跟 watcher 本身有冲突的一个地方：watcher的 `"ctx.payload.hits.total" : { "gt" : 0 }` 这种写法，如果是普通索引，会因为字段名里带 `.` 直接写入失败的；
3. 因为是在 Kibana 里面运行，所以从 ES 拿到的只有 payload(也就是查询响应)，所以把里面的 `ctx.` 都删掉了。

好，然后创建插件：

```
cd kibana-4.3.0-darwin-x64/src/plugins
mkdir alert
```

在自定义插件目录底下创建 `package.json` 描述：

```json
{
  "name": "alert",
  "version": "0.0.1"
}
```

以及最终的 `index.js` 代码：

```javascript
'use strict';
module.exports = function (kibana) {
  var later = require('later');
  var _ = require('lodash');
  var mustache = require('mustache');

  return new kibana.Plugin({
    init: function init(server) {
      var client = server.plugins.elasticsearch.client;
      var sched = later.parse.text('every 10 minute');
      later.setInterval(doalert, sched);
      function doalert() {
        getCount().then(function(resp){
          getWatcher(resp.count).then(function(resp){
            _.each(resp.hits.hits, function(hit){
              var watch = hit._source;
              var every = watch.trigger.schedule.interval;
              var watchSched = later.parse.recur().every(every).second();
              var wt = later.setInterval(watching, watchSched);
              function watching() {
                var request = watch.input.search.request;
                var condition = watch.condition.script.script;
                var transform = watch.transform.search.request;
                var actions = watch.actions;
                client.search(request).then(function(payload){
                  var ret = eval(condition);
                  if (ret) {
                    client.search(transform).then(function(payload) {
                      _.each(_.values(actions), function(action){
                        if(_.has(action, 'email')) {
                          var subject = mustache.render(action.email.subject, {"payload":payload});
                          var body = mustache.render(action.email.body, {"payload":payload});
                          console.log(subject, body);
                        }
                      });
                    });
                  }
                });
              }
            });
          });
        });
      }
      function getCount() {
        return client.count({
          index:'watcher',
          type:"watch"
        });
      }
      function getWatcher(count) {
        return client.search({
          index:'watcher',
          type:"watch",
          size:count
        });
      }
    }
  });
};
```

其中用到了两个 npm 模块，later 模块用来实现定时器和 crontab 文本解析，mustache 模块用来渲染邮件内容模板，这也是 watcher 本身采用的渲染模块。

需要安装一下：

```
npm install later
npm install mustache
```

然后运行 `./bin/kibana`，就可以看到终端上除了原有的内容以外，还会定期输出 alert 的 email 内容了。

## 要点解释

这个极简示例中，主要有两段：

1. 注册为插件

```
module.exports = function (kibana) {
  return new kibana.Plugin({
    init: function init(server) {
```

注意上一节的可视化插件，这块是：

```
module.exports = function (kibana) {
  return new kibana.Plugin({
    uiExports: {
      visTypes: [
```

2. 引用 ES client

```
    init: function init(server) {
      var client = server.plugins.elasticsearch.client;
```

这里通过调用 `server.plugins` 来直接引用 Kibana 里其他插件里的对象。这样，alert 插件就可以跟其他功能共用同一个 ES client，免去单独配置自己的 ES 设置项和新开网络连接的资源消耗。

本节代码后续优化改进，见：<https://github.com/chenryn/kaae>。项目中还附带有一个 spy 式插件，有兴趣的读者可以继续学习 spy 这类不太常见的插件扩展的用法。
