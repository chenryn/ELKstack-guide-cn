# Kibana 截图报表

Elastic Stack 本身作为一个实时数据检索聚合的系统，在定期报表方面，是有一定劣势的。因为基本上不可能把源数据长期保存在 Elasticsearch 集群中。即便保存了，为了一些已经成形的数据，再全面查询一次过久的冷数据，也是有额外消耗的。那么，对这种报表数据的需求，如何处理？其实很简单，把整个 Kibana 页面截图下来即可。

FireFox 有插件用来截全网页图。不过如果作为定期的工作，这么搞还是比较麻烦的，需要脚本化下来。这时候就可以用上 phantomjs 软件了。phantomjs 是一个基于 webkit 引擎做的 js 脚本库。可以通过 js 程序操作 webkit 浏览器引擎，实现各种浏览器功能。

phantomjs 在 Linux 平台上没有二进制分发包，所以必须源代码编译：

```
# yum -y install gcc gcc-c++ make flex bison gperf ruby \
  openssl-devel freetype-devel fontconfig-devel libicu-devel sqlite-devel \
  libpng-devel libjpeg-devel
# git clone git://github.com/ariya/phantomjs.git
# cd phantomjs
# git checkout 2.0
# ./build.sh
```

想要给 kibana 页面截图，几行代码就够了。

## Kibana3 截屏代码

下面是一个 Kibana3 的截屏示例。 `capture-kibana.js` 如下：

```
var page = require('webpage').create();
var address = 'http://kibana.example.com/#/dashboard/elasticsearch/h5_view';
var output = 'kibana.png';
page.viewportSize = { width: 1366, height: 600 };
page.open(address, function (status) {
    if (status !== 'success') {
        console.log('Unable to load the address!');
        phantom.exit();
    } else {
        window.setTimeout(function () {
            page.render(output);
            phantom.exit();
        }, 30000);
    }
});
```

然后运行 `phantomjs capture-kibana.js` 命令，就能得到截图生成的 kibana.png 图片了。

这里两个要点：

1. 要设置 `viewportSize` 里的宽度，否则效果会变成单个 panel 依次往下排列。
3. 要设置 `setTimeout`，否则在获取完 index.html 后就直接返回了，只能看到一个大白板。用 phantomjs 截取 angularjs 这类单页 MVC 框架应用时一定要设置这个。

## Kibana4 的截屏开源项目

Kibana4 的 dashboard 同样可以采取上面 Kibana3 的方式自己实现。不过社区最近有人完成了一个辅助项目，把截屏配置、历史记录展示等功能都界面化了。项目地址见：<https://github.com/parvez/snapshot_for_kibana>。

![](https://raw.githubusercontent.com/parvez/snapshot_for_kibana/master/screenshots/4.%20schedule.png)
