## controller 和 service

controller 里没太多可讲的。kibana 3 里，pulldown 其实跟 row 差别不大，看这简单的几行代码里，最关键的就是几个注入：

```
define(['angular','app','lodash'], function (angular, app, _) {
  'use strict';
  angular.module('kibana.controllers').controller('RowCtrl', function($scope, $rootScope, $timeout,ejsResource, querySrv) {
      var _d = {
        title: "Row",
        height: "150px",
        collapse: false,
        collapsable: true,
        editable: true,
        panels: [],
        notice: false
      };
      _.defaults($scope.row,_d);

      $scope.init = function() {
        $scope.querySrv = querySrv;
        $scope.reset_panel();
      };
      $scope.init();
    }
  );
});
```

这里面，注入了 `$scope`, `ejsResource` 和 `querySrv`。`$scope` 是控制器作用域内的模型数据对象，这是 angular 提供的一个特殊变量。`ejsResource` 是一个 factory ，前面已经讲过。`querySrv` 是一个 service，下面说一下。

service 跟 factory 的概念非常类似，一般来说，可能 factory 偏向用来共享一个类，而 service 用来共享一组函数功能。

kibana 3 里，比较有用和常用的 services 包括：

### dashboard

dashboard.js 里提供了关于 Kibana 3 仪表板的读写操作。其中主要的几个是提供了三种读取仪表板布局纲要的方式，也就是读取文件，读取存在 `.kibana-int` 索引里的数据，读取 js 脚本。下面是读取 js 脚本的相关函数：

```
    this.script_load = function(file) {
      return $http({
        url: "app/dashboards/"+file.replace(/\.(?!js)/,"/"),
        method: "GET",
        transformResponse: function(response) {
          /*jshint -W054 */
          var _f = new Function('ARGS','kbn','_','moment','window','document','angular','require','define','$','jQuery',response);
          return _f($routeParams,kbn,_,moment);
        }
      }).then(function(result) {
        if(!result) {
          return false;
        }
        self.dash_load(dash_defaults(result.data));
        return true;
      },function() {
        alertSrv.set('Error',
          "Could not load <i>scripts/"+file+"</i>. Please make sure it exists and returns a valid dashboard" ,
          'error');
        return false;
      });
    };
```

可以看到，最关键的就是那个 `new Function`。知道这步传了哪些函数进去，也就知道你的 js 脚本里都可以调用哪些内容了~

最后调用的 `dash_load` 方法也需要提一下。这个方法的最后，有几行这样的代码：

```
      self.availablePanels = _.difference(config.panel_names,
        _.pluck(_.union(self.current.nav,self.current.pulldowns),'type'));

      self.availablePanels = _.difference(self.availablePanels,config.hidden_panels);
```

从最外层的 `config.js` 里读取了 `panel_names` 数组，然后取出了 nav 和 pulldown 用过的 panel，剩下就是我们能在 row 里添加的 panel 类型了。

### querySrv

querySrv.js 里定义了跟 query 框相关的函数和属性。主要有几个值得注意的。

* 一个是 `color` 列表；
* 一个是 `queryTypes`，尤其是里么的 `topN`，可以看到 topN 方式其实就是先请求了一次 termsFacet，然后把结果 map 成一组普通的 query。
* 一个是 `ids` 和 `idsByMode`。之后图表的绑定具体 query 的时候，就是通过这个函数来选择的。

### filterSrv

filterSrv.js 跟 querySrv 相似。特殊的是两个函数。

* 一个是 `toEjsObjs`。根据不同的 filter 类型调用不同的 ejs 方法。
* 一个是 `timeRange`。因为在 histogram panel 上拖拽，会生成好多个 range 过滤器，都是时间。这个方法会选择最后一个类型为 time 的 filter，作为实际要用的 filter。这样保证请求 ES 的是最后一次拖拽选定的时间段。

### fields

fields.js 里最重要的作用就是通过 mapping 接口获取索引的字段列表，存在 `fields.list` 里。这个数组后来在每个 panel 的编辑页里，都以 `bs-typeahead="fields.list"` 的形式作为文本输入时的自动补全提示。在 table panel 里，则是左侧栏的显示来源。

### esVersion

esVersion.js 里提供了对 ES 版本号的对比函数。之所以专门提供这么个 service，一来是因为不同版本的 ES 接口有变化，比如我自己开发的 percentile panel 里，就用 esVersion 判断了两次版本。因为 percentile 接口是 1.0 版之后才有，而从 1.3 版以后返回数据的结构又发生了一次变动。二来 ES 的版本号格式比较复杂，又有点又有字母。

