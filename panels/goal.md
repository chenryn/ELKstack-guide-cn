# goal

Status: Stable

The goal panel display progress towards a fixed goal on a pie chart

## 参数

* donut
    Draw a hole in the middle of the pie, creating a tasty donut.
* tilt
    Tilt the pie back into an oval shape
* legend
    The location of the legend, above, below or none
* labels
    Set to false to disable drawing labels inside the pie slices
* spyable
    Set to false to disable the inspect function.

**query**

* query object
  * query.goal
      the fixed goal for goal mode

**请求(queries)**

* 请求对象
    这个对象描述本面板使用的请求。
  * queries.mode
    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`
  * queries.ids
    如果设为 `selected` 模式，具体被选的请求编号。
