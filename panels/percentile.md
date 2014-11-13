# percentile

状态: Beta

基于 Elasticsearch 的 percentile Aggre 接口实现的统计聚合展示面板。

## 参数

* format

  返回值的格式。默认是 number，可选值还有：money，bytes，float。

* style

  主数字的显示大小，默认为 24pt。

* modes

  用来做百分比的聚合值。包括25%, 50%, 75%, 90%, 95%, 99%。

* show

  统计表格中具体展示的哪些列。默认为全部展示，可选列名即 modes 中的可选值。

* spyable

  设为假，不显示审查(inspect)按钮。

**请求**

* 请求对象

    这个对象描述本面板使用的请求。

  * queries.mode

    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`

  * queries.ids

    如果设为 `selected` 模式，具体被选的请求编号。
