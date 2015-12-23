# ossec

*本节作者：林鹏*

## 配置OSSEC SYSLOG 输出 （所有agent）

1. 编辑ossec.conf 文件（默认为/var/ossec/etc/ossec.conf）
2. 在ossec.conf中添加下列内容（10.0.0.1 为 接收syslog 的服务器）

```
<syslog_output>
   <server>10.0.0.1</server>
   <port>9000</port>
   <format>default</format>
</syslog_output>
```

3. 开启OSSEC允许syslog输出功能

```
/var/ossec/bin/ossec-control enable client-syslog
```

4. 重启 OSSEC服务
```
/var/ossec/bin/ossec-control start
```

## 配置LOGSTASH

1. 在logstash 中 配置文件中增加(或新建)如下内容：（假设10.0.0.1 为ES服务器,假设文件名为logstash-ossec.conf ）

```
input {
    udp {
        port => 9000
        type => "syslog"
    }
}
filter {
    if [type] == "syslog" {
        grok {
            match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_host} %{DATA:syslog_program}: Alert Level: %{BASE10NUM:Alert_Level}; Rule: %{BASE10NUM:Rule} - %{GREEDYDATA:Description}; Location: %{GREEDYDATA:Details}" }
            add_field => [ "ossec_server", "%{host}" ]
        }
        mutate {
            remove_field => [ "syslog_hostname", "syslog_message", "syslog_pid", "message", "@version", "type", "host" ]
        }
    }
}
output {
    elasticsearch_http {
        host => "10.0.0.1"
    }
}
```

## 推荐 Kibana dashboard

社区已经有人根据 ossec 的常见需求，制作有 dashboard 可以直接从 Kibana3 页面加载使用。

![](https://raw.githubusercontent.com/magenx/Logstash/master/Kibana%203%20%20%20OSSEC%20MONITORING%20DASHBOARD.png)

dashboard 的 JSON 文件见：<https://github.com/magenx/Logstash/raw/master/kibana/kibana_dashboard.json>

加载方式，请阅读本书稍后 Kibana 章节相关内容。
