# percentile

状态: Beta

基于 Elasticsearch 的 percentile Aggregation 接口实现的统计聚合展示面板。

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

----------------

## 界面配置说明

percentile 面板界面与 stats 面板界面类似。

![percentile aggr](http://ww4.sinaimg.cn/large/3dbd9afatw1eh4kh2se8lj209m093aai.jpg)

## 代码实现要点

1. percentile Aggregation 是 Elasticsearch 从 1.1.0 开始新加入的实验性功能，而且在 1.3.0 之后其返回的数据结构发生了变动。所以代码中对 ESversion 要做判断和兼容性处理。
2. percentile Aggregation 返回的数据中，强制保留了百分数的小数点后一位，这导致在 js 处理中会把小数点当做是属性调用的操作符。所以需要在前端展示的 "." 替换成后端使用的 "_"。
3. percentile Aggregation 请求中，不支持使用中文做 aggregation name。如果 `query.alias` 写了中文的，就会出问题。所以这里直接采用序号了。
