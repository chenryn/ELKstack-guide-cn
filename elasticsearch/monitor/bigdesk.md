# bigdesk

要想最快的了解 ES 各节点的性能细节，推荐使用 bigdesk 插件，其 GitHub 地址见：<https://github.com/lukas-vlcek/bigdesk>

bigdesk 通过浏览器直连 ES 节点，发起 RESTful 请求，并渲染结果成图。所以其安装部署极其简单：

```
# git clone https://github.com/lukas-vlcek/bigdesk
# cd bigdesk
# python -mSimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

浏览器打开 `http://localhost:8000` 即可看到 bigdesk 页面。在 **endpoint** 输入框内填写要连接的 ES 节点地址，选择 refresh 间隔和 keep 时长，点击 **connect**，完成。

![](./bigdesk-banner.png)

注意：设置 refresh 间隔请考虑 ELK Stack 使用的 template 里实际的 `refresh_interval` 是多少。否则你可能看到波动太大的数据，不足以说明情况。

点选某个节点后，就可以看到该节点性能的实时走势。一般重点关注 JVM 性能和索引性能。

有关 JVM 部分截图如下：

![](./bigdesk-jvm.png)

有关数据读写性能部分截图如下：

![](./bigdesk-indexing.png)

## Elasticsearch 2.x 上的部署方法

bigdesk 的源码开发在 2015 年夏天就已经停止，所以默认是无法支持 Elasticsearch 2.x 的监控的。不过总体来说，Elasticsearch 性能监控接口没有什么变化，我们只需要稍微修改 bigdesk，使之符合 Elasticsearch 2.x 的插件规范即可继续使用。

Elasticsearch 2.x 的插件目录规范，可以参考 marvel-agent，kopf 等。

```
mkdir -p $ES_HOME/plugins/bigdesk
cd $ES_HOME/plugins/bigdesk
git clone https://github.com/lukas-vlcek/bigdesk _site
sed -i '142s/==/>=/' _site/js/store/BigdeskStore.js
cat >plugin-descriptor.properties<<EOF
description=bigdesk - Live charts and statistics for Elasticsearch cluster.
version=2.5.1
site=true
name=bigdesk
EOF
```

然后启动 Elasticsearch 服务，通过 `http://127.0.0.1:9200/_plugin/bigdesk` 访问即可。

*注意：这种方式只能恢复 node 监控，cluster 部分，bigdesk 是采用了 `/_status` 接口，这个接口从 Elasticsearch 1.2 开始就已经宣布废弃，2.x 里已经彻底没有了，数据被拆分成了 `/_stats` 和 `/_recovery` 两个接口。不是简单修改可以搞定的了。*
