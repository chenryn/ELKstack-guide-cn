# 搜索页

前文已经说到，kibana.js 中依次加载了各主要功能模块的入口。比如搜索页是 `src/core_plugins/kibana/public/discover/index.js`。通过这个文件路径就可以猜到，有关搜索页的功能，代码应该都在 `src/core_plugins/kibana/public/discover/` 里了。这个目录下的文件有：

* \_hit\_sort\_fn.js
* components/
* controllers/
* directives/
* index.html
* index.js
* partials/
* saved\_searches/
* styles/

这也是一个比较标准的 angular 模块的目录结构了。一眼就能知道，controller、directive 等等分别应该进哪里去看。当然首先第一步还是看 index.js：

```
import 'plugins/kibana/discover/saved_searches/saved_searches';
import 'plugins/kibana/discover/directives/no_results';
import 'plugins/kibana/discover/directives/timechart';
import 'ui/collapsible_sidebar';
import 'plugins/kibana/discover/components/field_chooser/field_chooser';
import 'plugins/kibana/discover/controllers/discover';
import 'plugins/kibana/discover/styles/main.less';
import 'ui/doc_table/components/table_row';
import savedObjectRegistry from 'ui/saved_objects/saved_object_registry';

savedObjectRegistry.register(require('plugins/kibana/discover/saved_searches/saved_search_register'));
```

已存搜索、事件数趋势图、事件列表、字段列表，各自载入了。下面可以看一下这几个功能点的实现。

## plugins/kibana/discover/saved_searches/saved_searches.js

* 定义 savedSearches 这个 angular service，用来操作 kbnIndex 索引里 search 这个类型下的数据；
* 加载了 `saved_searches/_saved_searches.js` 提供的 savedSearch 这个 angular factory，这里定义了一个搜索 (search) 在 `kbnIndex` 里的数据结构，包括 title, description, hits, column, sort, version 等字段(这部分内容，可以直接通过读取 Elasticsearch 中的索引内容看到，比阅读代码更直接，本章最后即专门介绍 `kbnIndex` 中的数据结构)，看的是不是有点眼熟？没错，这个 savedSearch 就是继承了上一节我们介绍的那个 courier 的 savedObject：

```
  module.factory('SavedSearch', function (courier) {
    _.class(SavedSearch).inherits(courier.SavedObject);
    function SavedSearch(id) {
      ...
```

* 还加载并注册了 `plugins/kibana/management/saved_object_registry`，表示可以在 management 里修改这里的 savedSearches 对象。

## plugins/kibana/discover/directives/timechart.js

* 加载 `ui/vislib`。
* 提供 discoverTimechart 这个 angular directive，监听 "data" 并调用 `vislib.Chart` 对象绘图。

vislib 是整个 Kibana 可视化的实现部分，下一节会更详细的讲述。

## plugins/kibana/discover/components/field_chooser/field_chooser.js

* 提供 discFieldChooser 这个 angular directive，其中监听字段和事件的变化，并调用计算常用字段排行，

```
        $scope.$watchMulti([
          '[]fieldCounts',
          '[]columns',
          '[]hits'
        ], function (cur, prev) {
          var newHits = cur[2] !== prev[2];
          var fields = $scope.fields;
          var columns = $scope.columns || [];
          var fieldCounts = $scope.fieldCounts;

          if (!fields || newHits) {
            $scope.fields = fields = getFields();
          }

          _.chain(fields)
          .each(function (field) {
            field.displayOrder = _.indexOf(columns, field.name) + 1;
            field.display = !!field.displayOrder;
            field.rowCount = fieldCounts[field.name];
          })
          .sortBy(function (field) {
            return (field.count || 0) * -1;
          })
          .groupBy(function (field) {
            if (field.display) return 'selected';
            return field.count > 0 ? 'popular' : 'unpopular';
          })
          .tap(function (groups) {
            groups.selected = _.sortBy(groups.selected || [], 'displayOrder');
            groups.popular = groups.popular || [];
            groups.unpopular = groups.unpopular || [];
            var extras = groups.popular.splice(config.get('fields:popularLimit'));
            groups.unpopular = extras.concat(groups.unpopular);
          })
          .each(function (group, name) {
            $scope[name + 'Fields'] = _.sortBy(group, name === 'selected' ? 'display' : 'name');
          })
          .commit();

          $scope.fieldTypes = _.union([undefined], _.pluck(fields, 'type'));
        });
```

监听 "data" 并调用 `$scope.details()` 方法，

提供 `$scope.vizLocation()` 方法。方法中，根据字段的类型不同，分别可能使用 `date_histogram`/`geohash_grid`/`terms` 聚合函数，创建可视化模型，然后带着当前页这些设定——前面说过，各 app 之间通过 globalState 共享状态，也就是 URL 中的 `?_a=(...)`。各 app 会通过 `rison.decode($location.search()._a)` 和 `rison.encode($location.search()._a)` 设置和读取——跳转到 "/visualize/create" 页面，相当于是这三个常用聚合的快速可视化操作。

默认的 create 页的 rison 如下：

```
          return '#/visualize/create?' + $.param(_.assign($location.search(), {
            indexPattern: $scope.state.index,
            type: type,
            _a: rison.encode({
              filters: $scope.state.filters || [],
              query: $scope.state.query || undefined,
              vis: {
                type: type,
                aggs: [
                  agg,
                  {schema: 'metric', type: 'count', 'id': '2'}
                ]
              }
            })
          });
```

之前章节的 url 示例中，读者如果注意的话，会发现 id 是从 2 开始的，原因即在此。

* 加载 `plugins/kibana/discover/components/field_chooser/lib/field_calculator.js` ，提供 `fieldCalculator.getFieldValueCounts()` 方法，在 `$scope.details()` 中读取被点击的字段值的情况。
* 加载 `plugins/kibana/discover/components/field_chooser/discover_field.js`，提供 discoverField 这个 angular directive，用于弹出浮层展示临时的 visualize(调用上一条提供的 `$scope.details()` 方法)，同时给被点击的字段加常用度；加载 `plugins/kibana/discover/components/field_chooser/lib/detail_views/string.html` 网页，用于浮层效果。网页中对 indexed 或 scripted 类型的字段，可以调用前面提到的 `vizLocation()` 方法。

```
        import detailsHtml from 'plugins/kibana/discover/components/field_chooser/lib/detail_views/string.html';
        $scope.toggleDetails = function (field, recompute) {
          if (_.isUndefined(field.details) || recompute) {
            // This is inherited from fieldChooser
            $scope.details(field, recompute);
            detailScope.$destroy();
            detailScope = $scope.$new();
            detailScope.warnings = getWarnings(field);

            detailsElem = $(detailsHtml);
            $compile(detailsElem)(detailScope);
            $elem.append(detailsElem).addClass('active');
          } else {
            delete field.details;
            detailsElem.remove();
            $elem.removeClass('active');
          }
        };
```

* 加载并渲染 `plugins/kibana/discover/components/field_chooser/field_chooser.html` 网页。网页中使用了上一条提供的 discover-field。

## plugins/kibana/discover/controllers/discover.js

加载了诸多 js，主要做了：

* 为 "/discover/:id?" 提供 route 并加载 `plugins/discover/index.html` 网页。
* 提供 discoverApp 这个 angular controller。
* 加载 `ui/vis.js` 并在 setupVisualization 函数中绘制 histogram 图。

```
      var visStateAggs = [
        {
          type: 'count',
          schema: 'metric'
        },
        {
          type: 'date_histogram',
          schema: 'segment',
          params: {
            field: $scope.opts.timefield,
            interval: $state.interval
          }
        }
      ];

      $scope.vis = new Vis($scope.indexPattern, {
        title: savedSearch.title,
        type: 'histogram',
        params: {
          addLegend: false,
          addTimeMarker: true
        },
        listeners: {
          click: function (e) {
            timefilter.time.from = moment(e.point.x);
            timefilter.time.to = moment(e.point.x + e.data.ordered.interval);
            timefilter.time.mode = 'absolute';
          },
          brush: brushEvent
        },
        aggs: visStateAggs
      });

      $scope.searchSource.aggs(function () {
        $scope.vis.requesting();
        return $scope.vis.aggs.toDsl();
      });
```
