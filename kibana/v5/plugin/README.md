# Kibana 插件

Kibana 从 4.2 以后，引入了完善的插件化机制。目前分为 app，vistype，fieldformatter、spymode 等多种插件类型。原先意义上的 Kibana 现在已经变成了 Kibana 插件框架下的一个默认 app 类型插件。

本节用以讲述 Kibana 插件的安装使用和定制开发。

## 部署命令

安装 Kibana 插件有两种方式：

1. 通过 Elastic.co 公司的下载地址：

```
bin/kibana_plugin --install <org>/<package>/<version>
```

version 是可选项。这种方式目前适用于官方插件，比如：

```
bin/kibana_plugin -i elasticsearch/marvel/latest
bin/kibana_plugin -i elastic/timelion
```

2. 通过 zip 压缩包：

支持本地和远程 HTTP 下载两种，比如：

```
bin/kibana_plugin --install sense -u file:///tmp/sense-2.0.0-beta1.tar.gz
bin/kibana_plugin -i heatmap -u https://github.com/stormpython/heatmap/archive/master.zip
bin/kibana_plugin -i kibi_timeline_vis -u https://github.com/sirensolutions/kibi_timeline_vis/raw/0.1.2/target/kibi_timeline_vis-0.1.2.zip
bin/kibana_plugin -i oauth2 -u https://github.com/trevan/oauth2/releases/download/0.1.0/oauth2-0.1.0.zip
```

目前已知的 Kibana Plugin 列表见官方 WIKI：<https://github.com/elastic/kibana/wiki/Known-Plugins>

注意：kibana 目前版本变动较大，不一定所有插件都可以成功使用

## 查看与切换

插件安装完成后，可以在 Kibana 页面上通过 app switcher 界面切换。界面如下：

![](https://www.elastic.co/guide/en/kibana/current/images/app-picker.png)

## 默认插件

除了 Kibana 本身以外，其实还有一些其他默认插件，这些插件本身在 app switcher 页面上是隐藏的，但是可以通过 url 直接访问到，或者通过修改插件的 `index.js` 配置项让它显示出来。

这些隐藏的默认插件中，最有可能被用到的，是 statusPage 插件。

我们可以通过 `http://localhost:5601/status` 地址访问这个插件的页面：

![](https://www.elastic.co/guide/en/kibana/current/images/kibana-status-page.png)

页面会显示 Kibana 的运行状态。包括 nodejs 的内存使用、负载、响应性能，以及各插件的加载情况。
