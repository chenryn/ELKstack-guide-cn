# 仪表板纲要

Kibana 仪表板可以很容易的在浏览器中创建出来，而且绝大多数情况下，浏览器已经足够支持你创建一个很有用很丰富的节目了。不过，当你真的需要一点小修改的时候，Kibana 也可以让你直接编辑仪表板的纲要。

注意：本节内容只针对高级用户。JSON 语法非常严格，多一个逗号，少一个大括号，都会导致你的仪表板无法加载。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/dashboard_schema/schema_dashboard.png)

我们会用上面这个仪表板作为示例。你可以导出任意的仪表板纲要，点击右上角的保存按钮，指向高级(Advanced)菜单，然后点击导出纲要(Export Schema)。示例使用的纲要文件可以在这里下载： [schema.json](http://www.elasticsearch.org/guide/en/kibana/3.0/snippets/schema.json)

因为仪表板是由特别长的 JSON 文档组成的，我们只能分成一段段的内容，分别介绍每段的作用和目的。

和所有的 JSON 文档一样，都是以一个大括号开始的。

```
{
```

## 服务(services)

```
  "services": {
```

服务(Services)是被多个面板使用的持久化对象。目前仪表板对象附加有 2 种服务对象，不指明的话，就会自动填充成请求(query)和过滤(filter)服务了。

* Query
* Filter

**query**

```
    "query": {
      "list": {
        "0": {
          "query": "play_name:\"Romeo and Juliet\"",
          "alias": "",
          "color": "#7EB26D",
          "id": 0,
          "pin": false,
          "type": "lucene",
          "enable": true
        }
      },
      "ids": [
        0
      ]
    },
```

请求服务主要是由仪表板顶部的请求栏控制的。有两个属性：

* List: 一个以数字为键的对象。每个值描述一个请求对象。请求对象的键命名一目了然，就是描述请求输入框的外观和行为的。
* Ids: 一个由 ID 组成的数组。每个 ID 都对应前面 list 对象的键。 ids 数组用来保证显示时 list 的排序问题。

**filter**

```
    "filter": {
      "list": {
        "0": {
          "type": "querystring",
          "query": "speaker:ROMEO",
          "mandate": "must",
          "active": true,
          "alias": "",
          "id": 0
        }
      },
      "ids": [
        0
      ]
    }
  },
```

过滤的行为和请求很像，不过过滤不能在面板级别选择，而是对全仪表板生效。过滤对象和请求对象一样有 list 和 ids 两个属性，各属性的行为和请求对象也一样。

## 垂幕(pulldown)

```
  "pulldowns": [
```

垂幕是一种特殊的面板。或者说，是一个特殊的可以用来放面板的地方。在垂幕里的面板就跟在行里的一样，区别就是不能设置 span 宽度。垂幕里的面板永远都是全屏宽度。此外，垂幕里的面板也不可以吧被使用者移动或编辑。所以垂幕特别适合放置输入框。垂幕的属性是一个由面板对象构成的数组。关于特定的面板，请阅读 [Kibana Panels](http://www.elasticsearch.org/guide/en/kibana/3.0/panels.html)

```
    {
      "type": "query",
      "collapse": false,
      "notice": false,
      "enable": true,
      "query": "*",
      "pinned": true,
      "history": [
        "play_name:\"Romeo and Juliet\"",
        "playname:\"Romeo and Juliet\"",
        "romeo"
      ],
      "remember": 10
    },
    {
      "type": "filtering",
      "collapse": false,
      "notice": true,
      "enable": true
    }
  ],
```

垂幕面板有 2 个普通行面板没有的选项：

* Collapse: 设置为真假值，代表着面板被折叠还是展开。
* Notice: 面板设置这个值，控制在垂幕的标签主题上出现一个小星星。用来通知使用者，这个面板里发生变动了。

## 导航(nav)

`nav` 属性里也有一个面板列表，只是这些面板是被用来填充在页首导航栏里德。目前唯一支持导航的面板是时间选择器(timepicker)

```
  "nav": [
    {
      "type": "timepicker",
      "collapse": false,
      "notice": false,
      "enable": true,
      "status": "Stable",
      "time_options": [
        "5m",
        "15m",
        "1h",
        "6h",
        "12h",
        "24h",
        "2d",
        "7d",
        "30d"
      ],
      "refresh_intervals": [
        "5s",
        "10s",
        "30s",
        "1m",
        "5m",
        "15m",
        "30m",
        "1h",
        "2h",
        "1d"
      ],
      "timefield": "@timestamp"
    }
  ],
```

## loader

`loader` 属性描述了仪表板顶部的保存和加载按钮的行为。

```
  "loader": {
    "save_gist": false,
    "save_elasticsearch": true,
    "save_local": true,
    "save_default": true,
    "save_temp": true,
    "save_temp_ttl_enable": true,
    "save_temp_ttl": "30d",
    "load_gist": false,
    "load_elasticsearch": true,
    "load_elasticsearch_size": 20,
    "load_local": false,
    "hide": false
  },
```

## 行数组

`rows` 就是通常放置面板的地方。也是唯一可以通过浏览器页面添加的位置。

```
"rows": [
    {
      "title": "Charts",
      "height": "150px",
      "editable": true,
      "collapse": false,
      "collapsable": true,
```

行对象包含了一个面板列表，以及一些行的具体参数，如下所示：

* title: 行的标题
* height: 行的高度，单位是像素，记作 px
* editable: 真假值代表面板是否可被编辑
* collapse: 真假值代表行是否被折叠
* collapsable: 真价值代表使用者是否可以折叠行

**面板数组**

行的 `panels` 数组属性包括有一个以自己出现次序排序的面板对象的列表。各特定面板本身的属性列表和说明，阅读 [Kibana Panels](http://www.elasticsearch.org/guide/en/kibana/3.0/panels.html)

```
      "panels": [
        {
          "error": false,
          "span": 8,
          "editable": true,
          "type": "terms",
          "loadingEditor": false,
          "field": "speech_number",
          "exclude": [],
          "missing": false,
          "other": false,
          "size": 10,
          "order": "count",
          "style": {
            "font-size": "10pt"
          },
          "donut": false,
          "tilt": false,
          "labels": true,
          "arrangement": "horizontal",
          "chart": "bar",
          "counter_pos": "above",
          "spyable": true,
          "queries": {
            "mode": "all",
            "ids": [
              0
            ]
          },
          "tmode": "terms",
          "tstat": "total",
          "valuefield": "",
          "title": "Longest Speeches"
        },
        {
          "error": false,
          "span": 4,
          "editable": true,
          "type": "goal",
          "loadingEditor": false,
          "donut": true,
          "tilt": false,
          "legend": "none",
          "labels": true,
          "spyable": true,
          "query": {
            "goal": 111397
          },
          "queries": {
            "mode": "all",
            "ids": [
              0
            ]
          },
          "title": "Percentage of Total"
        }
      ]
    }
  ],
```

## 索引设置

索引属性包括了 Kibana 交互的 Elasticsearch 索引的信息。

```
  "index": {
    "interval": "none",
    "default": "_all",
    "pattern": "[logstash-]YYYY.MM.DD",
    "warm_fields": false
  },
```

* interval: none, hour, day, week, month。这个属性描述了索引所遵循的时间间隔模式。
* default: 如果 `interval` 被设置为 none，或者后面的 `failover` 设置为 `true` 而且没有索引能匹配上正则模式的话，搜索这里设置的索引。
* pattern: 如果 `interval` 被设置成除了 none 以外的其他值，就需要解析这里设置的模式，启用时间过滤规则，来确定请求哪些索引。
* warm_fields: 是否需要解析映射表来确定字段列表。

## 其余

下面四个也是顶层的仪表板配置项

```
  "failover": false,
  "editable": true,
  "style": "dark",
  "refresh": false
}
```

* failover: 真假值，确定在没有匹配上索引模板的时候是否使用 `index.default`。
* editable: 真假值，确定是否在仪表板上显示配置按钮。
* style: "亮色(light)" 或者 "暗色(dark)"
* refresh: 可以设置为 "false" 或者其他 elasticsearch 支持的时间表达式(比如 10s, 1m, 1h)，用来描述多久触发一次面板的数据更新。

## 导入纲要

默认是不能导入纲要的。不过在仪表板配置屏的控制(Controls)标签里可以开启这个功能，启用 "Local file" 选项即可。然后通过仪表板右上角加载图标的高级设置，选择导入文件，就可以导入纲要了。
