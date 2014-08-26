# terms

状态：稳定

基于 Elasticsearch 的 terms facet 接口数据展现表格，条带图，或者饼图。

## 参数

* field
    用于计算 facet 的字段名称。
* exclude
    要从结果数据中排除掉的 terms
* missing
    设为假，就可以不显示数据集内有多少结果没有你指定的字段。
* other
    设为假，就可以不显示聚合结果在你的 `size` 属性设定范围以外的总计数值。
* size
    显示多少个 terms
* order
    terms 模式可以设置：count, term, reverse_count 或者 reverse_term；terms_stats 模式可以设置：term, reverse_term, count, reverse_count, total, reverse_total, min, reverse_min, max, reverse_max, mean 或者 reverse_mean
* donut
    在饼图(pie)模式，在饼中画个圈，变成甜甜圈样式。
* tilt
    在饼图(pie)模式，倾斜饼变成椭圆形。
* lables
    在饼图(pie)模式，在饼图分片上绘制标签。
* arrangement
    在条带(bar)或者饼图(pie)模式，图例的摆放方向。可以设置：水平(horizontal)或者垂直(vertical)。
* chart
    可以设置：table, bar 或者 pie
* counter_pos
    图例相对于图的位置，可以设置：上(above)，下(below)，或者不显示(none)。
* spyable
    设为假，不显示审查(inspect)按钮。

**请求(queries)**

* 请求对象
    这个对象描述本面板使用的请求。
  * queries.mode
    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`
  * queries.ids
    如果设为 `selected` 模式，具体被选的请求编号。
* tmode
    Facet 模式：terms 或者 terms_stats
* tstat
    Terms_stats facet stats 字段。
* valuefield
    Terms_stats facet value 字段。
