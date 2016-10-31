要使用 Kibana，你就得告诉它你想要探索的 Elasticsearch 索引是那些，这就要配置一个或者更多的索引模式。此外，你还可以：

* 创建脚本化字段，这个字段可以实时从你的数据中计算出来。你可以浏览这种字段，并且在它基础上做可视化，但是不能搜索这种字段。
* 设置高级选项，比如表格里显示多少行，常用字段显示多少个。修改高级选项的时候要千万小心，因为一个设置很可能跟另一个设置是不兼容的。
* 为生产环境配置 Kibana。

## 创建一个连接到 Elasticsearch 的索引模式

一个索引模式定义了一个或者多个你打算探索的 Elasticsearch 索引。Kibana 会查找匹配指定模式的索引名。模式中的通配符(\*)匹配零到多个字符。比如，模式 `myindex-*` 匹配所有名字以 `myindex-` 开头的索引，比如 `myindex-1` 和 `myindex-2`。

如果你用了事件时间来创建索引名(比如说，如果你是用 Logstash 往 Elasticsearch 里写数据)，索引模式里也可以匹配一个日期格式。在这种情况下，模式的静态文本部分必须用中括号包含起来，日期格式能用的字符，参见表 1 "日期格式码"。

比如，`[logstash-]YYYY.MM.DD` 匹配所有名字以 `logstash-` 为前缀，后面跟上 `YYYY.MM.DD` 格式时间戳的索引，比如 `logstash-2015.01.31` 和 `logstash-2015-02-01`。

索引模式也可以简单的设置为一个单独的索引名字。

要创建一个连接到 Elasticsearch 的索引模式：

1. 切换到 `Settings > Indices` 标签页。
2. 指定一个能匹配你的 Elasticsearch 索引名的索引模式。默认的，Kibana 会假设你是要处理 Logstash 导入的数据。

> 当你在顶层标签页之间切换的时候，Kibana 会记住你之前停留的位置。比如，如果你在 Settings 标签页查看了一个索引模式，然后切换到 Discover 标签，再切换回 Settings 标签，Kibana 还会显示上次你查看的索引模式。要看到创建模式的表单，需要从索引模式列表里点击 `Add` 按钮。

3. 如果你索引有时间戳字段打算用来做基于事件的对比，勾选 `Index contains time-based events` 然后选择包含了时间戳的索引字段。Kibana 会读取索引映射，列出所有包含了时间戳的字段供选择。
4. 如果新索引是周期性生成，名字里有时间戳的，勾选 `Use event times to create index names` 和 `Index pattern interval` 选项。这会让 Kibana 只搜索哪些包含了你指定的时间范围内的数据的索引。当你使用 Logstash 往 Elasticsearch 写数据的时候非常有用。
5. 点击 `Create` 添加索引模式。
6. 要设置新模式作为你查看 Discover 页是的默认模式，点击 `favorite` 按钮。

表 1. 日期格式码

|格式  | 描述                        |
|------|-----------------------------|
|M     | Month - cardinal: 1 2 3 … 12|
|Mo    | Month - ordinal: 1st 2nd 3rd … 12th|
|MM    | Month - two digit: 01 02 03 … 12|
|MMM   | Month - abbreviation: Jan Feb Mar … Dec|
|MMMM  | Month - full: January February March … December|
|Q     | Quarter: 1 2 3 4|
|D     | Day of Month - cardinal: 1 2 3 … 31|
|Do    | Day of Month - ordinal: 1st 2nd 3rd … 31st|
|DD    | Day of Month - two digit: 01 02 03 … 31|
|DDD   | Day of Year - cardinal: 1 2 3 … 365|
|DDDo  | Day of Year - ordinal: 1st 2nd 3rd … 365th|
|DDDD  | Day of Year - three digit: 001 002 … 364 365|
|d     | Day of Week - cardinal: 0 1 3 … 6|
|do    | Day of Week - ordinal: 0th 1st 2nd … 6th|
|dd    | Day of Week - 2-letter abbreviation: Su Mo Tu … Sa|
|ddd   | Day of Week - 3-letter abbreviation: Sun Mon Tue … Sat|
|dddd  | Day of Week - full: Sunday Monday Tuesday … Saturday|
|e     | Day of Week (locale): 0 1 2 … 6|
|E     | Day of Week (ISO): 1 2 3 … 7|
|w     | Week of Year - cardinal (locale): 1 2 3 … 53|
|wo    | Week of Year - ordinal (locale): 1st 2nd 3rd … 53rd|
|ww    | Week of Year - 2-digit (locale): 01 02 03 … 53|
|W     | Week of Year - cardinal (ISO): 1 2 3 … 53|
|Wo    | Week of Year - ordinal (ISO): 1st 2nd 3rd … 53rd|
|WW    | Week of Year - two-digit (ISO): 01 02 03 … 53|
|YY    | Year - two digit: 70 71 72 … 30|
|YYYY  | Year - four digit: 1970 1971 1972 … 2030|
|gg    | Week Year - two digit (locale): 70 71 72 … 30|
|gggg  | Week Year - four digit (locale): 1970 1971 1972 … 2030|
|GG    | Week Year - two digit (ISO): 70 71 72 … 30|
|GGGG  | Week Year - four digit (ISO): 1970 1971 1972 … 2030|
|A     | AM/PM: AM PM|
|a     | am/pm: am pm|
|H     | Hour: 0 1 2 … 23|
|HH    | Hour - two digit: 00 01 02 … 23|
|h     | Hour - 12-hour clock: 1 2 3 … 12|
|hh    | Hour - 12-hour clock, 2 digit: 01 02 03 … 12|
|m     | Minute: 0 1 2 … 59|
|mm    | Minute - two-digit: 00 01 02 … 59|
|s     | Second: 0 1 2 … 59|
|ss    | Second - two-digit: 00 01 02 … 59|
|S     | Fractional Second - 10ths: 0 1 2 … 9|
|SS    | Fractional Second - 100ths: 0 1 … 98 99|
|SSS   | Fractional Seconds - 1000ths: 0 1 … 998 999|
|Z     | Timezone - zero UTC offset (hh:mm format): -07:00 -06:00 -05:00 .. +07:00|
|ZZ    | Timezone - zero UTC offset (hhmm format): -0700 -0600 -0500 … +0700|
|X     | Unix Timestamp: 1360013296|
|x     | Unix Millisecond Timestamp: 1360013296123|

## 设置默认索引模式

默认索引模式会在你查看 **Discover** 标签的时候自动加载。Kibana 会在 **Settings > Indices** 标签页的索引模式列表里，给默认模式左边显示一个星号。你创建的第一个模式会自动被设置为默认模式。

要设置一个另外的模式为默认索引模式：

1. 进入 `Settings > Indices` 标签页。
2. 在索引模式列表里选择你打算设置为默认值的模式。
3. 点击模式的 `Favorite` 标签。

> 你也可以在 **Advanced > Settings** 里设置默认索引模式。

## 重加载索引的字段列表

当你添加了一个索引映射，Kibana 自动扫描匹配模式的索引以显示索引字段。你可以重加载索引字段列表，以显示新添加的字段。

重加载索引字段列表，也会重设 Kibana 的常用字段计数器。这个计数器是跟踪你在 Kibana 里常用字段，然后来排序字段列表的。

要重加载索引的字段列表：

1. 进入 `Settings > Indices` 标签页。
2. 在索引模式列表里选择一个索引模式。
3. 点击模式的 `Reload` 按钮。

## 删除一个索引模式

要删除一个索引模式：

1. 进入 `Settings > Indices` 标签页。
2. 在索引模式列表里选择你打算删除的模式。
3. 点击模式的 `Delete` 按钮。
4. 确认你是想要删除这个索引模式。

## 创建一个脚本化字段

脚本化字段从你的 Elasticsearch 索引数据中即时计算得来。在 **Discover** 标签页，脚本化字段数据会作为文档数据的一部分显示，而且你还可以在可视化里使用脚本化字段。(脚本化字段的值是在请求的时候计算的，所以它们没有被索引，不能搜索到)

> 即时计算脚本化字段非常消耗资源，会直接影响到 Kibana 的性能。而且记住，Elasticsearch 里没有内置对脚本化字段的验证功能。如果你的脚本有 bug，你会在查看动态生成的数据时看到 exception。

脚本化字段使用 Lucene 表达式语法。更多细节，请阅读 [Lucene Expressions Scripts](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-scripting.html#_lucene_expressions_scripts)。

你可以在表达式里引用任意单个数值类型字段，比如：

    doc['field_name'].value

要创建一个脚本化字段：

1. 进入 `Settings > Indices`
2. 选择你打算添加脚本化字段的索引模式。
3. 进入模式的 `Scripted Fields` 标签。
4. 点击 `Add Scripted Field`。
5. 输入脚本化字段的名字。
6. 输入用来即时计算数据的表达式。
7. 点击 `Save Scripted Field`.

有关 Elasticsearch 的脚本化字段的更多细节，阅读 [Scripting](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-scripting.html)。

## 更新一个脚本化字段

要更新一个脚本化字段：

1. 进入 `Settings > Indices`。
2. 点击你要更新的脚本化字段的 `Edit` 按钮。
3. 完成变更后点击 `Save Scripted Field` 升级。

> 注意 Elasticsearch 里没有内置对脚本化字段的验证功能。如果你的脚本有 bug，你会在查看动态生成的数据时看到 exception。

## 删除一个脚本化字段

要删除一个脚本化字段：

1. 进入 `Settings > Indices`。
2. 点击你要删除的脚本化字段的 `Delete` 按钮。
3. 确认你确实想删除它。

## 设置高级参数

高级参数页允许你直接编辑那些控制着 Kibana 应用行为的设置。比如，你可以修改显示日期的格式，修改默认的索引模式，设置十进制数值的显示精度。

> 修改高级参数可能带来意想不到的后果。如果你不确定自己在做什么，最好离开这个设置页面。

要设置高级参数：

1. 进入 `Settings > Advanced`。
2. 点击你要修改的选项的 `Edit` 按钮。
3. 给这个选项输入一个新的值。
4. 点击 `Save` 按钮。

# 管理已保存的搜索，可视化和仪表板

你可以从 **Settings > Objects** 查看，编辑，和删除已保存的搜索，可视化和仪表板。

查看一个已保存的对象会显示在 **Discover**, **Visualize** 或 **Dashboard** 页里已选择的项目。要查看一个已保存对象：

1. 进入 `Settings > Objects`。
2. 选择你想查看的对象。
3. 点击 `View` 按钮。

编辑一个已保存对象让你可以直接修改对象定义。你可以修改对象的名字，添加一段说明，以及修改定义这个对象的属性的 JSON。

如果你尝试访问一个对象，而它关联的索引已经被删除了，Kibana 会显示这个对象的编辑(Edit Object)页。你可以：

* 重建索引这样就可以继续用这个对象。
* 删除对象，然后用另一个索引重建对象。
* 在对象的 `kibanaSavedObjectMeta.searchSourceJSON` 里修改引用的索引名，指向一个还存在的索引模式。这个在你的索引被重命名了的情况下非常有用。

> 对象属性没有验证机制。提交一个无效的变更会导致对象不可用。通常来说，你还是应该用 Discover, Visualize 或 Dashboard 页面来创建新对象而不是直接编辑已存在的对象。

要编辑一个已保存的对象：

1. 进入 `Settings > Objects`。
2. 选择你想编辑的对象。
3. 点击 `Edit` 按钮。
4. 修改对象定义。
5. 点击 `Save Object` 按钮。

要删除一个已保存的对象：

1. 进入 `Settings > Objects`。
2. 选择你想删除的对象。
3. 点击 `Delete` 按钮。
4. 确认你确实想删除这个对象。

# 设置 kibana 服务器属性

Kibana 服务器在启动的时候会从 `kibana.yml` 文件读取属性设置。默认设置是运行在 `localhost:5601`。要变更主机或端口，或者连接远端主机上的 Elasticsearch，你都需要更新你的 `kibana.yml` 文件。你还可以开启 SSL 或者设置其他一系列选项：

表 2. Kibana 服务器属性

|属性                         | 描述                                                           |
|-----------------------------|----------------------------------------------------------------|
|port                         | Kibana 服务器运行的端口。默认：`port: 5601`。                  |
|host                         | Kibana 服务器监听的地址。默认：`host: "0.0.0.0"`。             |
|elasticsearch_url            | 你想请求的索引存在哪个 Elasticsearch 实例上。默认：`elasticsearch_url: "http://localhost:9200"`。|
|elasticsearch_preserve_host  | 默认的，浏览器请求中的主机名即作为 Kibana 发送给 Elasticsearch 时请求的主机名。如果你设置这个参数为 `false`, Kibana 会改用 `elasticsearch_url` 里的主机名。你应该不用担心这个设置 —— 直接用默认即可。默认：`elasticsearch_preserve_host: true`。|
|kibana_index                 | 保存搜索，可视化，仪表板信息的索引的名字。默认：`kibana_index: .kibana`。|
|default_app_id               | 进入 Kibana 是默认显示的页面。可以为 `discover`, `visualize`, `dashboard` 或 `settings`。默认：`default_app_id: "discover"`。|
|request_timeout              | 等待 Kibana 后端或 Elasticsearch 的响应的超时时间，单位毫秒。默认：`request_timeout: 500000`。|
|shard_timeout                | Elasticsearch 等待分片响应的超时时间。设置为 0 表示关闭超时控制。默认：`shard_timeout: 0`。|
|verify_ssl                   | 定义是否验证 Elasticsearch SSL 证书。设置为 false 关闭 SSL 认证。默认：`verify_ssl: true`。|
|ca                           | 你的 Elasticsearch 实例的 CA 证书的路径。如果你是自己签的证书，必须指定这个参数，证书才能被认证。否则，你需要关闭 `verify_ssl`。默认：none。|
|ssl_key_file                 | Kibana 服务器的密钥文件路径。设置用来加密浏览器和 Kibana 之间的通信。默认：none。|
|ssl_cert_file                | Kibana 服务器的证书文件路径。设置用来加密浏览器和 Kibana 之间的通信。默认：none。|
|pid_file                     | 你想用来存进程 ID 文件的位置。如果没有指定，PID 文件存在 `/var/run/kibana.pid`。默认：none。|
