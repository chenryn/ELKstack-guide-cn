# 心跳检测

缺少内部队列状态的监控办法一直是 logstash 最为人诟病的一点。从 logstash-1.5.1 版开始，新发布了一个 logstash-input-heartbeat 插件，实现了一个最基本的队列堵塞状态监控。

配置示例如下：

```
input {
    heartbeat {
        message => "epoch"
        interval => 60
        type => "heartbeat"
        add_field => {
            "zbxkey" => "logstash.heartbeat",
            "zbxhost" => "logstash_hostname"
        }
    }
    tcp {
        port => 5160
    }
}
output {
    if [type] == "heartbeat" {
        file {
            path => "/data1/logstash-log/local6-5160-%{+YYYY.MM.dd}.log"
        }
        zabbix {
            zabbix_host => "zbxhost"
            zabbix_key => "zbxkey"
            zabbix_server_host => "zabbix.example.com"
            zabbix_value => "clock"
        }
    } else {
        elasticsearch { }
    }
}
```

示例中，同时将数据输出到本地文件和 zabbix server。注意，logstash-output-zabbix 并不是标准插件，需要额外安装：

```
bin/logstash-plugin install logstash-output-zabbix
```

文件中记录的就是 heartbeat 事件的内容，示例如下：

```
{"clock":1435191129,"host":"logtes004.mweibo.bx.sinanode.com","@version":"1","@timestamp":"2015-06-25T00:12:09.042Z","type":"heartbeat","zbxkey":"logstash.heartbeat","zbxhost":"logstash_hostname"}
```

可以通过文件最后的 clock 和 @timestamp 内容，对比当前时间，来判断 logstash 内部队列是否有堵塞。
