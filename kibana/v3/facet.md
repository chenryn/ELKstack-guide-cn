# range facet 面板开发

查看响应时间在不同区间内占比的是非常常见的一个监控和 SLA 需求。Elasticsearch 对此有直接的接口支持。考虑到 kibana3 内大多数还是用 facet 接口，这里也沿用：<http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/search-facets-range-facet.html>。

range facet 本身的使用非常简单，就像官网示例那样，直接 curl 命令就可以完成调试：

```
curl -XPOST http://localhost:9200/logstash-2014.08.18/_search?pretty=1 -d '{
    "query" : {
        "match_all" : {}
    },
    "facets" : {
        "range1" : {
            "range" : {
                "field" : "resp_ms",
                "ranges" : [
                    { "to" : 100 },
                    { "from" : 101, "to" : 500 },
                    { "from" : 500 }
                ]
            }
        }
    }
}'
```

不过在 kibana 里，我们就不要再自己拼 JSON 发请求了。elastic.js 关于 range facet 的文档见：<http://docs.fullscale.co/elasticjs/ejs.RangeFacet.html>

因为 range facet 本身比较简单，所以 RangeFacet 对象支持的方法也比较少。一个 `addRange` 方法添加 ranges 数组，一个 `field` 方法添加 field 名称即可。

## 代码实现难点

面板代码的主要层次和方法，在上节中已经讲过。在二次开发中，完全可以复制一个类似的现有 panel ，然后开始编辑。比如本节预备开发一个以饼图展示区间统计的面板，即可复制 terms 面板代码：

```
# cp -r src/app/panels/terms src/app/panels/ranges
# sed -i 's/terms/ranges/g' src/app/panels/ranges/*
```

terms 面板中，设计有 fmode 和 tmode ，分别控制 terms 和 term_stats 时的参数情况，而 ranges 面板中并不需要，所以去除这部分属性，在 `module.js` 中，有关 tmode 的内容更加简单。

### 构建请求

```
      if($scope.panel.tmode === 'ranges') {
        rangefacet = $scope.ejs.RangeFacet('ranges');
        // AddRange
        _.each($scope.panel.values, function(v) {
          rangefacet.addRange(v.from, v.to);
        });
        request = request
          .facet(rangefacet
          .field($scope.field)
          .facetFilter($scope.ejs.QueryFilter(
            $scope.ejs.FilteredQuery(
              boolQuery,
              filterSrv.getBoolFilter(filterSrv.ids())
            )))).size(0);
      }
```

### 组织结果

```
        function build_results() {
          var k = 0;
          scope.data = [];
          _.each(scope.results.facets.ranges.ranges, function(v) {
            var slice;
            if(scope.panel.tmode === 'ranges') {
              slice = { label : [v.from,v.to], data : [[k,v.count]], actions: true};
            }
            scope.data.push(slice);
            k = k + 1;
          });

          scope.data.push({label:'Missing field',
            data:[[k,scope.results.facets.ranges.missing]],meta:"missing",color:'#aaa',opacity:0});

          if(scope.panel.tmode === 'ranges') {
            scope.data.push({label:'Other values',
              data:[[k+1,scope.results.facets.ranges.other]],meta:"other",color:'#444'});
          }
        }
```

### 区间范围配置编辑

这个新 panel 的实现，更复杂的地方在配置编辑上如何让 range 范围值支持自定义添加和填写。对此，设计有一个 `$scope.panel.values` 数组，对应每个区间：

* values
    用于计算 facet 的数值范围数组。数组每个元素包括：
    * from
        range 范围的起始点
    * to
        range 范围的结束点

`editor.html` 里 Field 栏由普通的文本输入框改成表格输入：

```
      <div class="editor-option">
        <label class="small">Field</label>
        <input type="text" class="input-small" bs-typeahead="fields.list" ng-model="panel.field" ng-change="set_refresh(true)">
      </div>
      <div class="editor-row">
        <table class="table table-condensed table-striped">
          <thead>
            <tr>
              <th>From</th>
              <th>To</th>
              <th ng-show="panel.values.length > 1">Delete</th>
            </tr>
          </thead>
          <tbody>
            <tr ng-repeat="value in panel.values">
              <td>
                <div class="editor-option">
                  <input class="input-small" type="number" ng-model="value.from" ng-change="set_refresh(true)">
                </div>
              </td>
              <td>
                <div class="editor-option">
                  <input class="input-small" type="number" ng-model="value.to" ng-change="set_refresh(true)">
                </div>
              </td>
              <td ng-show="panel.values.length > 1">
                <i ng-click="panel.values = _.without(panel.values, value);set_refresh(true)" class="pointer icon-remove"></i>
              </td>
            </tr>
          </tbody>
        </table>
        <button type="button" class="btn btn-success" ng-click="add_new_value(panel);set_refresh(true)"><i class="icon-plus-sign"></i> Add value</button>
      </div>
    </div>
```

这里使用了一个 `add_new_value` 函数，需要在 `module.js` 中定义：

```
    $scope.defaultValue = {
      'from': 0,
      'to'  : 100
    };
    $scope.add_new_value = function(panel) {
      panel.values.push(angular.copy($scope.defaultValue));
    };
```

### 面板单击生成的 filtering 条件

另一个需要注意的地方是饼图出来以后，单击饼图区域，自动生成的 `filterSrv` 内容。一般的面板这里都是 `terms` 类型的 `filterSrv`，传递的是面板的 label 值。而我们这里 label 值显然不是 ES 有效的 terms 语法，还好 `filterSrv` 有 `range` 类型(histogram 面板的 `time` 类型的 `filterSrv` 是在 daterange 基础上实现的)，所以稍微修改就可以了。

```
    $scope.build_search = function(range,negate) {
      if(_.isUndefined(range.meta)) {
        filterSrv.set({type:'range',field:$scope.field,from:range.label[0],to:range.label[1],
          mandate:(negate ? 'mustNot':'must')});
      } else if(range.meta === 'missing') {
        filterSrv.set({type:'exists',field:$scope.field,
          mandate:(negate ? 'must':'mustNot')});
      } else {
        return;
      }
    };
```

## 面板效果

最终效果如下：

![](https://github.com/chenryn/kibana/raw/master/src/img/chenryn_img/range-panel.jpg)

属性编辑界面效果如下：

![](https://github.com/chenryn/kibana/raw/master/src/img/chenryn_img/range-setting.jpg)

