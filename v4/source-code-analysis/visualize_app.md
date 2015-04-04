# visualize app

index.js 中，首要当然是注册自己。此外，还加载两部分功能：`plugins/visualize/editor/editor.js` 和 `plugins/visualize/wizard/wizard.js`。然后定义了一个 route，默认跳转 `/visualize` 到 `/visualize/step/1`。

## editor

editor.js 中也定义了两个 route，分别是 `/visualize/create` 和 `/visualize/edit/:id`。然后还定义了一个controller，叫 `VisEditor`，对应的 HTML 是 `plugins/visualize/editor/editor.html`，其中用到两个 directive，分别是 `visualize` 和 `vis-editor-sidebar`。

其中 create 是先加载 `registry/vis_types`，并检查 `$route.current.params.type` 是否存在，然后调用 `savedVisualizations.get($route.current.params)` 方法；而 edit 是直接调用 `savedVisualizations.get($route.current.params.id)`。

### vis_types

这部分内容在 `plugins/vis_types/index.js` 里加载，可以看到目前有 histogram, line, pie, area, tile_map。以 histogram 为例：

` plugins/vis_types/vislib/histogram.js` 首先加载 `plugins/vis_types/vislib/_vislib_vis_type` 和 `plugins/vis_types/_schemas`，这些都是规范整个 vis_types 的数据格式的；然后 _vislib_vis_type.js 中加载了 `plugins/vis_types/vislib/_vislib_renderbot`，这里面又加载 `plugins/vis_types/vislib/_build_chart_data.js`，build 出来的数据就可以交给 `VislibRenderbot. vislibVis.render()` 方法渲染绘图了。

这个 vislibVis 是一个 vislib.Vis 对象，定义在 `components/vislib/vislib.js` 里。其中加载了 `components/vislib/lib/handler/handler_types` 和 `components/vislib/visualizations/vis_types`。

`components/vislib/lib/handler/handler_types` 中，根据不同的 vis_types，分别返回不同的处理对象，主要出自 `components/vislib/lib/handler/types/point_series`, ` components/vislib/lib/handler/types/pie ` 和 `components/vislib/lib/handler/types/tile_map`。比如 histogram 就是 `pointSeries.column`。可以看到 point_series.js 中，对 column 是加上了 `zeroFill:true, expandLastBucket:true` 两个参数调用 `create()` 方法。而 `create()` 方法里的 `new Handler()` 传递的，显然就是给 d3.js 的绘图参数。而 Handler 具体初始化和渲染过程，则在被加载的 `components/vislib/lib/handler/handler.js` 中。`Handler.prototype.render` 中如下一段：

```
d3.select(this.el)
.selectAll('.chart')
.each(function (chartData) {
  var chart = new self.ChartClass(self, this, chartData);
  var enabledEvents;
  if (chart.events.dispatch) {
    enabledEvents = self.vis.eventTypes.enabled;
    d3.rebind(chart, chart.events.dispatch, 'on');
    if (enabledEvents.length) {
      enabledEvents.forEach(function (event) {
        self.enable(event, chart);
      });
    }
  }
  charts.push(chart);
  chart.render();
  });
};
```

这里面的 `ChartClass()` 就是在 vislib.js 中加载了的 `components/vislib/visualizations/vis_types` 。它会根据不同的 vis_types，分别返回不同的可视化对象，包括：`components/vislib/visualizations/column_chart`, `components/vislib/visualizations/pie_chart`, `components/vislib/visualizations/line_chart`, `components/vislib/visualizations/area_chart` 和 `components/vislib/visualizations/tile_map`。比如之前已经在 v3/bettermap 章节介绍过的 leaflet 地图，在 v4 中，就是在 `components/vislib/visualizations/tile_map` 完成实际绘制的。还想更换高德地图的读者，修改该文件中 `tileLayer` 变量的参数定义(*https://otile{s}-s.mqcdn.com/tiles/1.0.0/map/{z}/{x}/{y}.jpeg*)即可。

### savedVisualizations

这个类在 `plugins/visualize/saved_visualizations/saved_visualizations.js` 里定义。其中分三步，加载 `plugins/visualize/saved_visualizations/_saved_vis`，注册到 `plugins/settings/saved_object_registry`，以及定义一个 angular service 叫 `savedVisualizations`。

`plugins/visualize/saved_visualizations/_saved_vis` 里是定义一个 angular factory 叫 `SavedVis`。这个类继承自 **courier.SavedObject**，主要有 `_getLinkedSavedSearch` 方法调用 `savedSearches` 获取在 discover 中保存的 search 对象，以及 visState 属性。该属性保存了 visualize 定义的 JSON 数据。

savedVisualizations 里主要就是初始化 SavedVis 对象，以及提供了一个 find 搜索方法。

### Visualize

这个 directive 在 `components/visualize/visualize.js` 中定义。而我们可以上拉看到的请求、响应、表格、性能数据，则使用的是 `components/visualize/spy/spy.js` 中定义的另一个 directive `visualizeSpy`。

真正画图的地方，反而没有用 directive (kibana 3 里用了)，而是定义了一个普通的 div，其 class 为 **visualize-chart**，在 visualize.js 中，通过 `getter('.visualize-chart')` 方法获取 div 元素，然后通过 `$scope.renderbot = vis.type.createRenderbot(vis, $visEl);` 创建一个 renderbot，再在 `$scope.$watch('esResp',function(){})` 里头，调用 `$scope.renderbot.render(resp);` 完成渲染。

### VisEditorSidebar

这个 directive 在 `plugins/visualize/editor/sidebar.js` 中定义。对应的 HTML 是 `plugins/visualize/editor/sidebar.html`，其中又用到两个 directive，分别是 `vis-editor-agg-group` 和 `vis-editor-vis-options`。它们分别有 sidebar.js 加载的 `plugins/visualize/editor/agg_group` 和 `plugins/visualize/editor/vis_options` 提供。然后继续 HTML -> directive 下去，基本上 `plugins/visualize/editor/` 目录下那堆 `agg*.js` 和 `agg*.html` 都是做这个用的。

这其中，比较重要的是 `plugins/visualize/editor/agg_params.js`。其中加载了 `components/agg_types/index.js`，又监听了 "agg.type" 变量，也就是实现了选择不同的 agg_types 时，提供不同的 agg_params 选项。比方说，选择 date_histogram，字段就只能是 `@timestamp` 这种 date 类型的字段。

`components/agg_types/index.js` 中定义了所有可选 agg_types 的类。其中 metrics 包括：count, avg, sum, min, max, std_deviation, cardinality, percentiles, percentile_rank，具体实现分别存在 `components/agg_types/metrics/` 目录下的*同名.js*文件里；buckets 包括：date_histogram, histogram, range, date_range, ip_range, terms, filters, significant_terms, geo_hash，具体实现分别存在 `components/agg_types/buckets/` 目录下的*同名.js*文件里。

这些类定义中，都有比较类似的格式，其中 params 数组的第一个元素，都是类似这样：

```
{
  name: 'field',
  filterFieldTypes: 'string'
}
```

*注： terms.js 里还多了一行 `scriptable: true`，而且 filterFieldTypes 是数组。*

这个 `filterFieldTypes` 在 `components/vis/_agg_config.js` 中，通过 `fieldTypeFilter(this.vis.indexPattern.fields, fieldParam.filterFieldTypes);` 得到可选字段列表。fieldTypeFilter 的具体实现在 `filters/filed_type.js` 中。

## wizard

wizard.js 中提供两个 route 和对应的 controller。分别是 `/visualize/step/1` 对应 `VisualizeWizardStep1`，`/visualize/step/2` 对应 `VisualizeWizardStep2`。这两个的最终结果，都是跳转到 `/visualize/create?type=*` 下。
