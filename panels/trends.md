# trends

Status: Beta

A stock-ticker style representation of how queries are moving over time. For example, if the time is 1:10pm, your time picker was set to "Last 10m", and the "Time Ago" parameter was set to "1h", the panel would show how much the query results have changed since 12:00-12:10pm

## 参数

* ago

    A date math formatted string describing the relative time period to compare the queries to.

* arrangement

    ‘horizontal’ or ‘vertical’

* spyable

    设为假，不显示审查(inspect)按钮。

**请求(queries)**

* 请求对象

    这个对象描述本面板使用的请求。

  * queries.mode

    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`

  * queries.ids

    如果设为 `selected` 模式，具体被选的请求编号。
