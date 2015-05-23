## 入口和模块依赖

这一部分是网页项目的基础。从 index.html 里就可以学到 angularjs 最基础的常用模板语法了。出现的指令有：`ng-repeat`, `ng-controller`, `ng-include`, `ng-view`, `ng-slow`, `ng-click`, `ng-href`，以及变量绑定的语法：`{{ dashboard.current.** }}`。

index.html 中，需要注意 js 的加载次序，先 `require.js`，然后再 `require.config.js`，最后 `app`。整个 kibana 项目都是通过 **requrie** 方式加载的。而具体的模块，和模块的依赖关系，则定义在 `require.config.js` 里。这些全部加载完成后，才是启动 app 模块，也就是项目本身的代码。

require.config.js 中，主要分成两部分配置，一个是 `paths`，一个是 `shim`。paths 用来指定依赖模块的导出名称和模块 js 文件的具体路径。而 shim 用来指定依赖模块之间的依赖关系。比方说：绘制图表的 js，kibana3 里用的是 jquery.flot 库。这个就首先依赖于 jquery 库。(通俗的说，就是原先普通的 HTML 写法里，要先加载 jquery.js 再加载 jquery.flot.js)

在整个 paths 中，需要单独提一下的是 `elasticjs:'../vendor/elasticjs/elastic-angular-client'`。这是串联 elastic.js 和 angular.js 的文件。这里面实际是定义了一个 angular.module 的 factory，名叫 `ejsResource`。后续我们在 kibana 3 里用到的跟 Elasticsearch 交互的所有方法，都在这个 `ejsResource` 里了。

*factory 是 angular 的一个单例对象，创建之后会持续到你关闭浏览器。Kibana 3 就是通过这种方式来控制你所有的图表是从同一个 Elasticsearch 获取的数据*

app.js 中，定义了整个应用的 routes，加载了 controller, directives 和 filters 里的全部内容。就是在这里，加载了主页面 `app/partials/dashboard.html`。当然，这个页面其实没啥看头，因为里面就是提供 pulldown 和 row 的 div，然后绑定到对应的 controller 上。

