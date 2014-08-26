# histogram

状态：稳定

histogram 面板用以显示时间序列图。它包括好几种模式和变种，用以显示时间的计数，平均数，最大值，最小值，以及数值字段的和，计数器字段的导数。

## 参数

**轴(axis)参数**

* mode
    用于 Y 轴的值。除了 count 以外，其他 `mode` 设置都要求定义 `value_field` 参数。可选值为：count, mean, max, min, total。
* time_field
    X 轴字段。必须是在 Elasticsearch 中定义为时间类型的字段。
* value_field
    如果 `mode` 设置为 mean, max, min 活着 total，Y 轴字段。必须是数值型。
* x-axis
    是否显示 X 轴。
* y-axis
    是否显示 Y 轴。
* scale
    以该因子规划 Y 轴
* y_format
   Y 轴数值格式，可选：none, bytes, short

**注释**

* 注释对象
    可以指定一个请求的结果作为标记显示在图上。比如说，标记某时刻部署代码了。
  * annotate.enable
    是否显示注释(即标记)
  * annotate.query
    标记使用的 Lucene query_string 语法请求
  * annotate.size
    最多显示多少标记
  * annotate.field
    显示哪个字段
  * annotate.sort
    数组排序，格式为 [field,order]。比如 [‘@timestamp’,‘desc’] ，这是一个内部参数。
* auto_int
    是否自动调整间隔
* resolution
    If auto_int is true, shoot for this many bars.
* interval
    如果 `auto_int` 设为假，用这个值做间隔
* intervals
    在 `View` 选择器里可见的间隔数组。比如 [‘auto’,‘1s’,‘5m’,‘3h’]，这是绘图参数。
* lines
    显示折线图
* fill
    折线图的区域填充因子，从 1 到 10。
* linewidth
    折线的宽度，单位为像素
* points
    在图上显示数据点
* pointradius
    数据点的大小，单位为像素
* bars
    显示条带图
* stack
    堆叠多个序列
* spyable
    显示审核图标
* zoomlinks
    显示 ‘Zoom Out’ 链接
* options
    显示快捷的 view 选项区域
* legend
    显示图例
* show_query
    如果没设别名(alias)，是否显示请求
* interactive
    允许点击拖拽进行放大
* legend_counts
    在图例上显示计数
* timezone
    是否调整成浏览器时区。可选值为：browser, utc
* percentage
    把 Y 轴数据显示成百分比样式。仅对多个请求时有效。
* zerofill
    提高折线图准确度，稍微消耗一点性能。
* derivative
    在 X 轴上显示该点数据在前一个点数据上变动的数值。
* 提示框(tooltip)对象
  * tooltip.value_type
    控制 tooltip 在堆叠图上怎么显示，可选值：独立(individual)还是累计(cumulative)
  * tooltip.query_as_alias
    如果没设别名(alias)，是否显示请求
* 网格(grid)对象
    Y 轴的最大值和最小值
  * grid.min
    Y 轴的最小值
  * grid.max
    Y 轴的最大值

**请求(queries)**

* 请求对象
    这个对象描述本面板使用的请求。
  * queries.mode
    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`
  * queries.ids
    如果设为 `selected` 模式，具体被选的请求编号。
