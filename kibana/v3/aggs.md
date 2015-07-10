# percentile agg 面板开发

Kibana3.1 中有一个面板是 stats 类型，返回对应请求的某指定数值字段的数学统计值，包括最大值、最小值、平均值、方差和标准差。这个 stats 图表是利用 Elasticsearch 的 facets 功能来实现的。而在 Elasticsearch 1.0 版本以后，新出现了一个更细致的功能叫 aggregation，按照官方文档所说，会慢慢的彻底替代掉 facets。具体到 1.1 版本的时候， aggregation 里多了一项 percentile，可以具体返回某指定数值字段的区间分布情况。这对日志分析可是大有帮助。对这项功能，Elasticsearch 官方也很得意的专门在博客上写了一篇报道：[Averages can be misleading: try a percentile](http://www.elasticsearch.org/blog/averages-can-dangerous-use-percentile/)。

percentile agg 请求示例如下：

```
# curl -XPOST http://127.0.0.1:9200/logstash-2014.07.11/_search -d '{
    "aggs" : {
        "request_time_percentiles" : {
            "percentiles" : {
                "field" : "request_time", 
                "percents" : [50,75,90,99]
            }
        }
    }
}'
```

本节就讲解如何基于 Elasticsearch 的 percentile Aggregation 接口实现统计聚合展示面板。其他 Aggregation 也可以类比。

和上一节实现 facet 接口面板相比，页面布局和主体逻辑基本一致。其难点在以下几方面：

* Kibana3 使用了社区流行的 `elastic.js` 第三方库作为基础依赖库。而 `kibana/src/vendor/elasticjs/elastic.js` 文件开头写着版本号是 `v1.1.1`，但是其实它是 2013-08-14 发布的。而具体加上 aggregation 支持的时间是 2014-03-16 ，但是版本号依然是 `v1.1.1`！所以在官网文档看到 1.1.1 版的 aggs 语法，其实在 Kibana 里都用不了。
* elastic.js 新版在支持 aggs 接口的同时，对本身的底层依赖也做了大幅度改动，和 ES 的实际交互已经改用官方的 `elasticsearch.js` 库，elastic.js 本身相当于只是做一个封装。但是 `elasticsearch.js` 本身目录结构复杂，加入到 `require.config.js` 里也不是那么容易。

所以，这里有两种解决办法。

1. 直接使用 angularjs 框架的 `$.http` 对象，手拼 JSON 请求体。把 kibana 当做 curl 来处理得了。

```
      request = {
        'stats': {
          'filter': JSON.parse($scope.ejs.QueryFilter(
            $scope.ejs.FilteredQuery(
              boolQuery,
              filterSrv.getBoolFilter(filterSrv.ids())
            )
          ).toString(), true),
          'aggs': {
            'stats': {
              'percentiles': {
                'field': $scope.panel.field,
                'percents': $scope.modes
              }
            }
          }
        }
      };

      $.each(queries, function (i, q) {
        var query = $scope.ejs.BoolQuery();
        query.should(querySrv.toEjsObj(q));
        var qname = 'stats_'+i;
        var aggsquery = {};
        aggsquery[qname] = {
          'percentiles': {
            'field': $scope.panel.field,
            'percents': $scope.modes
          }
        };
        request[qname] = {
          'filter': JSON.parse($scope.ejs.QueryFilter(
            $scope.ejs.FilteredQuery(
              query,
              filterSrv.getBoolFilter(filterSrv.ids())
            )
          ).toString(), true),
          'aggs': aggsquery
        };
      });

      $scope.inspector = angular.toJson({aggs:request},true);

      results = $http({
        url: config.elasticsearch + '/' + dashboard.indices + '/_search?size=0',
        method: "POST",
        data: { aggs: request }
      });
```

2. 统一升级整个 kibana3 的基础依赖库版本，然后采用 agg 接口方法。

```
      request = $scope.ejs.Request();

      $scope.panel.queries.ids = querySrv.idsByMode($scope.panel.queries);
      queries = querySrv.getQueryObjs($scope.panel.queries.ids);
      boolQuery = $scope.ejs.BoolQuery();
      _.each(queries,function(q) {
        boolQuery = boolQuery.should(querySrv.toEjsObj(q));
      });

      var percents = _.keys($scope.panel.show);

      request = request
        .aggregation(
          $scope.ejs.FilterAggregation('stats')
            .filter($scope.ejs.QueryFilter(
              $scope.ejs.FilteredQuery(
                boolQuery,
                filterSrv.getBoolFilter(filterSrv.ids())
              )
            ))
            .aggregation($scope.ejs.PercentilesAggregation('stats')
              .field($scope.panel.field)
              .percents(percents)
              .compression($scope.panel.compression)
            )
          ).size(0);
```

页面参数上，根据 percentile 的请求参数需要，主要需要提供一个 `$scope.panel.modes` 数组即可。不赘述。

## 代码实现要点

### 1.1 和 1.3 版本的返回结果集层次变动

percentile Aggregation 是 Elasticsearch 从 1.1.0 开始新加入的实验性功能，而且在 1.3.0 之后其返回的数据结构发生了变动。所以代码中对 ES 的版本要做判断和兼容性处理。

Kibana3 提供了一个 service 叫 `esVersion`，所以我们可以直接这样：

```
  module.controller('percentiles', function ($scope, querySrv, dashboard, filterSrv, $http, esVersion) {
    ...
      results.then(function(results) {
        $scope.panelMeta.loading = false;
        esVersion.gte('1.3.0').then(function(is) {
          if (is) {
            var value = results.aggregations.stats['stats']['values'][$scope.panel.mode+'.0'];
            ...
          } else {
            esVersion.gte('1.1.0').then(function(is) {
              if (is) {
                var value = results.aggregations.stats['stats'][$scope.panel.mode+'.0'];
                ...
              }
            });
          }
        });
      });
```

### 浮点数排序

percentile Aggregation 返回的数据中，强制保留了百分数的小数点后一位，这导致在 js 处理中会把小数点当做是属性调用的操作符，Kibana 提供的表头点击自动排序表格数据功能也就失效了。所以需要替换掉 sort_field 里的小数点。

`module.js` 中：

```
            var rows = queries.map(function (q, i) {
              var alias = q.alias || q.query;
              var obj = _.clone(q);
              obj.label = alias;
              obj.Label = alias.toLowerCase(); //sort field
              obj.value = results.aggregations['stats_'+i]['stats_'+i]['values'];
              obj.Value = results.aggregations['stats_'+i]['stats_'+i]['values']; //sort field
              var _V = {}
              for ( var k in obj.Value ) {
                  var v =  obj.Value[k];
                  k = k.replace('.','');
                  _V[k] = v;
              }
              obj.Value = _V;
              return obj;
            });
            $scope.data = {
              value: value,
              rows: rows
            };
            $scope.$emit('render');
```

`module.html` 中：

```
      <thead>
        <tr>
         <th><a href="" ng-click="set_sort('label')" ng-class="{'icon-chevron-down': panel.sort_field == 'label' && panel.sort_reverse == true, 'icon-chevron-up': panel.sort_field == 'label' && panel.sort_reverse == false}"> {{panel.label_name}} </a></th>
         <th ng-repeat="stat in modes" ng-show="panel.show[stat]">
          <a href=""
            ng-click="set_sort(stat)"
            ng-class="{'icon-chevron-down': panel.sort_field == stat.replace('.','') && panel.sort_reverse == true, 'icon-chevron-up': panel.sort_field == stat.replace('.','') && panel.sort_reverse == false}">
            {{stat}}%
          </a>
          </th>
        </tr>
      </thead>
```

### query alias 的中文支持

目前 Kibana 里都是以 alias 形式来区分每一个子请求的，具体内容是 `var alias = q.alias || q.query;`，即在页面上搜索框里写的查询语句或者是搜索框左侧色彩设置菜单里的 `Legend value`。

比如我的场景下，`q.query` 是 "xff:10.5.16.\*"，`q.alias` 是"教育网访问"。那么最后发送的请求里这条过滤项的 `facets_name` 就叫 "stats\_教育网访问"。

同样的写法迁移到 aggregation 上就完全不可解析了。**服务器会返回一条报错说：`aggregation_name` 只能是字母、数字、`_` 或者 `-` 四种。**

*这里比较怪的是抓包看到 facets 其实也报错说请求内容解析失败，但是居然同时也返回了结果，只能猜测目前是处在一种兼容状态？*

于是这里稍微修改了一下逻辑，把 `queries` 数组的 `_.each` 改用 `$.each` 来做，这样回调函数里不单返回数组元素，还返回数组下标，下标是一定为数字的，就可以以数组下标作为 `aggregation_name` 了。后面处理结果的 `queries.map` 同样以下标来获取即可。

```
      $.each(queries, function (i, q) {
        var query = $scope.ejs.BoolQuery();
        query.should(querySrv.toEjsObj(q));
        var qname = 'stats_'+i;

        request.aggregation(
          $scope.ejs.FilterAggregation(qname)
            .filter($scope.ejs.QueryFilter(
              $scope.ejs.FilteredQuery(
                query,
                filterSrv.getBoolFilter(filterSrv.ids())
              )
            ))
            .aggregation($scope.ejs.PercentilesAggregation(qname)
              .field($scope.panel.field)
              .percents(percents)
              .compression($scope.panel.compression)
            )
          );
      });
      $scope.inspector = request.toJSON();
      results = $scope.ejs.doSearch(dashboard.indices, request);
```

## 面板效果

最终的 percentile 面板界面与 stats 面板界面类似。

![percentile aggr](http://ww4.sinaimg.cn/large/3dbd9afatw1eh4kh2se8lj209m093aai.jpg)
