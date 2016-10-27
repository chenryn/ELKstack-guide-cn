# settings

`plugins/settings/index.js` 结构跟 visualize 类似，注册到 registry；设置默认跳转到 `/settings/indices` 的 route，提供一个 `kbnSettingsApp` 的directive。其中关联到 `plugins/settings/sections/index.js` 内注册的 indices, advanced, objects, about 四个区块。

因为结构基本类似，这里只介绍一个比较有趣的地方。 indices 中的 scripted_field，原本的设计中，是利用 groovy sandbox 来支持的。但是就在 kibana 4 正式版要发布的几天前，groovy sandbox 出安全漏洞，Elasticsearch 紧急取消掉了 groovy 的默认开启设置。同理，`plugins/settings/sections/indices/scripted_fields/index.js` 里也改成了 "expression" 引擎。

但是，如果私有集群在防火墙内部，依然可以开启 groovy sandbox 的，其实还是可以继续使用的。在稍后的章节中，我们会介绍如何直接修改 kibana_index 完成。而这里，我们则深入理解相关代码，介绍为什么不用修改 kibana 4 源码，就能继续使用。

scripted field 的修改页面，在 `plugins/settings/sections/indices/_scripted_fields.js`。这里面的 `$scope.columns` 数组，包括 name, script, type, popularity, controls 五列，也就是我们在页面上看到的内容。看起来似乎没有定义选用的引擎。

那往上一层，看 scripted field 相关入口 `plugins/settings/sections/indices/scripted_fields/index.js` 里，在 `$scope.submit()` 方法里，我们可以看到下面一段代码：

```
var field = _.defaults($scope.scriptedField, {
  type: 'number',
  lang: 'expression'
});
try {
  if (createMode) {
    $scope.indexPattern.addScriptedField(field.name, field.script, field.type, field.lang);
  } else {
    $scope.indexPattern.save();
  }
```

没错，添加 scripted field 到 index pattern 的过程就是这里！我们可以看到，这里是有 `field.lang` 设定的。只不过其默认值是 `expression` 而已。

现在就去看 `plugins/settings/sections/indices/scripted_fields/index.html` 里关于 `$scope.scriptedField` 是怎么处理的。

```
<form name="scriptedFieldForm" ng-submit="submit()">
  <div class="form-group">
    <label>Name</label>
    <input required type="text" ng-model="scriptedField.name" class="form-control span12">
  </div>
  <div class="form-group">
    <textarea required class="scripted-field-script form-control span12" ng-model="scriptedField.script"></textarea>
  </div>
</form>
<div class="form-group">
  <button class="btn btn-primary" ng-click="goBack()">Cancel</button>
  <button class="btn btn-success" ng-click="submit()" ng-disabled="scriptedFieldForm.$invalid">
    Save Scripted Field
  </button>
</div>
```

没错。HTML 里只提供了 name 和 script 两个值的输入框！也就是说，kibana 4 只是不提供让你输入 groovy 到 `field.lang` 的文本框而已。

所以，如果你有随时定义 scripted field 的需求，又嫌弃每次 curl 直接修改 kibana_index 太麻烦还可能出错，那么你只需要稍微修改几处 kibana4 代码就够了：

1. `plugins/settings/sections/indices/scripted_fields/index.html` 里提供对 `scriptedField.lang` 和 `scriptedField.type` 的输入框；
2. `plugins/settings/sections/indices/_scripted_fields.js` 里给 `$scope.columns` 数组和 `addRow` 方法多加上 `field.lang` 字段的展示。
