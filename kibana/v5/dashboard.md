一个 Kibana *dashboard* 能让你自由排列一组已保存的可视化。然后你可以保存这个仪表板，用来分享或者重载。

简单的仪表板像这样。![Example dashboard](https://www.elastic.co/guide/en/kibana/current/images/tutorial-dashboard.png)

## 开始

要用仪表板，你需要至少有一个已保存的 [visualization](http://www.elastic.co/guide/en/kibana/current/visualize.html)。

### 创建一个新的仪表板

你第一次点击 **Dashboard** 标签的时候，Kibana 会显示一个空白的仪表板

![New Dashboard screen](http://www.elastic.co/guide/en/kibana/current/images/NewDashboard.jpg)

通过添加可视化的方式来构建你的仪表板。默认情况下，Kibana 仪表板使用明亮风格。如果你想切换成黑色风格，点击 **Options**，然后勾选 **Use dark theme**。

![dark theme](https://www.elastic.co/guide/en/kibana/current/images/darktheme.png)

### 自动刷新页面

你可以设置页面自动刷新的间隔以查看最新索引进来的数据。这个设置会定期提交搜索请求。

设置刷新间隔后，它会出现在菜单栏的时间过滤器左侧。

要设置刷新间隔：

1. 点击菜单栏右上角的 **Time Filter**![](https://www.elastic.co/guide/en/kibana/current/images/TimeFilter.jpg)
2. 点击 **Refresh Interval** 标签
3. 从列表中选择一个刷新间隔

要自动刷新数据，点击 **Auto-Refresh**![](https://www.elastic.co/guide/en/kibana/current/images/autorefresh.png) 按钮选择自动刷新间隔：

![](https://www.elastic.co/guide/en/kibana/current/images/autorefresh-intervals.png)

开启自动刷新后，Kibana 顶部菜单栏会显示一个暂停按钮和自动刷新间隔：![](https://www.elastic.co/guide/en/kibana/current/images/autorefresh-pause.png)。点击这个暂停按钮可以暂停自动刷新。

### 添加可视化到仪表板上

要添加可视化到仪表板上，点击工具栏面板上的 **Add**。从列表中选择一个已保存的可视化。你可以在 **Visualization Filter** 里输入字符串来过滤想要找的可视化。

由你选择的这个可视化会出现在你仪表板上的一个*容器(container)*里。

### 保存仪表板

要保存仪表板，点击工具栏面板上的 **Save** 按钮，在 **Save As** 栏输入仪表板的名字，然后点击 **Save** 按钮。

### 加载已保存仪表板

点击 **Open** 按钮显示已存在的仪表板列表。已保存仪表板选择器包括了一个文本栏可以通过仪表板的名字做过滤，还有一个链接到 **Object Editor** 而已管理你的已保存仪表板。你也可以直接点击 **Settings > Edit Saved Objects** 来访问 **Object Editor**。

### 分享仪表板

你可以分享仪表板给其他用户。可以直接分享 Kibana 的仪表板链接，也可以嵌入到你的网页里。

> 用户必须有 Kibana 的访问权限才能看到嵌入的仪表板。

点击 **Share** 按钮显示 HTML 代码，就可以嵌入仪表板到其他网页里。还带有一个指向仪表板的链接。点击复制按钮![Copy](http://www.elastic.co/guide/en/kibana/current/images/Clipboard.png) 可以复制代码，或者链接到你的黏贴板。

### 嵌入仪表板

要嵌入仪表板，从 *Share* 页里复制出嵌入代码，然后粘贴进你外部网页应用内即可。

## 定制仪表板元素

仪表板里的可视化都存在可以调整大小的*容器*里。接下来会讨论一下容器。

### 移动容器

点击并按住容器的顶部，就可以拖动容器到仪表板任意位置。其他容器会在必要的时候自动移动，给你在拖动的这个容器空出位置。松开鼠标，容器就会固定在当前停留位置。

### 改变容器大小

移动光标到容器的右下角，等光标变成指向拐角的方向，点击并按住鼠标，拖动改变容器的大小。松开鼠标，容器就会固定成当前大小。

### 删除容器

点击容器右上角的 x 图标删除容器。从仪表板删除容器，并不会同时删除掉容器里用到的已存可视化。

### 查看详细信息

要显示可视化背后的原始数据，点击容器地步的条带。可视化会被有关原始数据详细信息的几个标签替换掉。如下所示：

![images/NYCTA-Table.jpg](http://www.elastic.co/guide/en/kibana/current/images/NYCTA-Table.jpg)

* 表格(Table)。底层数据的分页展示。你可以通过点击每列顶部的方式给该列数据排序。
* 请求(Request)。发送到服务器的原始请求，以 JSON 格式展示。
* 响应(Response)。从服务器返回的原始响应，以 JSON 格式展示。
* 统计值(Statistics)。和请求响应相关的一些统计值，以数据网格的方式展示。数据报告，请求时间，响应时间，返回的记录条目数，匹配请求的索引模式(index pattern)。

## 修改可视化

点击容器右上角的 *Edit* 按钮在 [Visualize](./visualize.md) 页打开可视化编辑。
