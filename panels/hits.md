# hits

Status: Stable

The hits panel displays the number of hits for each of the queries on the dashboard in a configurable format specified by the ‘chart’ property.

## 参数

* arrangement

    在条带(bar)或者饼图(pie)模式，图例的摆放方向。可以设置：水平(horizontal)或者垂直(vertical)。

* chart

    可以设置：none, bar 或者 pie

* counter_pos

    图例相对于图的位置，可以设置：上(above)，下(below)

* donut

    在饼图(pie)模式，在饼中画个圈，变成甜甜圈样式。

* tilt

    在饼图(pie)模式，倾斜饼变成椭圆形。

* lables

    在饼图(pie)模式，在饼图分片上绘制标签。

* spyable

    设为假，不显示审查(inspect)图标。

**请求(queries)**

* 请求对象

    这个对象描述本面板使用的请求。

  * queries.mode

    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`

  * queries.ids

    如果设为 `selected` 模式，具体被选的请求编号。
