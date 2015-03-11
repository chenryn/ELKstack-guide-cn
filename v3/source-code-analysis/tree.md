## 源码目录结构

下面是 kibana 源码的全部文件的 tree 图：

```
.
├── app
│   ├── app.js
│   ├── components
│   │   ├── extend-jquery.js
│   │   ├── kbn.js
│   │   ├── lodash.extended.js
│   │   ├── require.config.js
│   │   └── settings.js
│   ├── controllers
│   │   ├── all.js
│   │   ├── dash.js
│   │   ├── dashLoader.js
│   │   ├── pulldown.js
│   │   └── row.js
│   ├── dashboards
│   │   ├── blank.json
│   │   ├── default.json
│   │   ├── guided.json
│   │   ├── logstash.js
│   │   ├── logstash.json
│   │   ├── noted.json
│   │   ├── panel.js
│   │   └── test.json
│   ├── directives
│   │   ├── addPanel.js
│   │   ├── all.js
│   │   ├── arrayJoin.js
│   │   ├── configModal.js
│   │   ├── confirmClick.js
│   │   ├── dashUpload.js
│   │   ├── esVersion.js
│   │   ├── kibanaPanel.js
│   │   ├── kibanaSimplePanel.js
│   │   ├── ngBlur.js
│   │   ├── ngModelOnBlur.js
│   │   ├── resizable.js
│   │   └── tip.js
│   ├── factories
│   │   └── store.js
│   ├── filters
│   │   └── all.js
│   ├── panels
│   │   ├── bettermap
│   │   │   ├── editor.html
│   │   │   ├── leaflet
│   │   │   │   ├── images
│   │   │   │   │   ├── layers-2x.png
│   │   │   │   │   ├── layers.png
│   │   │   │   │   ├── marker-icon-2x.png
│   │   │   │   │   ├── marker-icon.png
│   │   │   │   │   └── marker-shadow.png
│   │   │   │   ├── leaflet-src.js
│   │   │   │   ├── leaflet.css
│   │   │   │   ├── leaflet.ie.css
│   │   │   │   ├── leaflet.js
│   │   │   │   ├── plugins.css
│   │   │   │   ├── plugins.js
│   │   │   │   └── providers.js
│   │   │   ├── module.css
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── column
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   ├── module.js
│   │   │   └── panelgeneral.html
│   │   ├── dashcontrol
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── derivequeries
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── fields
│   │   │   ├── editor.html
│   │   │   ├── micropanel.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── filtering
│   │   │   ├── editor.html
│   │   │   ├── meta.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── force
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── goal
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── histogram
│   │   │   ├── editor.html
│   │   │   ├── interval.js
│   │   │   ├── module.html
│   │   │   ├── module.js
│   │   │   ├── queriesEditor.html
│   │   │   ├── styleEditor.html
│   │   │   └── timeSeries.js
│   │   ├── hits
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── map
│   │   │   ├── editor.html
│   │   │   ├── lib
│   │   │   │   ├── jquery.jvectormap.min.js
│   │   │   │   ├── map.cn.js
│   │   │   │   ├── map.europe.js
│   │   │   │   ├── map.usa.js
│   │   │   │   └── map.world.js
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── multifieldhistogram
│   │   │   ├── editor.html
│   │   │   ├── interval.js
│   │   │   ├── markersEditor.html
│   │   │   ├── meta.html
│   │   │   ├── module.html
│   │   │   ├── module.js
│   │   │   ├── styleEditor.html
│   │   │   └── timeSeries.js
│   │   ├── percentiles
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── query
│   │   │   ├── editor.html
│   │   │   ├── editors
│   │   │   │   ├── lucene.html
│   │   │   │   ├── regex.html
│   │   │   │   └── topN.html
│   │   │   ├── help
│   │   │   │   ├── lucene.html
│   │   │   │   ├── regex.html
│   │   │   │   └── topN.html
│   │   │   ├── helpModal.html
│   │   │   ├── meta.html
│   │   │   ├── module.html
│   │   │   ├── module.js
│   │   │   └── query.css
│   │   ├── ranges
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── sparklines
│   │   │   ├── editor.html
│   │   │   ├── interval.js
│   │   │   ├── module.html
│   │   │   ├── module.js
│   │   │   └── timeSeries.js
│   │   ├── statisticstrend
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── stats
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── table
│   │   │   ├── editor.html
│   │   │   ├── export.html
│   │   │   ├── micropanel.html
│   │   │   ├── modal.html
│   │   │   ├── module.html
│   │   │   ├── module.js
│   │   │   └── pagination.html
│   │   ├── terms
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── text
│   │   │   ├── editor.html
│   │   │   ├── lib
│   │   │   │   └── showdown.js
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   ├── timepicker
│   │   │   ├── custom.html
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   ├── module.js
│   │   │   └── refreshctrl.html
│   │   ├── trends
│   │   │   ├── editor.html
│   │   │   ├── module.html
│   │   │   └── module.js
│   │   └── valuehistogram
│   │       ├── editor.html
│   │       ├── module.html
│   │       ├── module.js
│   │       ├── queriesEditor.html
│   │       └── styleEditor.html
│   ├── partials
│   │   ├── connectionFailed.html
│   │   ├── dashLoader.html
│   │   ├── dashLoaderShare.html
│   │   ├── dashboard.html
│   │   ├── dasheditor.html
│   │   ├── inspector.html
│   │   ├── load.html
│   │   ├── modal.html
│   │   ├── paneladd.html
│   │   ├── paneleditor.html
│   │   ├── panelgeneral.html
│   │   ├── querySelect.html
│   │   └── roweditor.html
│   └── services
│       ├── alertSrv.js
│       ├── all.js
│       ├── dashboard.js
│       ├── esVersion.js
│       ├── fields.js
│       ├── filterSrv.js
│       ├── kbnIndex.js
│       ├── monitor.js
│       ├── panelMove.js
│       ├── querySrv.js
│       └── timer.js
├── config.js
├── css
│   ├── angular-multi-select.css
│   ├── animate.min.css
│   ├── bootstrap-responsive.min.css
│   ├── bootstrap.dark.min.css
│   ├── bootstrap.light.min.css
│   ├── font-awesome.min.css
│   ├── jquery-ui.css
│   ├── jquery.multiselect.css
│   ├── normalize.min.css
│   └── timepicker.css
├── favicon.ico
├── font
│   ├── FontAwesome.otf
│   ├── fontawesome-webfont.eot
│   ├── fontawesome-webfont.svg
│   ├── fontawesome-webfont.ttf
│   └── fontawesome-webfont.woff
├── img
│   ├── annotation-icon.png
│   ├── cubes.png
│   ├── glyphicons-halflings-white.png
│   ├── glyphicons-halflings.png
│   ├── kibana.png
│   ├── light.png
│   ├── load.gif
│   ├── load_big.gif
│   ├── small.png
│   └── ui-icons_222222_256x240.png
├── index.html
└── vendor
    ├── LICENSE.json
    ├── angular
    │   ├── angular-animate.js
    │   ├── angular-cookies.js
    │   ├── angular-dragdrop.js
    │   ├── angular-loader.js
    │   ├── angular-resource.js
    │   ├── angular-route.js
    │   ├── angular-sanitize.js
    │   ├── angular-scenario.js
    │   ├── angular-strap.js
    │   ├── angular.js
    │   ├── bindonce.js
    │   ├── datepicker.js
    │   └── timepicker.js
    ├── blob.js
    ├── bootstrap
    │   ├── bootstrap.js
    │   └── less
    │       ├── accordion.less
    │       ├── alerts.less
    │       ├── bak
    │       │   ├── bootswatch.dark.less
    │       │   └── variables.dark.less
    │       ├── bootstrap.dark.less
    │       ├── bootstrap.less
    │       ├── bootstrap.light.less
    │       ├── bootswatch.dark.less
    │       ├── bootswatch.light.less
    │       ├── breadcrumbs.less
    │       ├── button-groups.less
    │       ├── buttons.less
    │       ├── carousel.less
    │       ├── close.less
    │       ├── code.less
    │       ├── component-animations.less
    │       ├── dropdowns.less
    │       ├── forms.less
    │       ├── grid.less
    │       ├── hero-unit.less
    │       ├── labels-badges.less
    │       ├── layouts.less
    │       ├── media.less
    │       ├── mixins.less
    │       ├── modals.less
    │       ├── navbar.less
    │       ├── navs.less
    │       ├── overrides.less
    │       ├── pager.less
    │       ├── pagination.less
    │       ├── popovers.less
    │       ├── progress-bars.less
    │       ├── reset.less
    │       ├── responsive-1200px-min.less
    │       ├── responsive-767px-max.less
    │       ├── responsive-768px-979px.less
    │       ├── responsive-navbar.less
    │       ├── responsive-utilities.less
    │       ├── responsive.less
    │       ├── scaffolding.less
    │       ├── sprites.less
    │       ├── tables.less
    │       ├── tests
    │       │   ├── buttons.html
    │       │   ├── css-tests.css
    │       │   ├── css-tests.html
    │       │   ├── forms-responsive.html
    │       │   ├── forms.html
    │       │   ├── navbar-fixed-top.html
    │       │   ├── navbar-static-top.html
    │       │   └── navbar.html
    │       ├── thumbnails.less
    │       ├── tooltip.less
    │       ├── type.less
    │       ├── utilities.less
    │       ├── variables.dark.less
    │       ├── variables.less
    │       ├── variables.light.less
    │       └── wells.less
    ├── chromath.js
    ├── elasticjs
    │   ├── elastic-angular-client.js
    │   └── elastic.js
    ├── elasticsearch.angular.js
    ├── filesaver.js
    ├── jquery
    │   ├── jquery-1.8.0.js
    │   ├── jquery-ui-1.10.3.js
    │   ├── jquery.flot.byte.js
    │   ├── jquery.flot.events.js
    │   ├── jquery.flot.js
    │   ├── jquery.flot.pie.js
    │   ├── jquery.flot.selection.js
    │   ├── jquery.flot.stack.js
    │   ├── jquery.flot.stackpercent.js
    │   ├── jquery.flot.threshold.js
    │   ├── jquery.flot.time.js
    │   ├── jquery.multiselect.filter.js
    │   └── jquery.multiselect.js
    ├── jsonpath.js
    ├── lodash.js
    ├── modernizr-2.6.1.js
    ├── moment.js
    ├── numeral.js
    ├── require
    │   ├── css-build.js
    │   ├── css.js
    │   ├── require.js
    │   ├── text.js
    │   └── tmpl.js
    ├── simple_statistics.js
    ├── timezone.js
    └── underscore.string.js
```

一目了然，我们可以归纳出下面几类主要文件：

* 入口：index.html
* 模块库：vendor/
* 程序入口：app/app.js
* 组件配置：app/components/
* 仪表板控制：app/controllers/
* 挂件页面：app/partials/
* 服务：app/services/
* 指令：app/directives/
* 图表：app/panels/

