# 保存和加载

你已经构建了一个漂亮的仪表板！现在你打算分享给团队，或者开启自动刷新后挂在一个大屏幕上？Kibana 可以把仪表板设计持久化到 Elasticsearch 里，然后在需要的时候通过加载菜单或者 URL 地址调用出来。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/saving_loading/awesome_dashboard.png)

## 保存你漂亮的仪表板

保存你的界面非常简单，打开保存下拉菜单，取个名字，然后点击保存图表即可。现在你的仪表板就保存在一个叫做 `kibana-int` 的 Elasticsearch 索引里了。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/saving_loading/savebutton.png)

## 调用你的仪表板

要搜索已保存的仪表板列表，点击右上角的加载图标。在这里你可以加载，分享和删除仪表板。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/saving_loading/searchdashboards.png)

## 分享仪表板

已保存的仪表板可以通过你浏览器地址栏里的 URL 分享出去。每个持久化到 Elasticsearch 里的仪表板都有一个对应的 URL，像下面这样：

```
http://your_host/index.html#/dashboard/elasticsearch/MYDASHBOARD
```

这个示例中 `MYDASHBOARD` 就是你在保存的时候给仪表板取得名字。

你还可以分享一个即时的仪表板链接，点击 Kibana 右上角的分享图标，会生成一个临时 URL。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/saving_loading/sharebutton.png)

默认情况下，临时 URL 保存 30 天。

![image](http://www.elasticsearch.org/guide/en/kibana/3.0/tutorials/saving_loading/sharelink.png)

## 保存成静态仪表板

仪表板可以保存到你的服务器磁盘上成为 `.json` 文件。把文件放到 `app/dashboards` 目录，然后通过下面地址访问

```
http://your_host/index.html#/dashboard/file/MYDASHBOARD.json
```

`MYDASHBOARD.json` 就是磁盘上文件的名字。注意路径中得 `/#dashboard/file/` 看起来跟之前访问保存在 Elasticsearch 里的仪表板很类似，不过这里访问的是文件而不是 elasticsearch。导出的仪表板纲要的详细信息，阅读 [The Dashboard Schema Explained](http://www.elasticsearch.org/guide/en/kibana/3.0/_dashboard_schema.html)

## 下一步

你现在知道怎么保存，加载和访问仪表板了。你可能想知道怎么通过 URL 传递参数来访问，这样可以在其他应用中直接链接过来。请阅读 [Templated and Scripted Dashboards](http://www.elasticsearch.org/guide/en/kibana/3.0/templated-and-scripted-dashboards.html)
