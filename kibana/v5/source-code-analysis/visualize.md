# visualize

index.js 中，首要当然是注册自己。此外，还加载两部分功能：`plugins/kibana/visualize/editor/*` 和 `plugins/kibana/visualize/wizard/wizard.js`。然后定义了一个 route，默认跳转 `/visualize` 到 `/visualize/step/1`。

## editor

editor.js 中也定义了两个 route，分别是 `/visualize/create` 和 `/visualize/edit/:id`。然后还定义了一个controller，叫 `VisEditor`，对应的 HTML 是 `plugins/kibana/visualize/editor/editor.html`，其中用到两个 directive，分别是 `visualize` 和 `vis-editor-sidebar`。

其中 create 是先加载 `ui/registry/vis_types`，并检查 `$route.current.params.type` 是否存在，然后调用 `savedVisualizations.get($route.current.params)` 方法；而 edit 是直接调用 `savedVisualizations.get($route.current.params.id)`。

### vis_types

实际注册了 `vis_types` 的地方包括：

* plugins/table\_vis/index.js
* plugins/metric\_vis/index.js
* plugins/markdown\_vis/index.js
* plugins/kbn\_vislib\_vis\_types/index.js

前三个是表单，最后一个是可视化图。内容如下：

```
import visTypes from 'ui/registry/vis_types';
visTypes.register(require('plugins/kbn_vislib_vis_types/histogram'));
visTypes.register(require('plugins/kbn_vislib_vis_types/line'));
visTypes.register(require('plugins/kbn_vislib_vis_types/pie'));
visTypes.register(require('plugins/kbn_vislib_vis_types/area'));
visTypes.register(require('plugins/kbn_vislib_vis_types/tile_map'));
```

以 histogram 为例解释一下 visTypes。下面的实现较长，我们拆成三部分：

第一部分，加载并生成VislibVisType对象：

```
import VislibVisTypeVislibVisTypeProvider from 'ui/vislib_vis_type/vislib_vis_type';
import VisSchemasProvider from 'ui/vis/schemas';
import histogramTemplate from 'plugins/kbn_vislib_vis_types/editors/histogram.html';

export default function HistogramVisType(Private) {
  const VislibVisType = Private(VislibVisTypeVislibVisTypeProvider);
  const Schemas = Private(VisSchemasProvider);

  return new VislibVisType({
    name: 'histogram',
    title: 'Vertical bar chart',
    icon: 'fa-bar-chart',
    description: 'The goto chart for oh-so-many needs. Great for time and non-time data. Stacked or grouped, ' +
    'exact numbers or percentages. If you are not sure which chart you need, you could do worse than to start here.',
```

第二部分，histogram 可视化所接受的参数默认值以及对应的参数编辑页面：

```
      params: {
        defaults: {
          shareYAxis: true,
          addTooltip: true,
          addLegend: true,
          legendPosition: 'right',
          scale: 'linear',
          mode: 'stacked',
          times: [],
          addTimeMarker: false,
          defaultYExtents: false,
          setYExtents: false,
          yAxis: {}
        },
        scales: ['linear', 'log', 'square root'],
        modes: ['stacked', 'percentage', 'grouped'],
        editor: histogramTemplate
      },
```

第三部分，histogram 可视化能接受的 Schema。一般来说，metric 数值聚合肯定是 Y 轴；bucket 聚合肯定是 X 轴；而在此基础上，Kibana4 还可以让 bucket 有不同效果，也就是 Schema 里的 segment(默认), group 和 split。根据效果不同，这里是各有增减的，比如饼图就不会有 group。

```
      schemas: new Schemas([
        {
          group: 'metrics',
          name: 'metric',
          title: 'Y-Axis',
          min: 1,
          aggFilter: '!std_dev',
          defaults: [
            { schema: 'metric', type: 'count' }
          ]
        },
        {
          group: 'buckets',
          name: 'segment',
          title: 'X-Axis',
          min: 0,
          max: 1,
          aggFilter: '!geohash_grid'
        },
        {
          group: 'buckets',
          name: 'group',
          title: 'Split Bars',
          min: 0,
          max: 1,
          aggFilter: '!geohash_grid'
        },
        {
          group: 'buckets',
          name: 'split',
          title: 'Split Chart',
          min: 0,
          max: 1,
          aggFilter: '!geohash_grid'
        }
      ])
    });
  };
});
```

这里使用的 VislibVisType 类，继承自 `ui/vis/VisType.js`， VisType.js 内容如下：

```
import VisSchemasProvider from 'ui/vis/schemas';

export default function VisTypeFactory(Private) {
  let VisTypeSchemas = Private(VisSchemasProvider);

function VisType(opts) {
  opts = opts || {};

    this.name = opts.name;
    this.title = opts.title;
    this.responseConverter = opts.responseConverter;
    this.hierarchicalData = opts.hierarchicalData || false;
    this.icon = opts.icon;
    this.description = opts.description;
    this.schemas = opts.schemas || new VisTypeSchemas();
    this.params = opts.params || {};
    this.requiresSearch = opts.requiresSearch == null ? true : opts.requiresSearch; // Default to true unless otherwise specified
  }

  return VisType;
};
```

基本跟上面 histogram 的示例一致，注意这里面的 responseConverter 和 hierarchicalData，是给不同的 visType 做相应数据转换的。在实际的 VislibVisType 中，就有下面一段：

```
      if (this.responseConverter == null) {
        this.responseConverter = pointSeries;
      }
``` 

可见默认情况下，Kibana 是尝试把聚合结果转换成点线图数组的。

VislibVisType 中另一部分，则是扩展了一个自己的方法 createRenderbot，用来生成 VislibRenderbot 对象。这个类的实现在 `ui/vislib_vis_type/vislib_renderbot.js`，其中最关键的几行是：

```
import VislibVisTypeBuildChartDataProvider from 'ui/vislib_vis_type/build_chart_data';
module.exports = function VislibRenderbotFactory(Private, $injector) {
  let buildChartData = Private(VislibVisTypeBuildChartDataProvider);
  ...
    self.vislibVis = new vislib.Vis(self.$el[0], self.vislibParams);
  ...
  VislibRenderbot.prototype.buildChartData = buildChartData;
  VislibRenderbot.prototype.render = function (esResponse) {
    this.chartData = this.buildChartData(esResponse);
    return AngularPromise.delay(1).then(() => {
      this.vislibVis.render(this.chartData, this.uiState);
    });
  };
```

也就是说，分为两部分，buildChartData 方法和 vislib.Vis 对象。

先来看 buildChartData 的实现：

```
import AggResponseIndexProvider from 'ui/agg_response/index';
...
    return function (esResponse) {
      let vis = this.vis;
      if (vis.isHierarchical()) {
        return aggResponse.hierarchical(vis, esResponse);
      }
      let tableGroup = aggResponse.tabify(vis, esResponse, {
        canSplit: true,
        asAggConfigResults: true
      });
      let converted = convertTableGroup(vis, tableGroup);
      if (!converted) {
        converted = { rows: []  };
      }
      converted.hits = esResponse.hits.total;
      return converted;
    };
    ....
    function convertTable(vis, table) {
      return vis.type.responseConverter(vis, table);
    }
```

又看到 responseConverter 和 hierarchical 两个熟悉的字眼了，不过这回是另一个对象的方法，那么我们继续跟踪下去，看看这个 aggResponse 类是怎么回事：

```
import AggResponseHierarchicalBuildHierarchicalDataProvider from 'ui/agg_response/hierarchical/build_hierarchical_data';
import AggResponsePointSeriesPointSeriesProvider from 'ui/agg_response/point_series/point_series';
import AggResponseTabifyTabifyProvider from 'ui/agg_response/tabify/tabify';
import AggResponseGeoJsonGeoJsonProvider from 'ui/agg_response/geo_json/geo_json';

export default function NormalizeChartDataFactory(Private) {
  return {
    hierarchical: Private(AggResponseHierarchicalBuildHierarchicalDataProvider),
    pointSeries: Private(AggResponsePointSeriesPointSeriesProvider),
    tabify: Private(AggResponseTabifyTabifyProvider),
    geoJson: Private(AggResponseGeoJsonGeoJsonProvider)
  };
};
```

然后我们看 vislib.Vis 对象，定义在 `ui/public/vislib/vis.js` 里。同时我们注意到，定义 vislib 这个服务的 `ui/public/vislib/index.js` 里，还导入了一个模块，叫 d3，没错，我们离真正的绘图越来越近了。

vis.js 中加载了 `ui/vislib/lib/handler/handler_types` 和 `ui/vislib/visualizations/vis_types`：

```
import VislibLibHandlerHandlerTypesProvider from 'ui/vislib/lib/handler/handler_types';
import VislibVisualizationsVisTypesProvider from 'ui/vislib/visualizations/vis_types';
```

chartTypes 用来定义图：

```
class Vis extends Events {
  constructor($el, config) {
    super(arguments);
    this.el = $el.get ? $el.get(0) : $el;
    this.binder = new Binder();
    this.ChartClass = chartTypes[config.type];
    this._attr = _.defaults({}, config || {}, {
      legendOpen: true
    });
```

接着是 handlerTypes 用来绘制图：

```
    render(data, uiState) {
      var chartType = this._attr.type;
      this.data = data;
      this.handler = handlerTypes[chartType](this) || handlerTypes.column(this);
      this._runWithoutResizeChecker('render');
    };


    _runWithoutResizeChecker(method) {
      this.resizeChecker.stopSchedule();
      this._runOnHandler(method);
      this.resizeChecker.saveSize();
      this.resizeChecker.saveDirty(false);
      this.resizeChecker.continueSchedule();
    };
    _runOnHandler = function (method) {
      this.handler[method]();
    };
```

`ui/vislib/lib/handler/handler_types` 中，根据不同的 vis\_types，分别返回不同的处理对象，主要出自 `ui/vislib/lib/handler/types/point_series`, ` ui/vislib/lib/handler/types/pie ` 和 `ui/vislib/lib/handler/types/tile_map`。比如 histogram 就是 `pointSeries.column`。可以看到 point\_series.js 中，对 column 是加上了 `zeroFill:true, expandLastBucket:true` 两个参数调用 `create()` 方法。而 `create()` 方法里的 `new Handler()` 传递的，显然就是给 d3.js 的绘图参数。而 Handler 具体初始化和渲染过程，则在被加载的 `ui/vislib/lib/handler/handler.js` 中。`Handler.prototype.render` 中如下一段：

```
const selection = d3.select(this.el);
const chartSelection = selection..selectAll('.chart');
chartSelection.each(function (chartData) {
  const chart = new self.ChartClass(self, this, chartData);
  self.vis.activeEvents().forEach(function (event) {
    self.enable(event, chart);
  });

  binder.on(chart.events, 'rendered', () => {
    loadedCount++;
    if (loadedCount === chartSelection.length) {
      charts[0].events.emit('renderComplete');
    }
  });
  charts.push(chart);
  chart.render();
});
```

这里面的 `ChartClass()` 就是在 vislib.js 中加载了的 `ui/vislib/visualizations/vis_types` 。它会根据不同的 vis\_types，分别返回不同的可视化对象，包括：`ui/vislib/visualizations/column_chart`, `ui/vislib/visualizations/pie_chart`, `ui/vislib/visualizations/line_chart`, `ui/vislib/visualizations/area_chart` 和 `ui/vislib/visualizations/tile_map`。

这些对象都有同一个基类：`ui/vislib/visualizations/_chart`，其中有这么一段：

```
    render() {
      const selection = d3.select(this.chartEl);

      selection.selectAll('*').remove();
      selection.call(this.draw());
    };
```

也就是说，各个可视化对象，只需要用 d3.js 或者其他绘图库，完成自己的 **draw()** 函数，就可以了！

draw 函数的实现一般格式，就像下面这段出自 LineChart 的代码：

```
    draw() {
      const self = this;
      const $elem = $(this.chartEl);
      const margin = this._attr.margin;
      const elWidth = this._attr.width = $elem.width();
      const elHeight = this._attr.height = $elem.height();
      const scaleType = this.handler.yAxis.getScaleType();
      const yScale = this.handler.yAxis.yScale;
      const xScale = this.handler.xAxis.xScale;
      const minWidth = 20;
      const minHeight = 20;
      const startLineX = 0;
      const lineStrokeWidth = 1;
      const addTimeMarker = this._attr.addTimeMarker;
      const times = this._attr.times || [];
      let timeMarker;

      return function (selection) {
        selection.each(function (data) {
          const el = this;
          const layers = data.series.map(function mapSeries(d) {
            const label = d.label;
            return d.values.map(function mapValues(e, i) {
              return {
                _input: e,
                label: label,
                x: self._attr.xValue.call(d.values, e, i),
                y: self._attr.yValue.call(d.values, e, i)
              };
            });
          });
          const width = elWidth - margin.left - margin.right;
          const height = elHeight - margin.top - margin.bottom;
          const div = d3.select(el);
          const svg = div.append('svg')
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom)
          .append('g')
          .attr('transform', 'translate(' + margin.left + ',' + margin.top + ')');

          // 处理 data 到 svg 上
          ...
          self.events.emit('rendered', {
            chart: data
          });
          return svg;
        });
      };
    };
```

当然，为了代码逻辑，有些比较复杂的绘制，还是会继续拆分成其他文件的。比如 leaflet 地图，就是在 `ui/vislib/visualizations/tile_map` 里加载的 `ui/vislib/visualizations/_map.js` 完成。

从数据到 d3 渲染，要经过的主要流程就是这样。如果打算自己亲手扩展一个新的可视化方案的读者，可以具体参考我实现的 sankey 图：<https://github.com/chenryn/kibana4/commit/4e0bcbeb4c8fd94807c3a0b1df2ac6f56634f9a5>

### savedVisualizations

这个类在 `core_plugins/kibana/public/visualize/saved_visualizations/saved_visualizations.js` 里定义。其中分三步，加载 `core_plugins/kibana/public/visualize/saved_visualizations/_saved_vis`，注册到 `plugins/kibana/management/saved_object_registry`，以及定义一个 angular service 叫 `savedVisualizations`。

`plugins/kibana/visualize/saved_visualizations/_saved_vis` 里是定义一个 angular factory 叫 `SavedVis`。这个类继承自 **courier.SavedObject**，主要有 `_getLinkedSavedSearch` 方法调用 `savedSearches` 获取在 discover 中保存的 search 对象，以及 visState 属性。该属性保存了 visualize 定义的 JSON 数据。

savedVisualizations 里主要就是初始化 SavedVis 对象，以及提供了一个 find 搜索方法。整个实现和上一节讲的 savedSearches 基本一样，就不再讲了。

### Visualize

这个 directive 在 `ui/visualize/visualize.js` 中定义。而我们可以上拉看到的请求、响应、表格、性能数据，则使用的是 `ui/visualize/spy/spy.js` 中定义的另一个 directive `visualizeSpy`。

visualize.html 上定义了一个普通的 div，其 class 为 **visualize-chart**，在 visualize.js 中，通过 `getter('.visualize-chart')` 方法获取 div 元素：

```
        function getter(selector) {
          return function () {
            let $sel = $el.find(selector);
            if ($sel.size()) return $sel;
          };
        }
        let getVisEl = getter('.visualize-chart');
```

然后创建一个 renderbot：

```
        $scope.$watch('vis', prereq(function (vis, oldVis) {
          let $visEl = getVisEl();
          if (!$visEl) return;

          if (!attr.editableVis) {
            $scope.editableVis = vis;
          }

          if (oldVis) $scope.renderbot = null;
          if (vis) $scope.renderbot = vis.type.createRenderbot(vis, $visEl);
        }));
```

最后在 searchSource 对象变化，即有新的搜索响应返回时，完成渲染：

```
        $scope.$watch('searchSource', prereq(function (searchSource) {
          if (!searchSource || attr.esResp) return;
          searchSource.onResults().then(function onResults(resp) {
            if ($scope.searchSource !== searchSource) return;

            $scope.esResp = resp;

            return searchSource.onResults().then(onResults);
          }).catch(notify.fatal);
          searchSource.onError(notify.error).catch(notify.fatal);
        }));
        $scope.$watch('esResp', prereq(function (resp, prevResp) {
          if (!resp) return;
          $scope.renderbot.render(resp);
        }));
```

### VisEditorSidebar

这个 directive 在 `plugins/kibana/visualize/editor/sidebar.js` 中定义。对应的 HTML 是 `plugins/kibana/visualize/editor/sidebar.html`，其中又用到两个 directive，分别是 `vis-editor-agg-group` 和 `vis-editor-vis-options`。它们分别有 sidebar.js 加载的 `plugins/kibana/visualize/editor/agg_group` 和 `plugins/kibana/visualize/editor/vis_options` 提供。然后继续 HTML -> directive 下去，基本上 `plugins/kibana/visualize/editor/` 目录下那堆 `agg*.js` 和 `agg*.html` 都是做这个用的。

其中比较有意思的，应该算是 `agg_add.js`。我们都知道，K4 最大的特点就是可以层叠子聚合，这个操作就是在这里完成的：

```
import VisAggConfigProvider from 'ui/vis/agg_config';
import uiModules from 'ui/modules';
import aggAddTemplate from 'plugins/kibana/visualize/editor/agg_add.html';

uiModules
.get('kibana')
.directive('visEditorAggAdd', function (Private) {
  const AggConfig = Private(VisAggConfigProvider);

  return {
    restrict: 'E',
    template: aggAddTemplate,
    controllerAs: 'add',
    controller: function ($scope) {
      const self = this;

      self.form = false;
      self.submit = function (schema) {
        self.form = false;

        const aggConfig = new AggConfig($scope.vis, {
          schema: schema
        });
        aggConfig.brandNew = true;
        $scope.vis.aggs.push(aggConfig);
      };
    }
  };
});
```

另一个比较重要的是 `core_plugins/kibana/public/visualize/editor/agg_params.js`。其中加载了 `ui/agg_types/index.js`，又监听了 "agg.type" 变量，也就是实现了选择不同的 agg\_types 时，提供不同的 agg\_params 选项。比方说，选择 date\_histogram，字段就只能是 `@timestamp` 这种 date 类型的字段。

`ui/agg_types/index.js` 中定义了所有可选 agg\_types 的类。其中 metrics 包括：count, avg, sum, min, max, std\_deviation, cardinality, percentiles, percentile\_rank，具体实现分别存在 `ui/agg_types/metrics/` 目录下的*同名.js*文件里；buckets 包括：date\_histogram, histogram, range, date\_range, ip\_range, terms, filters, significant\_terms, geo\_hash，具体实现分别存在 `ui/agg_types/buckets/` 目录下的*同名.js*文件里。

这些类定义中，都有比较类似的格式，其中 params 数组的第一个元素，都是类似这样：

```
{
  name: 'field',
  filterFieldTypes: 'string'
}
```

terms.js 里还多了一行 `scriptable: true`，而且 filterFieldTypes 是数组。

```
        {
          name: 'field',
          scriptable: true,
          filterFieldTypes: ['number', 'boolean', 'date', 'ip',  'string']
        }
```

这个 `filterFieldTypes` 在 `ui/vis/_agg_config.js` 中，通过 `fieldTypeFilter(this.vis.indexPattern.fields, fieldParam.filterFieldTypes);` 得到可选字段列表。fieldTypeFilter 的具体实现在 `filters/filed_type.js` 中。

## wizard

wizard.js 中提供两个 route 和对应的 controller。分别是 `/visualize/step/1` 对应 `VisualizeWizardStep1`，`/visualize/step/2` 对应 `VisualizeWizardStep2`。这两个的最终结果，都是跳转到 `/visualize/create?type=*` 下。
