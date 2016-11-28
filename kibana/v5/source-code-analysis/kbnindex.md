# kibana_index 结构

包括有以下 type：

## config

_id 为 kibana5 的 version。内容主要是 defaultIndex，设置默认的 index_pattern.

## search

_id 为 discover 上保存的搜索名称。内容主要是 title，column，sort，version，description，hits 和 kibanaSavedObjectMeta。kibanaSavedObjectMeta 内是一个 searchSourceJSON，保存搜索 json 的字符串。

## visualization

_id 为 visualize 上保存的可视化名称。内容包括 title，savedSearchId，description，version，kibanaSavedObjectMeta 和 visState，uiStateJSON。其中 visState 里保存了 聚合 json 的字符串。如果绑定了已保存的搜索，那么把其在 search 类型里的 _id 存在 savedSearchId 字段里，如果是从新搜索开始的，那么把搜索 json 的字符串直接存在自己的 kibanaSavedObjectMeta 的 searchSourceJSON 里。

## dashboard

_id 为 dashboard 上保存的仪表盘名称。内容包括 title, version，timeFrom，timeTo，timeRestore，uiStateJSON，optionsJSON，hits，refreshInterval，panelsJSON 和 kibanaSavedObjectMeta。其中 panelsJSON 是一个数组，每个元素是一个 panel 的属性定义。定义包括有：

* type: 具体加载的 app 类型，就默认来说，肯定就是 search 或者 visualization 之一。
* id: 具体加载的 app 的保存 id。也就是上面说过的，它们在各自类型下的 `_id` 内容。
* size_x: panel 的 X 轴长度。Kibana 采用 [gridster 库](http://gridster.net/)做挂件的动态划分，默认为 3。
* size_y: panel 的 Y 轴长度。默认为 2。
* col: panel 的左边侧起始位置。Kibana 指定 col 最大为 12。每行第一个 panel 的 col 就是 1，假如它的 size_x 是 4，那么第二个 panel 的 col 就是 5。
* row: panel 位于第几行。gridster 默认的 row 最大为 15。

## index-pattern

_id 为 setting 中设置的 index pattern。内容主要是匹配该模式的所有索引的全部字段与字段映射。如果是基于时间的索引模式，还会有主时间字段 timeFieldName 和时间间隔 intervalName 两个字段。

field 数组中，每个元素是一个字段的情况，包括字段的 type, name, indexed, analyzed, doc_values, count, scripted，searchable，aggregationable，format，popularity 这些状态。

如果 scripted 为 true，那么这个元素就是通过 kibana 页面添加的脚本化字段，那么这条字段记录还会额外多几个内容：

* script: 记录实际 script 语句。
* lang: 在 Elasticsearch 的 datanode 上采用什么 lang-plugin 运行。默认是 painless。即 Elasticsearch 5 开始默认启用的 脚本引擎。可以在 management 页面上切换成 Lucene expression 等其他你的集群中可用的脚本语言。

## timelion_sheet

包括有 timelion_chart_height，timelion_columns，timelion_interval，timelion_other_interval，timelion_rows，timelion_sheet 等字段

*小贴士*

在本书之前介绍 packetbeat 时提到的自带 dashboard 导入脚本，其实就是通过 `curl`命令上传这些 JSON 到 kibana_index 索引里。
