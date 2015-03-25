# 搜索页

`plugins/discover/index.js` 中主要就是注册自己的 id, name, order 到上节最后说的 `registry.apps` 里。此外就是加载本 app 目录内的其他文件。依次说明如下：

## plugins/discover/saved_searches/saved_searches.js

* 定义 savedSearches 这个 angular service，用来操作 kibana_index 索引里 search 这个类型下的数据；
* 加载了 `saved_searches/_saved_searches.js` 提供的 savedSearch 这个 angular factory，这里定义了一个搜索 (search) 在 `kibana_index` 里的数据结构，包括 title, description, hits, column, sort, version 等字段(这部分内容，可以直接通过读取 Elasticsearch 中的索引内容看到，比阅读代码更直接，本章最后即专门介绍 `kibana_index` 中的数据结构)，然后用前面提到的 `components/couries/saved_object/saved_object.js` 跟索引交互；
* 还加载并注册了 `plugins/settings/saved_object_registry.js`，表示可以在 settings 里修改这里的 savedSearches 对象。

## plugins/discover/directives/timechart.js

* 加载 `components/vislib/index.js`。
* 提供 discoverTimechart 这个 angular directive，监听 "data" 并调用 `vislib.Chart` 对象绘图。

## plugins/discover/ components/field_chooser/field_chooser.js

* 提供 discFieldChooser 这个 angular directive，其中监听 "fields" 并调用 calculateFields 计算常用字段排行，监听 "data" 并调用 `$scope.details()` 方法，提供 `$scope.runAgg()` 方法。方法中，根据字段的类型不同，分别可能使用 `date_histogram`/`geohash_grid`/`terms` 聚合函数，创建可视化模型，然后带着当前页这些设定(前面说过，各 app 之间通过 sessionStorage 共享了状态的)跳转到 "/visualize/create" 页面，相当于是这三个常用聚合的快速可视化操作。
* 加载 `plugins/discover/components/field_chooser/lib/field_calculator.js` ，提供 `fieldCalculator.getFieldValueCounts()` 方法，在 `$scope.details()` 中读取被点击的字段值的情况。
* 加载 `plugins/discover/components/field_chooser/discover_field.js`，提供 discoverField 这个 angular directive，用于弹出浮层展示零时的 visualize(调用上一条提供的 `$scope.details()` 方法)，同时给被点击的字段加常用度；加载 `plugins/discover/components/field_chooser/lib/detail_views/string.html` 网页，用于浮层效果。网页中对 indexed 或 scripted 类型的字段，可以调用前面提到的 `runAgg()` 方法。
* 加载并渲染 `plugins/discover/components/field_chooser/field_chooser.html` 网页。网页中使用了上一条提供的 discover-field 标签。

## plugins/discover/controllers/discover.js

加载了诸多 js，主要做了：

* 为 "/discover/:id" 提供 route 并加载 `plugins/discover/index.html` 网页。
* 提供 discover 这个 angular controller。
* 加载 `components/vis/vis.js` 并在 setupVisualization 函数中绘制 histogram 图。
* 加载 `components/filter_manager/filter_manager.js` ，根据字段类型生成不同的 filter 语句，存在全局 state 里。
* 加载 `components/doc_table/lib/get_sort.js`(存疑：docTable 这个 directive 是在哪里加载的？)
