# winlogbeat

winlogbeat 通过标准的 windows API 获取 windows 系统日志，常见的有 application，hardware，security 和 system 四类。winlogbeat 示例配置如下：

```
winlogbeat.event_logs:
    - name: Application
      provider:
          - Application Error
          - Application Hang
          - Windows Error Reporting
          - EMET
    - name: Security
      level: critical, error, warning
      event_id: 4624, 4625, 4700-4800, -4735
    - name: System
      ignore_older: 168h
    - name: Microsoft-Windows-Windows Defender/Operational
      include_xml: true

output.elasticsearch:
    hosts:
        - localhost:9200
    pipeline: "windows-pipeline-id"

logging.to_files: true
    logging.files:
        path: C:/ProgramData/winlogbeat/Logs
        logging.level: info
```

和其他 beat 一样，这里示例的配置不都是必填项。事实上只有 `event_logs.name` 是必须的。而 winlogbeat 的输出字段中，除了 beats 家族的通用内容外，还包括一下特有字段：


* activity\_id
* computer\_name：如果运行在 Windows 事件转发模式，这个值会和 beat.hostname 不一样。
* event\_data
* event\_id
* keywords
* log\_name
* level：可选值包括 Success, Information, Warning, Error, Audit Success, and Audit Failure.
* message
* message\_error
* record\_number
* related\_activity\_id
* opcode
* provider\_guid
* process\_id
* source\_name
* task
* thread\_id
* user\_data
* user.identifier
* user.name
* user.domain
* user.type
* version
* xml
