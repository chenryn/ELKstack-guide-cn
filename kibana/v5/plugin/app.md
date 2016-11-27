# app 开发

前面两节，我们分别看过了如何开发可视化部分和服务器端部分。现在，我们把这两头综合起来，做一个可以在 Kibana 菜单栏上切换使用的，完整的 app。就像 Kibana 5 默认分发的 timelion 和 console 那样。

当然我们这里不会真的特意搞一个很复杂的可视化应用。我们只做一个 Elasticsearch 状态展示页面就好了。这个方式正好可以串联从前到后的请求、展示部分。

## app 模块的 index.js 结构

我们已经讲过了在 index.js 中如何使用 uiExports.visType 和 init。那么 app 的 index.js 是什么样子的呢？

```
export default function (kibana) {
    return new kibana.Plugin({
        require: ['elasticsearch'],

        uiExports: {
            app: {
                title: 'Indices',
                description: 'An awesome Kibana plugin',
                main: 'plugins/elasticsearch_status/app',
                icon: 'plugins/elasticsearch_status/icon.svg'
            }
        }
    });
}
```

这个示例中有两处特殊的代码：

1. `require` 指令加载了 elasticsearch 模块；这表示后续我们会用到这个模块，所以提前加载好。
2. `uiExports` 中使用了 app 键值对定义。其中这几对键值的含义如下：
    * title: app 的名称，用来显示在 Kibana 左侧边栏上的文字；
    * icon: app 的图表，用来显示在 Kibana 左侧边栏上的图标；
    * main: app 的主入口 js 文件。

### 服务器端部分

作为完整的 app，自然也还是有服务器端的部分。上节已经讲过，在 index.js 中的 init 部分定义这块：

```
return new kibana.Plugin({
      // ...
    init(server, options) {
        server.route({
            path: '/api/elasticsearch_status/index/{name}',
            method: 'GET',
            handler(req, reply) {
                server.plugins.elasticsearch.callWithRequest(req, 'cluster.state', {
                    metric: 'metadata',
                    index: req.params.name
                }).then(function (response) {
                    reply(response.metadata.indices[req.params.name]);
                });
            }
        });
    }
});
```

传入的 server 参数，我们在上节只是用来获取了一下 ESClient。这次，我们正式使用一些 Hapi.js 真正的功能。比如路由。

这里创建的是一个可以被 GET 方法访问的 URL 地址 `/api/elasticsearch_status/index/<index name>`。对访问的处理应答，使用 handler 方法。

handler 方法传入两个参数：

* req: 有关请求的信息都可以通过 req 对象获取，比如路由中捕获的 `<index name>` 就可以用 `req.params.name` 来引用。
* reply: 应答处理的内容通过 reply 返回。

然后采用 `server.plugins.elasticsearch.callWithRequest` 来发送 Elasticsearch 请求，通过 Promise.then 来异步返回最终的 reply。这部分和上节讲的类似，就不展开了。

## 前台界面的 app.js

index.js 和后台数据已经就绪，下面就是前台界面展示问题。我们已经在 index.js 里定义过了 main 文件，是 app.js。

The first two lines we will insert into the file are the following:

```
import 'ui/autoload/styles';
import './less/main.less';
import uiRoutes from 'ui/routes';
import uiModules from 'ui/modules';

import overviewTemplate from './templates/index.html';
import detailTemplate from './templates/detail.html';

uiRoutes.enable();
uiRoutes
.when('/', {
      template: overviewTemplate,
      controller: 'elasticsearchStatusController',
      controllerAs: 'ctrl'

})
.when('/index/:name', {
      template: detailTemplate,
      controller: 'elasticsearchDetailController',
      controllerAs: 'ctrl'

});
```

文件第一行永远要是加载 `ui/autoload/styles`。这一行的作用是保证你的 app 界面和 Kibana 总体保持统一风格。这也是 Kibana5 才有的新内容。

然后通过 `uiRoutes` 来完成 Angular 框架的路由定义。这方面在之前的 Kibana 源码介绍中已经反复出现过。这里我们定义好路由对应的控制器和模板文件。

### 控制器

作为一个简单示例，我们可以直接在 app.js 里继续实现控制器部分。如果是复杂应用，一般这里可以拆分成单独文件。

在 Kibana 中，实现控制器的方式如下：

```
uiModules
.get('app/elasticsearch_status')
.controller('elasticsearchStatusController', function ($http) {
    $http.get('../api/elasticsearch_status/indices').then((response) => {
        this.indices = response.data;
    });
});
```

这里采用的是 Kibana 框架已经封装好的 `uiModules.get().controller()`，比标准的 `angular.module` 省去了一些创建声明、依赖处理之类的工作。同样也是之前的源码讲解里很熟悉的部分了。

这里作为和后端的配合，我们使用 Angular 标准的 `$http` 来调用 `../api/elasticsearch_status/indices` 地址。这正是之前我们声明好的服务器端 URL。

### 页面模板

Angular 模板语言也是已经见过很多面的老朋友了，这块我们用最简单的一个 `ng-repeat` 循环展示列表即可。

```
<div class="container">
  <div class="row">
    <div class="col-12-sm">
      <h1>Elasticsearch Status</h1>
      <ul class="indexList">
        <li ng-repeat="index in ctrl.indices">
          <a href="#/index/{{index}}">{{ index }}</a>
        </li>
      </ul>
    </div>
  </div>
</div>
```

完毕。一个带有前后台乃至菜单栏的完整 app 就是这么简单：

![](https://www.timroes.de/images/kibana-plugins/kibana-app.gif)
