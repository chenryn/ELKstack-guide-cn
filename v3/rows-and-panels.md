# 行和面板

Kibana 的仪表板是由行和面板组成的。这些都可以随意的添加，删除和重组。

这节我们会介绍：

* 加载一个空白仪表板
* 添加，隐藏行，以及修改行高
* 添加面板和修改面板宽度
* 删除面板和行

我们假设你已经：

* 在自己电脑上安装好了 Elasticsearch
* 在自己电脑上搭建好了网站服务器，并把 Kibana 发行包解压到了发布目录里
* 读过 [Using Kibana for the first time](http://www.elasticsearch.org/guide/en/kibana/current/using-kibana-for-the-first-time.html) 并且按照文章内容准备好了存有莎士比亚文集的索引

## 加载一个空白仪表板

![home screen](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/home.png)

从主屏里选择第三项，就会加载一个空白仪表板(Blank Dashboard)。默认情况下，空白仪表板会搜索 Elasticsearch 的 `_all` 索引，也就是你的全部索引。要指定搜索某个索引的，阅读 [Using Kibana for the first time](http://www.elasticsearch.org/guide/en/kibana/current/using-kibana-for-the-first-time.html)。

## 添加一行

![Adding a row](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/Addingrow.png)

你的新空白仪表板上只有展开的请求和过滤区域，页面顶栏上有个时间过滤选择器，除此以外什么都没有。在右下方，点击添加行(ADD A ROW)按钮，添加你的第一行。

![Adding a row](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/addedrow.png)

给你的行取个名字，然后点击创建(Create Row)按钮。你会看到你的新行出现在左侧的行列表里。点击保存(Save)

## 行的控制

![Row buttons](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/rowbuttons.png)

现在你有了一行，你会注意到仪表板上多了点新元素。主要是左侧多出来的三个小小的不同颜色的长方形。移动鼠标到它们上面

![Row buttons](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/buttons_expanded.png)

哈哈！看到了吧，这三个按钮是让你做这三件事情的：

* 折叠行(蓝色)
* 配置行(橘色)
* 添加面板(绿色)

## 添加面板

现在我们专注在行控制力的绿色按钮上，试试点击它。你也可以点击空白行内的灰色按钮(Add panel to empty row)，不过它是灰色的啊，有啥意思……

![Add panel](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/addpanel.png)

让我们来添加一个 terms 面板。terms 面板可以让我们用上 Elasticsearch 的 terms facet 功能，查找一个字段内最经常出现的几个值。

![Add panel](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/terms_settings.png)

你可以看到，terms 面板有一系列可配置选选，不过我们现在先只管第一段里德通用配置好了：

1. Title: 面板的名称
2. Span: 面板的宽度。Kibana 仪表板等分成 12 个 *spans*
   面板最大就是到 12 个 spans 宽。但是行可以容纳超过 12 个 spans 的总宽度，因为它会自动把新的面板放到下面显示。现在我们先设置为 4。
3. Editable: 面板是否在之后可以继续被编辑。现在先略过。
4. Inspectable: 面板是否允许用户查看所用的请求内容。现在先略过。
5. 点击 **Save** 添加你的新 **terms** 面板到你的仪表板

**译者注：面板宽度也可以在仪表板内直接拖拽修改，将鼠标移动至面板左(右)侧边线处，鼠标会变成相应的箭头，按住左键拖拽成满意宽度松开即可**

![First panel](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/firstpanel.png)

太棒了！你现在有一个面板了！你可能意识到这个数据跟 [Using Kibana for the first time](http://www.elasticsearch.org/guide/en/kibana/current/using-kibana-for-the-first-time.html) 中的饼图数据一样。 `shakespeare` 数据集集中在 lines，还有少量的 acts 和 scenes。

## 折叠和展开行

![Row buttons](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/buttons_expanded.png)

蓝色按钮可以折叠你的行。被折叠行里的面板不会刷新数据，也就不要求 Elasticsearch 资源。所以折叠行可以用于那些你不需要经常看的数据。有需要的时候点击蓝色按钮展开就可以了。

![Collapsed row](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/collapsed.png)

顶部的请求和过滤区域也可以被折叠。点击彩色标签就可以折叠和展开。

![Collapsed top row](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/toprowscollapsed.png)

## 编辑行

通过行编辑器，可以给行重命名，改行高等其他配置。点击橙色按钮打开行编辑器。

![Row](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/roweditor.png)

这个对话框还允许你修改面板的排序和大小，以及删除面板。

![Removing Panels](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/rowpanels.png)

## 移动和删除面板

面板可以在本行，甚至其他行之间任意拖拽。按住面板右上角的十字架形状小图标然后拖动即可。

![Removing Panels](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/movepanel.png)

点击面板右上角的 *remove* 小图标就可以从仪表板上移除它。前面说到从行编辑器上也可以做到统一效果。

![Removing Panels](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/removing_panels.png)

## 移动和删除行

行可以在仪表板配置页中重新排序和删。点击屏幕右上角的配置按钮，选择行(Rows)标签切换到行配置层。看到这里你一定会记起来我们在添加第一个行时候的屏幕。

![Removing Rows](http://www.elasticsearch.org/guide/en/kibana/current/tutorials/rows_panels/rowmove.png)

左侧的箭头用来修改仪表板上行的次序。X 用来删除行。

## 下一步

在你关闭浏览器之前，你可能打算保存这个新仪表板。请阅读 [Saving and Loading dashboards](http://www.elasticsearch.org/guide/en/kibana/current/saving-and-loading-dashboards.html)。
