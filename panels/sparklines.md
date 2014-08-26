# sparklines

Status: Experimental

The sparklines panel shows tiny time charts. The purpose of these is not to give an exact value, but rather to show the shape of the time series in a compact manner

## 参数

* mode
    Value to use for the y-axis. For all modes other than count, `value_field` must be defined. Possible values: count, mean, max, min, total.
* time_field
    x-axis field. This must be defined as a date type in Elasticsearch.
* value_field
    y-axis field if mode is set to mean, max, min or total. Must be numeric.
* interval
    Sparkline intervals are computed automatically as long as there is a time filter present. In the absence of a time filter, use this interval.
* spyable
    Show inspect icon

**请求(queries)**

* 请求对象
    这个对象描述本面板使用的请求。
  * queries.mode
    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`
  * queries.ids
    如果设为 `selected` 模式，具体被选的请求编号。
