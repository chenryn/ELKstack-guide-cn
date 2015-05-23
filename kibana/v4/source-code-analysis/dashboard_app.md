# dashboard app

`plugins/dashboard/index.js` 结构跟 visualize 类似，注册到 registry；设置两个调用 `savedDashboards.get()` 方法的 routes，提供一个 directive。

savedDashboards 由 `plugins/dashboard/services/saved_dashboard.js` 提供，同样也是继承 savedObject，主要内容是 `panelsJSON` 数组字段。

dashboard-app 依次往下是 dashboard-grid 和 dashboard-panel 两个 directive。最后通过 `plugins/dashboard/components/panel/lib/load_panel.js` 加载 savedSearch 或者 savedVisualization。
