# ElastAlert

ElastAlert 是 Yelp 公司开源的一套用 Python2.6 写的报警框架。属于后来 Elastic.co 公司出品的 Watcher 同类产品。官网地址见：<http://elastalert.readthedocs.org/>。

## 安装

比官网文档说的步骤稍微复杂一点，因为其中 mock 模块安装时依赖的 setuptools 要求版本在 0.17 以上，CentOS6 默认的不够，需要通过 yum 命令升级，当前可以升级到的是 0.18 版。

```
# yum install python-setuptools
# git clone https://github.com/Yelp/elastalert.git
# cd elastalert
# python setup.py install
# cp config.yaml.example config.yaml
```

安装完成后会自带三个命令：

* elastalert-create-index
  ElastAlert 会把执行记录存放到一个 ES 索引中，该命令就是用来创建这个索引的，默认情况下，索引名叫 `elastalert_status`。其中有 4 个 _type，都有自己的 `@timestamp` 字段，所以同样也可以用 kibana 来查看这个索引的日志记录情况。
* elastalert-rule-from-kibana
  从 Kibana3 已保存的仪表盘中读取 Filtering 设置，帮助生成 `config.yaml` 里的配置。不过注意，它只会读取 filtering，不包括 queries。
* elastalert-test-rule
  测试自定义配置中的 rule 设置。

最后，运行命令:

```
# python -m elastalert.elastalert --config ./config.yaml
```

或者单独执行 `rules_folder` 里的某个 rule：

```
# python -m elastalert.elastalert --config ./config.yaml --rule ./examele_rules/one_rule.yaml
```

## 配置结构

和 Watcher 类似(或者说也只有这种方式)，ElastAlert 配置结构也分几个部分，但是它有自己的命名。

### query 部分

除了有关 ES 服务器的配置以外，主要包括：

* `run_every` 配置，用来设置定时向 ES 发请求，默认 5 分钟。
* `buffer_time` 配置，用来设置请求里时间字段的范围，默认 45 分钟。
* `rules_folder` 配置，用来加载下一阶段的 rule 设置，默认是 `example_rules`。
* `timestamp_field` 配置，设置 `buffer_time` 时针对哪个字段，默认是 `@timestamp`。
* `timestamp_type` 配置，设置 `timestamp_field` 的时间类型，ElastAlert 内部也需要转换成时间对象，默认是 `ISO8601`，也可以是 `UNIX`。

### rule 部分

rule 设置各自独立以文件方式存储在 `rules_folder` 设置的目录里。其中可以定义下面这些参数：

* `name` 配置，每个 rule 需要有自己独立的 name，一旦重复，进程将无法启动。
* `type` 配置，选择某一种数据验证方式。
* `index` 配置，从某类索引里读取数据，目前已经支持Ymd格式，需要先设置use_strftime_index: true，然后匹配索引，配置形如：index: logstash-es-test-%Y.%m.%d，表示匹配logstash-es-test名称开头，以年月日作为索引后缀的index。
* `filter` 配置，设置向 ES 请求的过滤条件。
* `timeframe` 配置，累积触发报警的时长。
* `alert` 配置，设置触发报警时执行哪些报警手段。

不同的 type 还有自己独特的配置选项。目前 ElastAlert 有以下几种自带 ruletype：

* `any`: 只要有匹配就报警；
* `blacklist`: `compare_key` 字段的内容匹配上 `blacklist` 数组里任意内容；
* `whitelist`: `compare_key` 字段的内容一个都没能匹配上 `whitelist` 数组里内容；
* `change`: 在相同 `query_key` 条件下，`compare_key` 字段的内容，在 `timeframe` 范围内发送变化；
* `frequency`: 在相同 `query_key` 条件下，`timeframe` 范围内有 `num_events` 个被过滤出来的异常；
* `spike`: 在相同 `query_key` 条件下，前后两个 `timeframe` 范围内数据量相差比例超过 `spike_height`。其中可以通过 `spike_type` 设置具体涨跌方向是`up`, `down`, `both`。还可以通过`threshold_ref` 设置要求上一个周期数据量的下限，`threshold_cur` 设置要求当前周期数据量的下限，如果数据量不到下限，也不触发；
* `flatline`: `timeframe` 范围内，数据量小于 `threshold` 阈值；
* `new_term`: `fields` 字段新出现之前 `terms_window_size`(默认 30 天) 范围内最多的 `terms_size`(默认 50) 个结果以外的数据；
* `cardinality`: 在相同 `query_key` 条件下，`timeframe` 范围内 `cardinality_field` 的值超过 `max_cardinality` 或者低于 `min_cardinality`。

### alert 部分

`alert` 配置是一个数组，目前支持 command, email，jira，opsgenie，sns，hipchat，slack 等方式。

* command

command 最灵活也最简单。默认会采用 `%(fieldname)s` 格式：

```
command: ["/bin/send_alert", "--username", "%(username)s", "--time", "%(key_as_string)s"]
```

如果要用的比较多，可以开启 `pipe_match_json` 参数，会把整个过滤到的内容，以一整个 JSON 字符串的方式管道输入指定脚本。

* email

email 方式采用 SMTP 协议，所以有一系列 `smtp_*` 配置，然后加上 `email` 参数提供收件人地址数组。

特殊的是，email 和 jira 两种方式，ElastAlert 提供了一些内容格式化模板：

比如可以这样控制邮件标题：

```
alert_subject: "Issue {0} occurred at {1}"
alert_subject_args:
  - issue.name
  - "@timestamp"
```

而默认的邮件内容模板是：

```
body = rule_name
       [alert_text]
       ruletype_text
       {top_counts}
       {field_values}
```

这些内容同样可以通过 `alert_text`(及对应 `alert_text_args`)等来灵活修改。

此外，alert 还有一系列控制报警风暴的选项，从属于 rule：

* `aggregation`：设置一个时长，则该时长内所有报警最终合在一起发一次；
* `realert`：设置一个时长，则该时长内，相同 `query_key` 的报警只发一个；
* `exponential_realert`：设置一个时长，必须大于 `realert` 设置。则在 `realert` 到 `exponential_realert` 之间，每次报警后，realert 自动翻倍。

### enhancements 部分

`match_enhancements` 配置，设置一个数组，在报警内容发送到 alert 之前修改具体数据。ElastAlert 默认不提供具体的 enhancements 实现，需要自己扩展。

不过，作为通用方式，ElastAlert 提供几个便捷选项，把 Kibana 地址加入报警：

* `generate_kibana_link`: 自动生成一个 Kibana3 的临时仪表盘附在报警内容上。
* `use_kibana_dashboard`: 采用现成的 Kibana3 仪表盘附在报警内容上。
* `use_kibana4_dashboard`: 采用现成的 Kibana4 仪表盘附在报警内容上。

## 扩展

### rule

创建一个自己的 rule，是以 Python 模块的形式存在的，所以首先创建目录：

```
# mkdir rule_modules
# cd rule_modules
# touch __init__.py example_rule.py
```

`example_rule.py` 的内容如下：

```
import dateutil.parser
from elastalert.util import ts_to_dt
from elastalert.ruletypes import RuleType

class AwesomeNewRule(RuleType):
    # 用来指定本 rule 对应的配置文件中必要的参数项
    required_options = set(['time_start', 'time_end', 'usernames'])
    # 每次运行获取的数据以时间排序数据传递给 add_data 函数
    def add_data(self, data):
        for document in data:
            # 配置文件中的设置可以通过 self.rule[] 获取
            if document['username'] in self.rule['usernames']:
                login_time = document['@timestamp'].time()
                time_start = dateutil.parser.parse(self.rule['time_start']).time()
                time_end = dateutil.parser.parse(self.rule['time_end']).time()
                if login_time > time_start and login_time < time_end:
                    # 最终过滤结果，使用 self.add_match 添加
                    self.add_match(document)

    # alert_text 中使用的文本
    def get_match_str(self, match):
        return "%s logged in between %s and %s" % (match['username'],
                                                   self.rule['time_start'],
                                                   self.rule['time_end'])
    def garbage_collect(self, timestamp):
        pass
```

配置中，指定

```
type: rule_modules.example_rule.AwesomeRule
time_start: "20:00"
time_end: "24:00"
usernames:
  - "admin"
  - "userXYZ"
  - "foobaz"
```

即可使用。

### alerter

alerter 也是以 Python 模块的形式存在的，所以还是要创建目录(如果之前二次开发 rule 已经创建过可以跳过)：

```
# mkdir rule_modules
# cd rule_modules
# touch __init__.py example_alert.py
```

`example_alert.py` 的内容如下：

```
from elastalert.alerts import Alerter, basic_match_string

class AwesomeNewAlerter(Alerter):
    required_options = set(['output_file_path'])
    def alert(self, matches):
        for match in matches:
            with open(self.rule['output_file_path'], "a") as output_file:
                # basic_match_string 函数用来转换异常数据成默认格式的字符串
                match_string = basic_match_string(self.rule, match)
                output_file.write(match_string)
    # 报警发出后，ElastAlert 会调用该函数的结果写入 ES 索引的 alert_info 字段内
    def get_info(self):
        return {'type': 'Awesome Alerter',
                'output_file': self.rule['output_file_path']}
```

配置中，指定

```
alert: "rule_modules.example_alert.AwesomeNewAlerter"
output_file_path: "/tmp/alerts.log"
```

即可使用。

### enhancement

enhancement 也是以 Python 模块的形式存在的，所以还是要创建目录(如果之前二次开发 rule 或 alert 已经创建过可以跳过)：

```
# mkdir rule_modules
# cd rule_modules
# touch __init__.py example_enhancement.py
```

`example_enhancement.py` 的内容如下：

```
from elastalert.enhancements import BaseEnhancement
class MyEnhancement(BaseEnhancement):
    def process(self, match):
        if 'domain' in match:
            url = "http://who.is/whois/%s" % (match['domain'])
            match['domain_whois_link'] = url
```

在需要的 rule 配置文件中添加如下内容即可启用：

```
match_enhancements:
  - "rule_modules.example_enhancement.MyEnhancement"
```

因为 `match_enhancements` 是个数组，也就是说，如果数组有多个 enhancement，会依次执行，完全完成后，才传递给 alert。
