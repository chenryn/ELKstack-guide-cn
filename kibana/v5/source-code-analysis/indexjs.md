# 主页入口

我们先从启动 Kibana 的命令行程序入手，可以看到这是一个 shell 脚本。最终执行的是 `node src/cli serve` 命令。然后跟着就可以找到 `src/cli/serve` 程序，其中最重要的是加载了 `src/server/kbn_server.js`。继续打开，可以看到它先后加载了 config, http, logging, plugin 和 uiExports。毫无疑问，其中重点是 http 和 uiExports 部分。

`http/index.js` 中，初始化了 Hapi.Server 对象，加载 hapi plugin，并声明了主要的 route。包括静态文件、模板文件、短地址跳转和主页默认跳转到 `/app/kibana`。目前来说 Kibana 在服务器端主动做的事情还比较少。在我们不基于 Hapi 框架做二次开发的情况下，不用过于关注这期间 Kibana 做了什么。

下面进入 `src/ui/` 目录继续。

`src/ui/index.js` 中完成了更细节的各类 app 的加载和路由分配：

```
const uiExports = kbnServer.uiExports = new UiExports({
  urlBasePath: config.get('server.basePath')
});
for (let plugin of kbnServer.plugins) {
  uiExports.consumePlugin(plugin);
}

const bundles = kbnServer.bundles = new UiBundleCollection(bundlerEnv, config.get('optimize.bundleFilter'));

for (let app of uiExports.getAllApps()) {
  bundles.addApp(app);
}
server.route({
  path: '/app/{id}',
  method: 'GET',
  handler: function (req, reply) {
    const id = req.params.id;
    const app = uiExports.apps.byId[id];
    if (!app) return reply(Boom.notFound('Unknown app ' + id));

    if (kbnServer.status.isGreen()) {
      return reply.renderApp(app);
    } else {
      return reply.renderStatusPage();
    }
  }
});
```

可以看到这里把所有的 app 都打包进了 bundle。这也是很多初次接触 Kibana 二次开发的新手很容易被绊倒的一点——改了一行代码怎么没生效？因为服务是优先使用 bundle 内容的，而不会每次都进到各源码目录执行。

如果确实在频繁修改代码的阶段，每次都等 bundle 确实太累了，可以看到上面代码段里有一个 `config.get('optimize.bundleFilter')`。是的，其实 Kibana 支持在 config 中设定具体的 optimize 行为，但是官方文档上并没有介绍。最完整的配置项，见 `src/server/config/schema.js`。前文说过，这是在启动 `kbn_server` 的时候最先加载的。

在 schema 中可以看到一个很可爱的配置：

```
optimize: _joi2['default'].object({
  enabled: _joi2['default'].boolean()['default'](true),
})
```

所以你只要在 `config/kibana.yml` 中加上这么一行配置就好了：`optimize.enabled: false`。

## kibana app

从 Kibana 4.5 版开始，Kibana 框架和 Kibana App 做了一个剥离。现在，我们进到 Kibana App 里看看。路径在 `src/core_plugins/kibana`。

我们可以看到路径中有如下文件：

* common/
* index.js
* package.json
* public/
* server/

这是一个很显然的普通 nodejs 模块的结构。我们可以看看作为模块描述的 package.json 里写了啥：

```
{
  "name": "kibana",
  "version": "kibana"
}
```

非常有趣的 version。事实上这个写法的意思是本插件的版本号和 Kibana 框架的版本号保持一致。事实上所有 core\_plugins 的版本号都写的是 kibana。

然后 index.js 中，调用 uiExports 完成了 app 注册。也这是之后我们自己开发新的 Kibana 应用时必须做的。我们下面摘主要段落分别看一下：

```
module.exports = function (kibana) {
  var kbnBaseUrl = '/app/kibana';
  return new kibana.Plugin({
    id: 'kibana',
    config: function config(Joi) {
      return Joi.object({
        enabled: Joi.boolean()['default'](true),
        defaultAppId: Joi.string()['default']('discover'),
        index: Joi.string()['default']('.kibana')
      })['default']();
    },
    uiExports: {
      app: {
         id: 'kibana',
         title: 'Kibana',
         listed: false,
         description: 'the kibana you know and love',
         main: 'plugins/kibana/kibana',
```

这是最基础的部分，注册成为一个 `kibana.Plugin`，id 叫什么，config 配置有什么，标题叫什么，入口文件是哪个，具体是什么类型的 uiExports，一般常见的选择有：app、visType。这两者也是做 Kibana 二次开发最容易入手的地方。

```
         uses: ['visTypes', 'spyModes', 'fieldFormats', 'navbarExtensions', 'managementSections', 'devTools', 'docViews'],
         injectVars: function injectVars(server, options) {...}
      },
```

uses 和 injectVars 是可选的方式，可以在 `src/ui/ui_app.js` 中看到起作用。分别是指明下列模块已经加载过，以后就不用再加载了；以及声明需要注入浏览器的 JSON 变量。

```
      links: [{
        id: 'kibana:discover',
        title: 'Discover',
        order: -1003,
        url: kbnBaseUrl + '#/discover',
        description: 'interactively explore your data',
        icon: 'plugins/kibana/assets/discover.svg'
      }, {
        ...
      }],
    },
```

这里是一个特殊的地方，一般来说其他应用不会用到 links 类型的 uiExports。因为 Kibana 应用本身不用单一的左侧边栏切换，而是需要把自己内部的 Discover、Visualize、Dashboard、Management 功能放上去。所以定义里，把自己的 `listed` 给 false 了，而把这具体的四项通过 links 的方式，添加到侧边栏上。links 具体可配置的属性，见 `src/ui/ui_nav_link.js`。这里就不细讲了。

```
    preInit: _asyncToGenerator(function* (server) {
        yield mkdirp(server.config().get('path.data'));
    }),
```

preInit 也是一个可选属性，如果有需要创建目录之类的要预先准备的操作，可以在这步完成。

```
    init: function init(server, options) {
       // uuid
       (0, _serverLibManage_uuid2['default'])(server);
       // routes
       (0, _serverRoutesApiIngest2['default'])(server);
       (0, _serverRoutesApiSearch2['default'])(server);
       (0, _serverRoutesApiSettings2['default'])(server);
       (0, _serverRoutesApiScripts2['default'])(server);

       server.expose('systemApi', systemApi);
    }
  });
}
```

init 是最后一步。我们看到 Kibana 应用的最后一步是继续加载了一些服务器端的 route 设置。比如这个 `_serverRoutesApiScripts2`，具体代码是在 `src/core_plugins/kibana/server/routes/api/scripts/register_languages.js` 里：

```
server.route({
  path: '/api/kibana/scripts/languages',
  method: 'GET',
  handler: function handler(request, reply) {
    var callWithRequest = server.plugins.elasticsearch.callWithRequest;
    return callWithRequest(request, 'cluster.getSettings', {
      include_defaults: true,
      filter_path: '**.script.engine.*.inline'
    }).then(function (esResponse) {
      var langs = _lodash2['default'].get(esResponse, 'defaults.script.engine', {});
      var inlineLangs = _lodash2['default'].pick(langs, function (lang) {
        return lang.inline === 'true';
      });
      var supportedLangs = _lodash2['default'].omit(inlineLangs, 'mustache');
      return _lodash2['default'].keys(supportedLangs);
    }).then(reply)['catch'](function (error) {
      reply((0, _libHandle_es_error2['default'])(error));
    });
  }
});
```

我在之前 K4 源码解析中曾经讲过的一个二次开发场景 —— 切换脚本引擎支持。在 Elasticsearch 5.0 中，在 `/_cluster/settings` 接口里提供了具体的可用引擎细节，诸如一个个 `script.engine.painless.inline` 的列表。这样，就不必像之前那样明确知道自己可以用什么，然后硬改代码来支持了；而是可以通过这个接口数据，拿到集群实际支持什么引擎，当前默认是什么引擎等设置，直接在 Kibana 中使用。上面这段代码，就是提供了这个数据。

*注意其中排除了 mustache，因为它只能做模板渲染，没法做字段值计算。*

好了。应用注册完成，我们看到了 main 入口，那么去看看 main 入口的内容吧。打开 `src/core_plugins/kibana/public/kibana.js`。主要如下：

```
import kibanaLogoUrl from 'ui/images/kibana.svg';
import 'ui/autoload/all';
import 'plugins/kibana/discover/index';
import 'plugins/kibana/visualize/index';
import 'plugins/kibana/dashboard/index';
import 'plugins/kibana/management/index';
import 'plugins/kibana/doc';
import 'plugins/kibana/dev_tools';
import 'ui/vislib';
import 'ui/agg_response';
import 'ui/agg_types';
import 'ui/timepicker';
import Notifier from 'ui/notify/notifier';
import 'leaflet';

routes.enable();

routes
.otherwise({
  redirectTo: `/${chrome.getInjected('kbnDefaultAppId', 'discover')}`
});

chrome
.setRootController('kibana', function ($scope, courier, config) {
  $scope.$on('application.load', function () {
    courier.start();
  });
  ...
});
```

基本上通过这一串 import 就可以看到 Kibana 中最主要的各项功能了。

而对内部比较重要的则是这个 `ui/autoload/all`。这里面其实是加载了 kibana 自定义的各种 angular module、directive 和 filter。像我们熟悉的 markdown、moment、auto\_select、json\_input、paginate、file\_upload 等都在这里面加载。这些都是网页开发的通用工具，这里就不再介绍细节了，有兴趣的读者可以在 `src/ui/public/` 下找到对应文件。

设置 routes 的具体操作在加载的 `src/ui/public/routes/route_manager.js` 文件里，其中会调用 `sr/ui/public/index_patterns/route_setup/load_default.js` 中提供的 `addSetupWork` 方法，在未设置 default index pattern 的时候跳转 URL 到 `whenMissingRedirectTo` 页面。

```
  uiRoutes
  .addSetupWork(...)
  .afterWork(
    // success
    null,

    // failure
    function (err, kbnUrl) {
       let hasDefault = !(err instanceof NoDefaultIndexPattern);
       if (hasDefault || !whenMissingRedirectTo) throw err; // rethrow

       kbnUrl.change(whenMissingRedirectTo);
       if (!defaultRequiredToasts) defaultRequiredToasts = [];
       else defaultRequiredToasts.push(notify.error(err));
    }
  )
```

而这个 `whenMissingRedirectTo` 页面是在 kibana 应用的源码里写死的，见 `src/core_plugins/kibana/public/management/index.js`：

```
uiRoutes
.when('/management', {
      template: landingTemplate
});

require('ui/index_patterns/route_setup/load_default')({
      whenMissingRedirectTo: '/management/kibana/index'
});
```

在原先的版本中，routes 里面还会检查 Elasticsearch 的版本号，在 5.0 版里，这件事情从 kibana plugin 改到 elasticsearch plugin 里完成了。

### courier 概述

kibana.js 的最后，控制器则会监听 `application.load` 事件，在页面加载完成的时候触发 `courier.start()` 函数。

`src/ui/public/courier/courier.js` 中定义了 **Courier** 类。**Courier** 是一个非常重要的东西，可以简单理解为 kibana 跟 ES 之间的一个 object mapper。简要的说，包括一下功能：

```
    import DocSourceProvider from './data_source/doc_source';
    ...
    function Courier() {
      var self = this;
      var DocSource = Private(DocSourceProvider);
      self.DocSource = DocSource;
      ...

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

* DocSource 和 SearchSource：继承自 `src/ui/public/courier/data_source/_abstract.js`，调用 `src/ui/public/courier/data_source/data_source/_doc_send_to_es.js` 完成跟 ES 数据的交互，用来做 savedObject 和 index\_pattern 的读写：

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

这个 es 在是调用了 `src/ui/public/es.js` 里定义的 service，里面内容超级简单，就是加载官方的 elasticsearch.js 库，然后初始化一个最简的 esFactory 客户端，包括超时都设成了 0，把这个控制交给 server 端。

```
import 'elasticsearch-browser';
import _ from 'lodash';
import uiModules from 'ui/modules';

let es; // share the client amongst all apps
uiModules
  .get('kibana', ['elasticsearch', 'kibana/config'])
  .service('es', function (esFactory, esUrl, $q, esApiVersion, esRequestTimeout) {
    if (es) return es;
    es = esFactory({
      host: esUrl,
      log: 'info',
      requestTimeout: esRequestTimeout,
      apiVersion: esApiVersion,
      plugins: [function (Client, config) {
        // esFactory automatically injects the AngularConnector to the config
        // https://github.com/elastic/elasticsearch-js/blob/master/src/lib/connectors/angular.js
        _.class(CustomAngularConnector).inherits(config.connectionClass);
        function CustomAngularConnector(host, config) {
          CustomAngularConnector.Super.call(this, host, config);

          this.request = _.wrap(this.request, function (request, params, cb) {
          if (String(params.method).toUpperCase() === 'GET') {
            params.query = _.defaults({ _: Date.now()  }, params.query);
          }
          return request.call(this, params, cb);
        });
      }
      config.connectionClass = CustomAngularConnector;
    }]
  });
  return es;
});
```

* searchLooper 和 docLooper：分别给 `Looper.start` 方法传递 searchStrategy 和 docStrategy，对应 ES 的 `/_msearch` 和 `/_mget` 请求。searchLooper 的实现如下：

```
import FetchProvider from '../fetch';
import SearchStrategyProvider from '../fetch/strategy/search';
import RequestQueueProvider from '../_request_queue';
import LooperProvider from './_looper';

export default function SearchLooperService(Private, Promise, Notifier, $rootScope) {
    let fetch = Private(FetchProvider);
    let searchStrategy = Private(SearchStrategyProvider);
    let requestQueue = Private(RequestQueueProvider);

    let Looper = Private(LooperProvider);
    let searchLooper = new Looper(null, function () {
      $rootScope.$broadcast('courier:searchRefresh');
      return fetch.these(
        requestQueue.getInactive(searchStrategy)
      );
    });
    ...
```

这里的关键方法是 `fetch.these()`，出自 `src/ui/public/courier/fetch/fetch_these.js`，其中调用的 `src/ui/public/courier/fetch/call_client.js` 有如下一段代码：

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
        return (esPromise = es[strategy.clientMethod]({ body }));
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
      ...
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

`src/ui/public/chrome/api/apps.js` 中，我们可以看到路径记忆功能是怎么实现的：

```
module.exports = function (chrome, internals) {
  internals.appUrlStore = internals.appUrlStore || window.sessionStorage;
  ...
  chrome.getLastUrlFor = function (appId) {
    return internals.appUrlStore.getItem(`appLastUrl:${appId}`);
  };
  chrome.setLastUrlFor = function (appId, url) {
    internals.appUrlStore.setItem(`appLastUrl:${appId}`, url);
  };
```

这里使用的 `sessionStorage` 是 HTML5 自带的新特性，这样，每次标签页切换的时候，都可以把 `$location.url` 保存下来。

