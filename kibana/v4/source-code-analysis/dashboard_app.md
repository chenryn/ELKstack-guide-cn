# dashboard app

`plugins/dashboard/index.js` 结构跟 visualize 类似，注册到 registry；设置两个调用 `savedDashboards.get()` 方法的 routes，提供一个 directive。

savedDashboards 由 `plugins/dashboard/services/saved_dashboard.js` 提供，同样也是继承 savedObject，主要内容是 `panelsJSON` 数组字段。

dashboard-app 依次往下是 dashboard-grid(在`plugins/dashboard/directives/grid.js`) 和 dashboard-panel(在`plugins/dashboard/components/panel/panel.js`) 两个 directive。最后通过 `plugins/dashboard/components/panel/lib/load_panel.js` 加载 savedSearch 或者 savedVisualization。获得的对象，传递给 `plugins/dashboard/components/panel/panel.html` 里的 visualize 和 doc-table 两个 directive。这两个正是之前在 visualize 和 discover 插件解析里提到过的，在 `components/` 底下实现。
