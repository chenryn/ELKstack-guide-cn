# 主页入口

kibana 4 主页入口，分析方法跟 kibana 3 一样，看 index.html 和 require.config.js 即可。由此可以看到，首先进入的，应该是 index.js。index.js 根据 configFile 设置默认 routes，然后执行 `kibana.load()` 函数，首先加载 `plugins/kibana/index.js`，然后加载其他 plugins。

设置 routes 的具体操作在加载的 `utils/routes/index.js` 文件里，其中调用 `utils/routes/_setup.js`，在未设置 default index pattern 的时候跳转 URL 到 "/settings/indices" 页面。

`plugins/kibana/index.js` 里又有一些列操作：首先加载 `components/setup/setup.js` 和 `components/config/config.js` 两个 angular.service，然后加载 `plugins/kibana/_init`, `plugins/kibana/_apps`, `plugins/kibana/_timepicker`。

## setup 过程

`components/setup/setup.js` 依次调用 `components/setup/steps/` 下的 `check_for_es`, `check_es_version`, `check_for_kibana_index`，如果没有 kibana index，再调用一个 `create_kibana_index`。完成。

`components/config/config.js` 主要是从 kibana index 里的 "config" type 中读取 "kbnVersion" id 的数据。这个 "kbnVersion" 是源代码(index.js)里的一个常量，在 grunt 编译时会生成的。

`plugins/kibana/_init` 里监听 `application.load` 事件，触发 `courier.start()` 函数。

`plugins/kibana/_apps` 提供路径记忆(lastPath)功能，这点在 kibana4 user guide 上被专门提到过；然后初始化 `registry/apps`，并循环调用 `assignPaths` 和 `getShow` 方法。

`plugins/kibana/_timepicker` 提供时间选择器页面。

## courier 概述

`components/courier/courier.js` 中，加载 `index_pattern` 和 `saved_objects`，启动 searchLooper 和 docLooper；设置整个页面的定期刷新。

**courier** 是一个非常重要的东西，除了上行提到的这几个以外，目录下还有 docSource 和 searchSource ，可以简单理解为 kibana 跟 ES 之间的一个 object mapper。其中和 ES 的实际交互，是调用了 `services/es.js` 里定义的 service，当然里面内容超级简单，就是加载官方的 elasticsearch.js 库，然后初始化一个最简的 esFactory 客户端，包括超时都设成了 0，把这个控制交给 server 端。

searchLooper, docLooper 则是限制在 `_request_queue` 里的 Looper 对象，分别给 `Looper.start` 方法传递 FetchStrategyForSearch, FetchStrategyForDoc，对应 ES 的 `/_msearch` 和 `/_mget` 请求。这两个在 `components/courier/fetch/strategy/search.js` 和 `components/courier/fetch/strategy/doc.js` 里定义。

## registry 概述

`registry/apps.js` 主要是加载 `registry/_registry.js`，把注册的 app 存入 `utils/indexed_array/index` 的 IndexedArray 对象。对象主要有几个值：id, name, order。前面说到的两个方法，assignPaths 里就是用 app.id 设置 lastPath，而 getShow 里就是用 order 来判断是否展示在页面上。

所以这里就体现出 kibana 4 的可扩展性了。事实上，在服务器端的 index.js 上，就有下面配置：

```
external_plugins_folder : process.env.PLUGINS_FOLDER || null,
bundled_plugins_folder  : path.resolve(public_folder, 'plugins'),
```

可以看到，官方的 apps，都是在 plugins 目录下的，而自己开发的 apps，可以通过环境变量 `$PLUGINS_FOLDER` 设置加载进来。

下一章，我们开始介绍官方提供的几个 apps。
