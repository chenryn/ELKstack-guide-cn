# fluentd

Fluentd 是另一个 Ruby 语言编写的日志收集系统。和 Logstash 不同的是，Fluentd 是基于 MRI 实现的，并不是利用多线程，而是利用事件驱动。

Fluentd 的开发和使用者，大多集中在日本。

## 配置示例

Fluentd 受 Scribe 影响颇深，包括节点间传输采用磁盘 buffer 来保证数据不丢失等的设计，也包括配置语法。下面是一段配置示例：

```
<source>
  type tail
  read_from_head true
  path /var/lib/docker/containers/*/*-json.log
  pos_file /var/log/fluentd-docker.pos
  time_format %Y-%m-%dT%H:%M:%S
  tag docker.*
  format json
</source>
# Using filter to add container IDs to each event
<filter docker.var.lib.docker.containers.*.*.log>
  type record_transformer
  <record>
    container_id ${tag_parts[5]}
  </record>
</filter>

<match docker.var.lib.docker.containers.*.*.log>
  type copy
  <store>
    # for debug (see /var/log/td-agent.log)
    type stdout
  </store>
  <store>
    type elasticsearch
    logstash_format true
    host "#{ENV['ES_PORT_9200_TCP_ADDR']}" # dynamically configured to use Docker's link feature
    port 9200
    flush_interval 5s
  </store>
</match>
```

注意，虽然示例中演示的是 tail 方式。Fluentd 对应用日志，并不推荐如此读取。FLuentd 为各种编程语言提供了客户端库，应用可以直接加载日志库发送日志。下面是一个 PHP 应用的示例：

```
<?php
require_once __DIR__.'/src/Fluent/Autoloader.php';
use Fluent\Logger\FluentLogger;
Fluent\Autoloader::register();
$logger = new FluentLogger("unix:///var/run/td-agent/td-agent.sock");
$logger->post("fluentd.test.follow", array("from"=>"userA", "to"=>"userB"));
```

Fluentd 使用如下配置接收即可：

```
<source>
  type unix
  path /var/run/td-agent/td-agent.sock
</source>
<match fluentd.test.**>
  type forward
  send_timeout 60s
  recover_wait 10s
  heartbeat_interval 1s
  phi_threshold 16
  hard_timeout 60s
  <server>
    name myserver1
    host 192.168.1.3
    port 24224
    weight 60
  </server>
  <server>
    name myserver2
    host 192.168.1.4
    port 24224
    weight 60
  </server>
  <secondary>
    type file
    path /var/log/fluent/forward-failed
  </secondary>
</match>
```

## fluentd 插件

作为用动态语言编写的软件，fluentd 也拥有大量插件。每个插件都以 RubyGem 形式独立存在。事实上，logstash 在这方面就是学习 fluentd 的。安装方式如下：

```
/usr/sbin/td-agent-gem install fluent-plugin-elasticsearch fluent-plugin-grok_parser
```

fluentd 插件列表，见：<http://www.fluentd.org/plugins>。
