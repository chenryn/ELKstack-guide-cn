# 请求和过滤

图啊，表啊，地图啊，Kibana 有好多种图表，我们怎么控制显示在这些图表上的数据呢？这就是请求和过滤起作用的地方。 Kibana 是基于 Elasticsearch 的，所以支持强大的 Lucene Query String 语法，同样还能用上 Elasticsearch 的过滤器能力。

## 我们的仪表板

我们的仪表板像下面这样，可以搜索莎士比亚文集的内容。如果你喜欢本章截图的这种仪表板样式，你可以[下载导出的仪表板纲要(dashboard schema)](http://www.elasticsearch.org/guide/en/kibana/3.0/snippets/plays.json)

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/plays.png)

## 请求

在搜索栏输入下面这个非常简单的请求

```
to be or not to be
```

你会注意到，表格里第一条就是你期望的《哈姆雷特》。不过下一行却是《第十二夜》的安德鲁爵士，这里可没有"to be"，也没有"not to be"。事实上，这里匹配上的是 `to OR be OR or OR not OR to OR be`。

我们需要这么搜索(译者注：即加双引号)来匹配整个短语：

```
"to be or not to be"
```

或者指明在某个特定的字段里搜索：

```
line_id:86169
```

我们可以用 AND/OR 来组合复杂的搜索，注意这两个单词必须大写：

```
food AND love
```

还有括号：

```
("played upon" OR "every man") AND stage
```

数值类型的数据可以直接搜索范围：

```
line_id:[30000 TO 80000] AND havoc
```

最后，当然是搜索所有：

```
*
```

## 多个请求

有些场景，你可能想要比对两个不同请求的结果。Kibana 可以通过 OR 的方式把多个请求连接起来，然后分别进行可视化处理。

**添加请求**

点击请求输入框右侧的 + 号，即可添加一个新的请求框。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/Addquery.png)

点击完成后你应该看到的是这样子

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/split.png)

在左边，绿色输入框，输入 `"to be"` 然后右边，黄色输入框，输入 `"not to be"`。这就会搜索每个包含有 `"to be"` 或者 `"not to be"` 内容的文档，然后显示在我们的 hits 饼图上。我们可以看到原先一个大大的绿色圆形变成下面这样：

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/pieslice.png)

**移除请求**

要移除一个请求，移动鼠标到这个请求输入框上，然后会出现一个 x 小图标，点击小图标即可：

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/remove.png)

## 颜色和图例

Kibana 会自动给你的请求分配一个可用的颜色，不过你也可以手动设置颜色。点击请求框左侧的彩色圆点，就可以弹出请求设置下拉框。这里面可以修改请求的颜色，或者设置为这个请求设置一个新的图例文字：

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/settings.png)

## 过滤

很多 Kibana 图表都是交互式的，可以用来过滤你的数据视图。比如，点击你图表上的第一个条带，你会看到一些变动。整个图变成了一个大大的绿色条带。这是因为点击的时候，就添加了一个过滤规则，要求匹配 `play_name` 字段里的单词。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/comedyoferrors.png)

你要问了“在哪里过滤了”？

答案就藏在过滤(FILTERING)标签上出现的白色小星星里。点击这个标签，你会发现在 *filtering* 面板里已经添加了一个过滤规则。在 *filtering* 面板里，可以添加，编辑，固定，删除任意过滤规则。很多面板都支持添加过滤规则，包括表格(table)，直方图(histogram)，地图(map)等等。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/queries_filters/filteradded.png)

过滤规则也可以自己点击 + 号手动添加。

