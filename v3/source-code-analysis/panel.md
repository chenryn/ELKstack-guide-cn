## panel 内部实现

终于说到最后了。大家进入到 `app/panels/` 下，每个目录都是一种 panel。原因前一节已经分析过了，因为 addPanel.js 里就是直接这样拼接的。入口都是固定的：module.js。

下面以 stats panel 为例。(因为我最开始就是抄的 stats 做的 percentile，只有表格没有图形，最简单)

每个目录下都会有至少一下三个文件：

### module.js

module.js 就是一个 controller。跟前面讲过的 controller 写法其实是一致的。在 `$scope` 对象上，有几个属性是 panel 实现时一般都会有的：

* `$scope.panelMeta`: 这个前面说到过，其中的 modals 用来定义 panelHeader。
* `$scope.panel`: 用来定义 panel 的属性。一般实现上，会有一个 default 值预定义好。你会发现这个 `$scope.panel` 其实就是仪表板纲要里面说的每个 panel 的可设置值！

然后一般 `$scope.init()` 都是这样的：

```
    $scope.init = function () {
      $scope.ready = false;
      $scope.$on('refresh', function () {
        $scope.get_data();
      });
      $scope.get_data();
    };
```

也就是每次有刷新操作，就执行 `get_data()` 方法。这个方法就是获取 ES 数据，然后渲染效果的入口。

```
    $scope.get_data = function () {
      if(dashboard.indices.length === 0) {
        return;
      }

      $scope.panelMeta.loading = true;

      var request,
        results,
        boolQuery,
        queries;

      request = $scope.ejs.Request();

      $scope.panel.queries.ids = querySrv.idsByMode($scope.panel.queries);
      queries = querySrv.getQueryObjs($scope.panel.queries.ids);

      boolQuery = $scope.ejs.BoolQuery();
      _.each(queries,function(q) {
        boolQuery = boolQuery.should(querySrv.toEjsObj(q));
      });

      request = request
        .facet($scope.ejs.StatisticalFacet('stats')
          .field($scope.panel.field)
          .facetFilter($scope.ejs.QueryFilter(
            $scope.ejs.FilteredQuery(
              boolQuery,
              filterSrv.getBoolFilter(filterSrv.ids())
              )))).size(0);

      _.each(queries, function (q) {
        var alias = q.alias || q.query;
        var query = $scope.ejs.BoolQuery();
        query.should(querySrv.toEjsObj(q));
        request.facet($scope.ejs.StatisticalFacet('stats_'+alias)
          .field($scope.panel.field)
          .facetFilter($scope.ejs.QueryFilter(
            $scope.ejs.FilteredQuery(
              query,
              filterSrv.getBoolFilter(filterSrv.ids())
            )
          ))
        );
      });

      $scope.inspector = request.toJSON();

      results = $scope.ejs.doSearch(dashboard.indices, request);

      results.then(function(results) {
        $scope.panelMeta.loading = false;
        var value = results.facets.stats[$scope.panel.mode];

        var rows = queries.map(function (q) {
          var alias = q.alias || q.query;
          var obj = _.clone(q);
          obj.label = alias;
          obj.Label = alias.toLowerCase(); //sort field
          obj.value = results.facets['stats_'+alias];
          obj.Value = results.facets['stats_'+alias]; //sort field
          return obj;
        });

        $scope.data = {
          value: value,
          rows: rows
        };

        $scope.$emit('render');
      });
    };
```

stats panel 的这段函数几乎就跟基础示例一样了。

1. 生成 Request 对象。
2. 获取关联的 query 对象。
3. 获取当前页的 filter 对象。
4. 调用选定的 facets 方法，传入参数。
5. 如果有多个 query，逐一构建 facets。
6. request 完成。生成一个 JSON 内容供 inspector 查看。
7. 发送请求，等待异步回调。
8. 回调处理数据成绑定在模板上的 `$scope.data`。
9. 渲染页面。

注：stats/module.js 后面还有一个 filter，terms/module.js 后面还有一个 directive，这些都是为了实际页面效果加的功能，跟 kibana 本身的 filter，directive 本质上是一样的。就不单独讲述了。

### module.html

module.html 就是 panel 的具体页面内容。没有太多可说的。大概框架是：

```
<div ng-controller='stats' ng-init="init()">
 <table ng-style="panel.style" class="table table-striped table-condensed" ng-show="panel.chart == 'table'">
    <thead>
      <th>Term</th> <th>{{ panel.tmode == 'terms_stats' ? panel.tstat : 'Count' }}</th> <th>Action</th>
    </thead>
    <tr ng-repeat="term in data" ng-show="showMeta(term)">
      <td class="terms-legend-term">{{term.label}}</td>
      <td>{{term.data[0][1]}}</td>
    </tr>
  </table>
</div>
```

主要就是绑定要 controller 和 init 函数。对于示例的 stats，里面的 `data` 就是 module.js 最后生成的 `$scope.data`。

### editor.html

editor.html 是 panel 参数的编辑页面主要内容，参数编辑还有一些共同的标签页，是在 kibana 的 `app/partials/` 里，就不讲了。

editor.html 里，主要就是提供对 `$scope.panel` 里那些参数的修改保存操作。当然实际上并不是所有参数都暴露出来了。这也是 kibana 3 用户指南里，官方说采用仪表板纲要，比通过页面修改更灵活细腻的原因。

editor.html 里需要注意的是，为了每次变更都能实时生效，所有的输入框都注册到了刷新事件。所以一般是这样子：

```
      <select ng-change="set_refresh(true)" class="input-small" ng-model="panel.format" ng-options="f for f in ['number','float','money','bytes']"></select>
```

这个 `set_refresh` 函数是在 `module.js` 里定义的：

```
    $scope.set_refresh = function (state) {
      $scope.refresh = state;
    };
```

## 总结

kibana 3 源码的主体分析，就是这样了。怎么样，看完以后，大家有没有信心也做些二次开发，甚至跟 grafana 一样，替换掉 esResource，换上一个你自己的后端数据源呢？

