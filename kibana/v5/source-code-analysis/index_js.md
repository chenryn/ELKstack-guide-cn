# 主页入口

kibana 4 主页入口，分析方法跟 kibana 3 一样，看 index.html 和 require.config.js 即可。由此可以看到，首先进入的，应该是 index.js。主要分为两步。

## 第一步，route 设置

```
define(function (require) {
  var angular = require('angular');
  var _ = require('lodash');
  var $ = require('jquery');
  var modules = require('modules');
  var routes = require('routes');

  var configFile = JSON.parse(require('text!config'));

  var kibana = modules.get('kibana', [
    'elasticsearch',
    'pasvaz.bindonce',
    'ngRoute',
    'ngClipboard'
  ]);

  kibana
    .constant('kbnVersion', window.KIBANA_VERSION)
    .constant('minimumElasticsearchVersion', '1.6.0')
    .constant('sessionId', Date.now())
    .config(routes.config);

  routes
    .otherwise({
      redirectTo: '/' + configFile.default_app_id
    });
```

index.js 根据 configFile 设置默认 routes，设置要求的 Elasticsearch 版本；


设置 routes 的具体操作在加载的 `utils/routes/index.js` 文件里，其中会调用 `utils/routes/_setup.js`，在未设置 default index pattern 的时候跳转 URL 到 "/settings/indices" 页面。

```
      handleKnownError: function (err) {
        if (err instanceof NoDefaultIndexPattern || err instanceof NoDefinedIndexPatterns) {
          kbnUrl.change('/settings/indices');
        } else {
          return Promise.reject(err);
        }
      }
```

此外，`utils/routes/index.js` 还会加载 `components/setup/setup.js` 完成整个环境的检查和启动(在 Kibana4.0 时，是在第二步 kibana plugin 加载的，4.1 开始往前挪到这里)。setup 过程包括：

```
    var checkForEs = Private(require('components/setup/steps/check_for_es'));
    var checkEsVersion = Private(require('components/setup/steps/check_es_version'));
    var checkForKibanaIndex = Private(require('components/setup/steps/check_for_kibana_index'));
    var createKibanaIndex = Private(require('components/setup/steps/create_kibana_index'));

    return _.once(function () {
      return checkForEs()
      .then(checkEsVersion)
      .then(checkForKibanaIndex)
      .then(function (exists) {
        if (!exists) return createKibanaIndex();
      })
    });
```

也就是依次调用 `components/setup/steps/` 下的 `check_for_es`, `check_es_version`, `check_for_kibana_index`，如果没有 kibana index，再调用一个 `create_kibana_index`。完成。

## 第二步，kibana 插件加载

```
  kibana.load = _.onceWithCb(function (cb) {
    var firstLoad = [ 'plugins/kibana/index' ];
    var thenLoad = _.difference(configFile.plugins, firstLoad);
    require(firstLoad, function loadApps() {
      require(thenLoad, cb);
    });
  });
```

执行 `kibana.load()` 函数，先加载 `plugins/kibana/index.js`，然后加载 configFile 里定义的其他 plugins。

`plugins/kibana/index.js` 里又有一系列操作：

1. 加载 `components/courier/courier.js` 和 `components/config/config.js` 两个 angular.service；
2. 加载 `plugins/kibana/_init`, `plugins/kibana/_apps`, `plugins/kibana/_timepicker`。

`components/config/config.js` 主要是从 kibana index 里的 "config" type 中读取 "kbnVersion" id 的数据。这个 "kbnVersion" 就是之前 index.js 里的加载的第一个常量，在 grunt build 编译时会自动生成。

`plugins/kibana/_init` 里监听 `application.load` 事件，触发 `courier.start()` 函数。

`plugins/kibana/_timepicker` 提供时间选择器页面。

### courier 概述

`components/courier/courier.js` 中定义了 **Courier** 类。**Courier** 是一个非常重要的东西，可以简单理解为 kibana 跟 ES 之间的一个 object mapper。简要的说，包括一下功能：

```
    function Courier() {
      var self = this;
      var DocSource = Private(require('components/courier/data_source/doc_source'));
      var SearchSource = Private(require('components/courier/data_source/search_source'));
      var searchStrategy = Private(require('components/courier/fetch/strategy/search'));
      var requestQueue = Private(require('components/courier/_request_queue'));
      var fetch = Private(require('components/courier/fetch/fetch'));
      var docLooper = self.docLooper = Private(require('components/courier/looper/doc'));
      var searchLooper = self.searchLooper = Private(require('components/courier/looper/search'));

      self.setRootSearchSource = Private(require('components/courier/data_source/_root_search_source')).set;
      self.SavedObject = Private(require('components/courier/saved_object/saved_object'));
      self.indexPatterns = indexPatterns;
      self.redirectWhenMissing = Private(require('components/courier/_redirect_when_missing'));
      self.DocSource = DocSource;
      self.SearchSource = SearchSource;

      self.start = function () {
        searchLooper.start();
        docLooper.start();
        return this;
      };
      self.fetch = function () {
        fetch.fetchQueued(searchStrategy).then(function () {
          searchLooper.restart();
        });
      };
      self.started = function () {
        return searchLooper.started();
      };
      self.stop = function () {
        searchLooper.stop();
        return this;
      };
      self.createSource = function (type) {
        switch (type) {
        case 'doc':
          return new DocSource();
        case 'search':
          return new SearchSource();
        }
      };
      self.close = function () {
        searchLooper.stop();
        docLooper.stop();

        _.invoke(requestQueue, 'abort');

        if (requestQueue.length) {
          throw new Error('Aborting all pending requests failed.');
        }
      };
```

从类的方法中可以看出，其实主要就是五个属性的控制：

* DocSource 和 SearchSource：继承自 `components/courier/data_source/_abstract.js`，调用 `components/courier/data_source/data_source/_doc_send_to_es.js` 完成跟 ES 数据的交互，用来做 savedObject 和 index_pattern 的读写：

```
      es[method](params)
      .then(function (resp) {
        if (resp.status === 409) throw new errors.VersionConflict(resp);

        doc._storeVersion(resp._version);
        doc.id(resp._id);

        var docFetchProm;
        if (method !== 'index') {
          docFetchProm = doc.fetch();
        } else {
          // we already know what the response will be
          docFetchProm = Promise.resolve({
            _id: resp._id,
            _index: params.index,
            _source: body,
            _type: params.type,
            _version: doc._getVersion(),
            found: true
          });
        }
```

这个 es 在是调用了 `services/es.js` 里定义的 service，里面内容超级简单，就是加载官方的 elasticsearch.js 库，然后初始化一个最简的 esFactory 客户端，包括超时都设成了 0，把这个控制交给 server 端。

```
define(function (require) {
  require('elasticsearch');
  var _ = require('lodash');
  var es; // share the client amoungst all apps
  require('modules')
    .get('kibana', ['elasticsearch', 'kibana/config'])
    .service('es', function (esFactory, configFile, $q) {
      if (es) return es;

      es = esFactory({
        host: configFile.elasticsearch,
        log: 'info',
        requestTimeout: 0,
        apiVersion: '1.4'
      });
      return es;
    });
});
```

* searchLooper 和 docLooper：分别给 `Looper.start` 方法传递 searchStrategy 和 docStrategy，对应 ES 的 `/_msearch` 和 `/_mget` 请求。searchLooper 的实现如下：

```
    var fetch = Private(require('components/courier/fetch/fetch'));
    var searchStrategy = Private(require('components/courier/fetch/strategy/search'));
    var requestQueue = Private(require('components/courier/_request_queue'));
    var Looper = Private(require('components/courier/looper/_looper'));
    var searchLooper = new Looper(null, function () {
      $rootScope.$broadcast('courier:searchRefresh');
      return fetch.these(
        requestQueue.getInactive(searchStrategy)
      );
    });
```

这里的关键方法是 `fetch.these()`，出自 `components/courier/fetch/_fetch_these.js`，其中调用的 `components/courier/fetch/_call_client.js` 有如下一段代码：

```
      Promise.map(executable, function (req) {
        return Promise.try(req.getFetchParams, void 0, req)
        .then(function (fetchParams) {
          return (req.fetchParams = fetchParams);
        });
      })
      .then(function (reqsFetchParams) {
        return strategy.reqsFetchParamsToBody(reqsFetchParams);
      })
      .then(function (body) {
        return (esPromise = es[strategy.clientMethod]({
          timeout: configFile.shard_timeout,
          ignore_unavailable: true,
          preference: sessionId,
          body: body
        }));
      })
      .then(function (clientResp) {
        return strategy.getResponses(clientResp);
      })
      .then(respond)
```

在这段代码中，我们可以看到 `strategy.reqsFetchParamsToBody()`, `strategy.getResponses()` 和 `strategy.clientMethod`，正是之前 searchLooper 和 docLooper 传递的对象属性。而最终发送请求，同样用的是前面解释过的 **es** 这个 service。 

此外，Courier 还提供了自动刷新的控制功能：

```
      self.fetchInterval = function (ms) {
        searchLooper.ms(ms);
        return this;
      };
      $rootScope.$watchCollection('timefilter.refreshInterval', function () {
        var refreshValue = _.get($rootScope, 'timefilter.refreshInterval.value');
        var refreshPause = _.get($rootScope, 'timefilter.refreshInterval.pause');
        if (_.isNumber(refreshValue) && !refreshPause) {
          self.fetchInterval(refreshValue);
        } else {
          self.fetchInterval(0);
        }
      });
```

### 路径记忆功能的实现

`plugins/kibana/_apps.js` 中，我们可以看到路径记忆功能是怎么实现的：

```
    function appKey(app) {
      return 'lastPath:' + app.id;
    }
    function assignPaths(app) {
      app.rootPath = '/' + app.id;
      app.lastPath = sessionStorage.get(appKey(app)) || app.rootPath;
      return app.lastPath;
    }
    function getShow(app) {
      app.show = app.order >= 0 ? true : false;
    }
    function setLastPath(app, path) {
      app.lastPath = path;
      return sessionStorage.set(appKey(app), path);
    }

    $scope.apps = Private(require('registry/apps'));
    // initialize each apps lastPath (fetch it from storage)
    $scope.apps.forEach(assignPaths);
    $scope.apps.forEach(getShow);

    function onRouteChange() {
      var route = $location.path().split(/\//);
      $scope.apps.forEach(function (app) {
        if (app.active = app.id === route[1]) {
          $rootScope.activeApp = app;
        }
      });
      if (!$rootScope.activeApp || $scope.appEmbedded) return;
      setLastPath($rootScope.activeApp, globalState.removeFromUrl($location.url()));
    }

    $rootScope.$on('$routeChangeSuccess', onRouteChange);
    $rootScope.$on('$routeUpdate', onRouteChange);
```

这里使用的 `sessionStorage` 是 HTML5 自带的新特性，这样，每次标签页切换的时候，都可以把 `$location.url` 保存下来。至于整个 Kibana 页面上标签页的初始状态，则通过 `registry/apps.js` 获取。

### 插件的加载

那么，各标签页插件是怎么进到 *registry/apps* 里的呢？

之前我们已经说过，index.js 一开始加载完 kibana 以后，会挨个 `require(configFile.plugins)`。这个 configFile.plugins 按说就应该是列出来各个标签页了，但实际上，K4 的配置文件 `server/config/kibana.yml` 里并没有一个参数叫 `plugins`。所以，还得看看 server 端的实现了。

和阅读 Kibana 主页入口一样，找到 server 端的主入口 `server/index.js`：

```
var requirePlugins = require('./lib/plugins/require_plugins');
var extendHapi = require('./lib/extend_hapi');
function Kibana(settings, plugins) {
  plugins = plugins || [];
  this.server = new Hapi.Server();
  extendHapi(this.server);
  var config = this.server.config();
  if (settings) config.set(settings);

  this.plugins = [];
  var externalPluginsFolder = config.get('kibana.externalPluginsFolder');
  if (externalPluginsFolder) {
    this.plugins = _([externalPluginsFolder])
      .flatten()
      .map(requirePlugins)
      .flatten()
      .value();
  }
  this.plugins = this.plugins.concat(plugins);
```

根据这段代码，我们知道，K4 的 server 端，统一也有插件机制，由 `server/lib/plugins/require_plugins.js` 加载内置 server 插件，然后再加上外部 server 插件目录。requirePlugins 中查找内置插件的方法如下：

```
  globPath = globPath || join(__dirname, '..', '..', 'plugins', '*', 'index.js');
  return glob.sync(globPath).map(function (file) {
    var module = require(file);
    var regex = new RegExp('([^/]+)/index.js');

    var matches = file.match(regex);
    if (!module.name && matches) {
      module.name = matches[1];
    }
```

也就是说，加载 `server/plugins/` 下所有子目录的 index.js。目前 K4 自带的有：config, elasticsearch, static, status。分别用来返回 config 数据，代理 ES 请求，处理纯静态文件请求，显示 server 端各插件状态。

我们具体看这个 `server/plugins/config/index.js` 的内容：

```
var listPlugins = require('../../lib/plugins/list_plugins');

module.exports = new kibana.Plugin({
  init: function (server, options) {

    server.route({
      method: 'GET',
      path: '/config',
      handler: function (request, reply) {
        var config = server.config();
        reply({
          kibana_index: config.get('kibana.index'),
          default_app_id: config.get('kibana.defaultAppId'),
          shard_timeout: config.get('elasticsearch.shardTimeout'),
          plugins: listPlugins(server)
        });
      }
    });

  }
});
```

很明显，一个标准的 node.js 的 route，用来响应对 "/config" 这个地址的 GET 请求，返回一个哈希 JSON，其中就有 plugins。没错，我们前面说的 configFiles.plugins 就是从这里获得的。

下面看这个 `listPlugins` 的实现。

```
    var config = server.config();
    var bundled_plugins = plugins(config.get('kibana.bundledPluginsFolder'));
    var external_plugins = _(server.plugins).map(function (plugin, name) {
      return plugin.self && plugin.self.publicPlugins || [];
    }).flatten().value();
```

这个 "bundledPluginsFolder" 也不是我们 kibana.yml 里存在的参数设置。所以，还得看上面这行 `server.config()` 了。回到最早先的 `server/index.js` 里，其实在 require_plugins 后面，还有一个 extend_hapi。Hapi 是 nodejs 的一个可扩展框架。我们看到 index.js 中，正是在 `extendHapi(this.server)` 后第一次获取了 `server.config()`。

extendHapi 只有两行：

```
  server.decorate('server', 'config', require('./config'));
  server.decorate('server', 'loadKibanaPlugins', require('./plugins/load_kibana_plugins'));
```

这个 `server/lib/config/index.js` 主要加载同目录下的：config.js 用来实现 Config 类，schema.js 用来实现具体的 Joi 对象。

然后我们看这个 `server/lib/config/schema.js`，就会发现各种配置属性全在这里了~和插件路径相关的几行如下：

```
var publicFolder = path.resolve(__dirname, '..', '..', 'public');
if (!checkPath(publicFolder)) publicFolder = path.resolve(__dirname, '..', '..', '..', 'kibana');

var bundledPluginsFolder = path.resolve(publicFolder, 'plugins');

module.exports = Joi.object({
  kibana: Joi.object({
    bundledPluginsFolder: Joi.string().default(bundledPluginsFolder),
```

这里有两个路径，因为一个是源码位置，一个是 grunt build 编译后的位置。就我们阅读源码来说，这个 `__dirname/../../../kibana/plugins` 就是我们最终找到的地方了，我们从 server 端源码里找了一圈，终于回到 kibana 前端页面的源码目录中，这就是 `kibana/plugins` 目录，自动加载其下所有子目录为内置插件。包括：

* dashboard
* discover
* doc
* kbn_vislib_vis_types
* kibana
* markdown_vis
* metric_vis
* settings
* table_vis
* vis_debug_spy
* visualize

除 kibana 以外，随意进一个，(还记得 index.js 里是把 kibana 去除掉了吧)，比如 visualize，看 `visualize/index.js` 的内容，最底下有这么一段：

```
  var apps = require('registry/apps');
  apps.register(function VisualizeAppModule() {
    return {
      id: 'visualize',
      name: 'Visualize',
      order: 1
    };
  });
```

其他目录也都一样。

这个 `registry/apps.js` 主要是加载 `registry/_registry.js`，把注册的 app 存入 `utils/indexed_array/index` 的 IndexedArray 对象。对象主要有几个值：id, name, order。前面说到的路径记忆功能要用的两个方法，assignPaths 里就是用 app.id 设置 lastPath，而 getShow 里就是用 order 来判断是否展示在页面上。

下一章，我们开始介绍官方提供的几个 apps。
