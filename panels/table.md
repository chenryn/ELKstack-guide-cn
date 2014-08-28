# table

状态：稳定

表格面板里是一个可排序的分页文档。你可以定义需要排列哪些字段，并且还提供了一些交互功能，比如执行 terms 聚合查询。

## 参数

* size

    每页显示多少条

* pages

    展示多少页

* offset

    当前页的页码

* sort

    定义表格排序次序的数组，示例如右：[‘@timestamp’,‘desc’]

* overflow

    css 的 overflow 属性。‘min-height’ (expand) 或 ‘auto’ (scroll)

* fields

    表格显示的字段数组

* highlight

    高亮显示的字段数组

* sortable

    设为假关掉排序功能

* header

    设为假隐藏表格列名

* paging

    设为假隐藏表格翻页键

* field_list

    设为假隐藏字段列表。使用者依然可以展开它，不过默认会隐藏起来

* all_fields

    设为真显示映射表内的所有字段，而不是表格当前使用到的字段

* trimFactor

    裁剪因子(trim factor)，是参考表格中的列数来决定裁剪字段长度。比如说，设置裁剪因子为 100，表格中有 5 列，那么每列数据就会被裁剪为 20 个字符。完整的数据依然可以在展开这个事件后查看到。

* localTime

    设为真调整 `timeField` 的数据遵循浏览器的本地时区。

* timeField

    如果 `localTime` 设为真，该字段将会被调整为浏览器本地时区。

* spyable

    设为假，不显示审查(inspect)按钮。

**请求(queries)**

* 请求对象

    这个对象描述本面板使用的请求。

  * queries.mode

    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`

  * queries.ids

    如果设为 `selected` 模式，具体被选的请求编号。
