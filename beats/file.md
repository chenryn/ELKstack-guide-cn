# filebeat

filebeat 是基于原先 logstash-forwarder 的源码改造出来的。换句话说：filebeat 就是新版的 logstash-forwarder，也会是 Elastic Stack 在 shipper 端的第一选择。

![](https://www.elastic.co/guide/en/beats/filebeat/current/images/filebeat.png)

## 安装部署

* deb:
```
curl -L -O https://download.elastic.co/beats/filebeat/filebeat_5.0.0_amd64.deb
sudo dpkg -i filebeat_5.0.0_amd64.deb
```
* rpm:
```
curl -L -O https://download.elastic.co/beats/filebeat/filebeat-5.0.0-x86_64.rpm
sudo rpm -vi filebeat-5.0.0-x86_64.rpm
```
* mac:
```
curl -L -O https://download.elastic.co/beats/filebeat/filebeat-5.0.0-darwin.tgz
tar xzvf filebeat-5.0.0-darwin.tgz
```
* win:

1. 下载 https://download.elastic.co/beats/filebeat/filebeat-5.0.0-windows.zip
2. 解压到 C:\Program Files
3. 重命名 filebeat-5.0.0-windows 目录为 Filebeat
4. 右键点击 PowerSHell 图标，选择『以管理员身份运行』
5. 运行下列命令，将 Filebeat 安装成 windows 服务：
```
PS > cd 'C:\Program Files\Filebeat'
PS C:\Program Files\Filebeat> .\install-service-filebeat.ps1
```

*注意*

可能需要额外授予执行权限。命令为：`PowerShell.exe -ExecutionPolicy RemoteSigned -File .\install-service-filebeat.ps1`.

## filebeat 配置

所有的 beats 组件在 output 方面的配置都是一致的，之前章节已经介绍过。这里只介绍 filebeat 在 input 段的配置，如下：

```yaml
filebeat:
    spool_size: 1024                                    # 最大可以攒够 1024 条数据一起发送出去
    idle_timeout: "5s"                                  # 否则每 5 秒钟也得发送一次
    registry_file: ".filebeat"                          # 文件读取位置记录文件，会放在当前工作目录下。所以如果你换一个工作目录执行 filebeat 会导致重复传输！
    config_dir: "path/to/configs/contains/many/yaml"    # 如果配置过长，可以通过目录加载方式拆分配置
    prospectors:                                        # 有相同配置参数的可以归类为一个 prospector
        -
            fields:
                ownfield: "mac"                         # 类似 logstash 的 add_fields
            paths:
                - /var/log/system.log                   # 指明读取文件的位置
                - /var/log/wifi.log
            include_lines: ["^ERR", "^WARN"]            # 只发送包含这些字样的日志
            exclude_lines: ["^OK"]                      # 不发送包含这些字样的日志
        -
            document_type: "apache"                     # 定义写入 ES 时的 _type 值
            ignore_older: "24h"                         # 超过 24 小时没更新内容的文件不再监听。在 windows 上另外有一个配置叫 force_close_files，只要文件名一变化立刻关闭文件句柄，保证文件可以被删除，缺陷是可能会有日志还没读完
            scan_frequency: "10s"                       # 每 10 秒钟扫描一次目录，更新通配符匹配上的文件列表
            tail_files: false                           # 是否从文件末尾开始读取
            harvester_buffer_size: 16384                # 实际读取文件时，每次读取 16384 字节
            backoff: "1s"                               # 每 1 秒检测一次文件是否有新的一行内容需要读取
            paths:
                - "/var/log/apache/*"                   # 可以使用通配符
            exclude_files: ["/var/log/apache/error.log"]
        -
            input_type: "stdin"                         # 除了 "log"，还有 "stdin"
            multiline:                                  # 多行合并
                pattern: '^[[:space:]]'
                negate: false
                match: after
output:
    ...
```

我们已完成了配置，当 `sudo service filebeat start` 之后，你就可以在 kibana 上看到你的日志了。

# 字段

Filebeat 发送的日志，会包含以下字段：

* `beat.hostname` beat 运行的主机名
* `beat.name` shipper 配置段设置的 `name`，如果没设置，等于 `beat.hostname`
* `@timestamp` 读取到该行内容的时间
* `type` 通过 `document_type` 设定的内容
* `input_type` 来自 "log" 还是 "stdin"
* `source` 具体的文件名全路径
* `offset` 该行日志的起始偏移量
* `message` 日志内容
* `fields` 添加的其他固定字段都存在这个对象里面

