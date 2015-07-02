# elapsed

```
filter {
  grok {
    match => ["message", "%{TIMESTAMP_ISO8601} START id: (?<task_id>.*)"]
    add_tag => [ "taskStarted" ]
  }
  grok {
    match => ["message", "%{TIMESTAMP_ISO8601} END id: (?<task_id>.*)"]
    add_tag => [ "taskTerminated"]
  }
  elapsed {
    start_tag => "taskStarted"
    end_tag => "taskTerminated"
    unique_id_field => "task_id"
  }
}
```
