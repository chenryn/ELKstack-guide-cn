# dashboard

`plugins/kibana/public/dashboard/index.js` 结构跟 visualize 类似，设置两个调用 `savedDashboards.get()` 方法的 routes，提供一个叫 dashboard-app 的 directive。

savedDashboards 由 `plugins/kibana/public/dashboard/services/saved_dashboard.js` 提供，调用 es.search 获取数据，生成 savedDashboard 对象，这个对象同样也是继承 savedObject，主要内容是 `panelsJSON` 数组字段。实现如下：

```
  module.factory('SavedDashboard', function (courier) {
    _.class(SavedDashboard).inherits(courier.SavedObject);
    function SavedDashboard(id) {
      SavedDashboard.Super.call(this, {
        type: SavedDashboard.type,
        mapping: SavedDashboard.mapping,
        searchSource: SavedDashboard.searchsource,
        id: id,
        defaults: {
          title: 'New Dashboard',
          hits: 0,
          description: '',
          panelsJSON: '[]',
          optionsJSON: angular.toJson({
            darkTheme: config.get('dashboard:defaultDarkTheme')
          }),
          uiStateJSON: '{}',
          version: 1,
          timeRestore: false,
          timeTo: undefined,
          timeFrom: undefined,
          refreshInterval: undefined
        },
        clearSavedIndexPattern: true
      });
    }
```

可以注意到，这个 panelsJSON 是一个字符串，这跟之前 kbnIndex 提到的是一致的。

dashboard-app 中，最重要的功能，是监听搜索框和过滤条件的变更，我们可以看到 init 函数中有下面这段：

```
        function updateQueryOnRootSource() {
          var filters = queryFilter.getFilters();
          if ($state.query) {
            dash.searchSource.set('filter', _.union(filters, [{
              query: $state.query
            }]));
          } else {
            dash.searchSource.set('filter', filters);
          }
        }

        $scope.$listen(queryFilter, 'update', function () {
          updateQueryOnRootSource();
          $state.save();
        });
```

在 index.html 里，实际承载面板的，是下面这行：

```
  <dashboard-grid></dashboard-grid>
```

这也是一个 angular directive，通过加载 `plugins/kibana/public/dashboard/directives/grid.js` 引入的。其中添加面板相关的代码有两部分：

```
          $scope.$watchCollection('state.panels', function (panels) {
            const currentPanels = gridster.$widgets.toArray().map(function (el) {
              return getPanelFor(el);
            });
            const removed = _.difference(currentPanels, panels);
            const added = _.difference(panels, currentPanels);
            if (added.length) added.forEach(addPanel);
            ...
```

这段用来监听 `$state.panels` 数组，一旦有新增面板，调用 `addPanel` 函数。同理也有删除面板的，这里就不重复贴了。

而 addPanel 函数的实现大致如下：

```
        var addPanel = function (panel) {
          _.defaults(panel, {
            size_x: 3,
            size_y: 2
          });
          ...
          panel.$scope = $scope.$new();
          panel.$scope.panel = panel;
          panel.$el = $compile('<li><dashboard-panel></li>')(panel.$scope);
          gridster.add_widget(panel.$el, panel.size_x, panel.size_y, panel.col, panel.row);
          ...
        };
```

这里即验证了之前 kbnIndex 小节中讲的 gridster 默认大小，又引入了一个新的 directive，叫 dashboard-panel。

dashboard-panel 在 `plugins/kibana/public/dashboard/components/panel/panel.js` 中实现，其中使用了 `plugins/kibana/public/dashboard/components/panel/panel.html` 页面。页面最后是这么一段：

```
 <visualize ng-switch-when="visualization"
    vis="savedObj.vis"
    search-source="savedObj.searchSource"
    class="panel-content">
  </visualize>

  <doc-table ng-switch-when="search"
    search-source="savedObj.searchSource"
    sorting="panel.sort"
    columns="panel.columns"
    class="panel-content"
    filter="filter">
  </doc-table>
```

这里使用的 savedObj 对象，来自 `plugins/kibana/public/dashboard/components/panel/lib/load_panel.js` 获取的 savedSearch 或者 savedVisualization。获得的对象，以 savedVisualization 为例：

```
define(function (require) {
  return function visualizationLoader(savedVisualizations, Private) { // Inject services here
    return function (panel, $scope) {
      return savedVisualizations.get(panel.id)
        .then(function (savedVis) {
          savedVis.vis.listeners.click = filterBarClickHandler($scope.state);
          savedVis.vis.listeners.brush = brushEvent;

          return {
            savedObj: savedVis,
            panel: panel,
            editUrl: savedVisualizations.urlFor(panel.id)
          };
        });
    };
  };
});
```

而 visualize 和 doc-table 两个 directive。这两个正是之前在 visualize 和 discover 插件解析里提到过的，在 `components/` 底下实现。
