# hits

Status: Stable

The hits panel displays the number of hits for each of the queries on the dashboard in a configurable format specified by the ‘chart’ property.

## 参数

* arrangement
    The arrangement of the legend. horizontal or vertical
* chart
    bar, pie or none
* counter_pos
    The position of the legend, above or below
* donut
    If the chart is set to pie, setting donut to true will draw a hole in the midle of it
* tilt
    If the chart is set to pie, setting tilt to true will tilt it back into an oval
* labels
    If the chart is set to pie, setting labels to true will draw labels in the slices
* spyable
    Setting spyable to false disables the inspect icon.

**请求(queries)**

* 请求对象
    这个对象描述本面板使用的请求。
  * queries.mode
    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`
  * queries.ids
    如果设为 `selected` 模式，具体被选的请求编号。
