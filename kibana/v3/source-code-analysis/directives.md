## panel 相关指令

### 添加 panel

前面在讲 `app/services/dashboard.js` 的时候，已经说到能添加的 panel 列表是怎么获取的。那么panel 是怎么加上的呢？

同样是之前讲过的 `app/partials/dashaboard.html` 里，加载了 `partials/roweditor.html` 页面。这里有一段：

```
    <form class="form-inline">
      <select class="input-medium" ng-model="panel.type" ng-options="panelType for panelType in dashboard.availablePanels|stringSort"></select>
      <small ng-show="rowSpan(row) > 11">
        Note: This row is full, new panels will wrap to a new line. You should add another row.
      </small>
    </form>

    <div ng-show="!(_.isUndefined(panel.type))">
      <div add-panel="{{panel.type}}"></div>
    </div>
```

这个 `add-panel` 指令，是有 `app/directives/addPanel.js` 提供的。方法如下：

```
          $scope.$watch('panel.type', function() {
            var _type = $scope.panel.type;
            $scope.reset_panel(_type);
            if(!_.isUndefined($scope.panel.type)) {
              $scope.panel.loadingEditor = true;
              $scope.require(['panels/'+$scope.panel.type.replace(".","/") +'/module'], function () {
                var template = '<div ng-controller="'+$scope.panel.type+'" ng-include="\'app/partials/paneladd.html\'"></div>';
                elem.html($compile(angular.element(template))($scope));
                $scope.panel.loadingEditor = false;
              });
            }
          });
```

可以看到，其实就是 require 了对应的 `panels/xxx/module.js`，然后动态生成一个 div，绑定到对应的 controller 上。

### 展示 panel

还是在 `app/partials/dashaboard.html` 里，用到了另一个指令 `kibana-panel`：

```
            <div
              ng-repeat="(name, panel) in row.panels|filter:isPanel"
              ng-cloak ng-hide="panel.hide"
              kibana-panel type='panel.type' resizable
              class="panel nospace" ng-class="{'dragInProgress':dashboard.panelDragging}"
              style="position:relative"  ng-style="{'width':!panel.span?'100%':((panel.span/1.2)*10)+'%'}"
              data-drop="true" ng-model="row.panels" data-jqyoui-options
              jqyoui-droppable="{index:$index,mutate:false,onDrop:'panelMoveDrop',onOver:'panelMoveOver(true)',onOut:'panelMoveOut'}">
            </div>
```

当然，这里面还有 `resizable` 指令也是自己实现的，不过一般我们用不着关心这个的代码实现。

下面看 `app/directives/kibanaPanel.js` 里的实现。

这个里面大多数逻辑跟 addPanel.js 是一样的，都是为了实现一个指令嘛。对于我们来说，关注点在前面那一大段 HTML 字符串，也就是变量 `panelHeader`。这个就是我们看到的实际效果中，kibana 3 每个 panel 顶部那个小图标工具栏。仔细阅读一下，可以发现除了每个 panel 都一致的那些 span 以外，还有一段是：

```
           '<span ng-repeat="task in panelMeta.modals" class="row-button extra" ng-show="task.show">' +
              '<span bs-modal="task.partial" class="pointer"><i ' +
                'bs-tooltip="task.description" ng-class="task.icon" class="pointer"></i></span>'+
            '</span>'
```

也就是说，每个 panel 可以在自己的 panelMeta.modals 数组里，定义不同的小图标，弹出不同的对话浮层。我个人给 table panel 二次开发加入的 exportAsCsv 功能，图标就是在这里加入的。

