# MySQL慢查询日志

MySQL 有多种日志可以记录，常见的有 error log、slow log、general log、bin log 等。其中 slow log 作为性能监控和优化的入手点，最为首要。本节即讨论如何用 logstash 处理 slow log。至于 general log，格式处理基本类似，不过由于 general 量级比 slow 大得多，推荐采用 packetbeat 协议解析的方式更高效的完成这项工作。相关内容阅读本书稍后章节。

MySQL slow log 的 logstash 处理配置示例如下：

```
input {
  file {
    type => "mysql-slow"
    path => "/var/log/mysql/mysql-slow.log"
    codec => multiline {
      pattern => "^# User@Host:"
      negate => true
      what => "previous"
    }
  }
}

filter {
  # drop sleep events
  grok {
    match => { "message" => "SELECT SLEEP" }
    add_tag => [ "sleep_drop" ]
    tag_on_failure => [] # prevent default _grokparsefailure tag on real records
  }
  if "sleep_drop" in [tags] {
    drop {}
  }
  grok {
    match => [ "message", "(?m)^# User@Host: %{USER:user}\[[^\]]+\] @ (?:(?<clienthost>\S*) )?\[(?:%{IP:clientip})?\]\s*# Query_time: %{NUMBER:query_time:float}\s+Lock_time: %{NUMBER:lock_time:float}\s+Rows_sent: %{NUMBER:rows_sent:int}\s+Rows_examined: %{NUMBER:rows_examined:int}\s*(?:use %{DATA:database};\s*)?SET timestamp=%{NUMBER:timestamp};\s*(?<query>(?<action>\w+)\s+.*)\n# Time:.*$" ]
  }
  date {
    match => [ "timestamp", "UNIX" ]
    remove_field => [ "timestamp" ]
  }
}
```

运行该配置，logstash 即可将多行的 MySQL slow log 处理成如下事件：

```
{
       "@timestamp" => "2014-03-04T19:59:06.000Z",
          "message" => "# User@Host: logstash[logstash] @ localhost [127.0.0.1]\n# Query_time: 5.310431  Lock_time: 0.029219 Rows_sent: 1  Rows_examined: 24575727\nSET timestamp=1393963146;\nselect count(*) from node join variable order by rand();\n# Time: 140304 19:59:14",
         "@version" => "1",
             "tags" => [
        [0] "multiline"
    ],
             "type" => "mysql-slow",
             "host" => "raochenlindeMacBook-Air.local",
             "path" => "/var/log/mysql/mysql-slow.log",
             "user" => "logstash",
       "clienthost" => "localhost",
         "clientip" => "127.0.0.1",
       "query_time" => 5.310431,
        "lock_time" => 0.029219,
        "rows_sent" => 1,
    "rows_examined" => 24575727,
            "query" => "select count(*) from node join variable order by rand();",
           "action" => "select"
}
```
