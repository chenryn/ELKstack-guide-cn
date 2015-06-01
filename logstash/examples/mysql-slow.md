# MySQL慢查询日志

```
input {
  file {
    type => "mysql-slow"
    path => "/var/log/mysql/mysql-slow.log"

    # Key breaking the log up on the # User@Host line, this will mean
    # sometimes a # Time line from the next entry will be incorrectly
    # included but since that isn't consistently present it can't be
    # keyed off of
    #
    # Due to the way the multiline codec works, previous must be used
    # to collect everything which isn't the User line up. Since
    # queries can be multiline the User line can't be pushed forward
    # as it would only collect the first line of the actual query
    # data.
    #
    # logstash will always be one slow query behind because it needs
    # the User line to trigger that it is done with the previous entry.
    # A periodic "SELECT SLEEP(1);" where 1 is above the slow query
    # threshold can be used to "flush" these events through at the
    # expense of having spurious log entries (see the drop filter)
    codec => multiline {
      pattern => "^# User@Host:"
      negate => true
      what => previous
    }
  }
}

filter {
  # drop sleep events
  grok {
    match => { "message" => "SELECT SLEEP" }
    drop_if_match => true # appears ineffective, thus tag and conditional
    add_tag => [ "sleep_drop" ]
    tag_on_failure => [] # prevent default _grokparsefailure tag on real records
  }
  if "sleep_drop" in [tags] {
    drop {}
  }
  # Capture user, optional host and optional ip fields
  # sample log file lines:
  # User@Host: logstash[logstash] @ localhost [127.0.0.1]
  # User@Host: logstash[logstash] @  [127.0.0.1]
  grok {
    match => [ "message", "^# User@Host: %{USER:user}(?:\[[^\]]+\])?\s+@\s+%{HOST:host}?\s+\[%{IP:ip}?\]" ]
  }
  # Capture query time, lock time, rows returned and rows examined
  # sample log file lines:
  # Query_time: 102.413328  Lock_time: 0.000167 Rows_sent: 0  Rows_examined: 1970
  # Query_time: 1.113464  Lock_time: 0.000128 Rows_sent: 1  Rows_examined: 0
  grok {
    match => [ "message", "^# Query_time: %{NUMBER:duration:float}\s+Lock_time: %{NUMBER:lock_wait:float} Rows_sent: %{NUMBER:results:int} \s*Rows_examined: %{NUMBER:scanned:int}"]
  }
  # Capture the time the query happened
  grok {
    match => [ "message", "^SET timestamp=%{NUMBER:timestamp};" ]
  }
  # Extract the time based on the time of the query and
  # not the time the item got logged
  date {
    match => [ "timestamp", "UNIX" ]
  }
  # Drop the captured timestamp field since it has been moved to the
  # time of the event
  mutate {
    remove_field => "timestamp"
  }
}
```




```
{
      "@timestamp" => "2014-03-04T19:59:06.000Z",
         "message" => "# User@Host: logstash[logstash] @ localhost [127.0.0.1]\n# Query_time: 5.310431  Lock_time: 0.029219 Rows_sent: 1  Rows_examined: 24575727\nSET timestamp=1393963146;\nselect count(*) from node join variable order by rand();\n# Time: 140304 19:59:14",
        "@version" => "1",
            "tags" => [
                [0] "multiline"
            ],
            "type" => "mysql-slow",
            "host" => [
                [0] "cj",
                [1] "localhost"
            ],
            "path" => "/var/log/mysql/mysql-slow.log",
            "user" => "root",
              "ip" => "127.0.0.1",
        "duration" => 5.310431,
       "lock_wait" => 0.029219,
         "results" => 1,
         "scanned" => 24575727,
    "date_matched" => "yes"
}
```
