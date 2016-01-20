# filebeat

filebeat 是基于原先 logstash-forwarder 的源码改造出来的。换句话说：filebeat 就是新版的 logstash-forwarder，也会是 ELK Stack 在 shipper 端的第一选择。

![](https://www.elastic.co/guide/en/beats/filebeat/current/images/filebeat.png)

## 安装部署

* deb:
```
curl -L -O https://download.elastic.co/beats/filebeat/filebeat_1.0.1_amd64.deb
sudo dpkg -i filebeat_1.0.1_amd64.deb
```
* rpm:
```
curl -L -O https://download.elastic.co/beats/filebeat/filebeat-1.0.1-x86_64.rpm
sudo rpm -vi filebeat-1.0.1-x86_64.rpm
```
* mac:
```
curl -L -O https://download.elastic.co/beats/filebeat/filebeat-1.0.1-darwin.tgz
tar xzvf filebeat-1.0.1-darwin.tgz
```
* win:

1. 下载 https://download.elastic.co/beats/filebeat/filebeat-1.0.1-windows.zip
2. 解压到 C:\Program Files
3. 重命名 filebeat-1.0.1-windows 目录为 Filebeat
4. 右键点击 PowerSHell 图标，选择『以管理员身份运行』
5. 运行下列命令，将 Filebeat 安装成 windows 服务：
```
PS > cd 'C:\Program Files\Filebeat'
PS C:\Program Files\Filebeat> .\install-service-filebeat.ps1
```
Note
If script execution is disabled on your system, you need to set the execution policy for the current session to allow the script to run. For example: PowerShell.exe -ExecutionPolicy RemoteSigned -File .\install-service-filebeat.ps1.


## logstash-input-beats 配置

在 logstash 作为 indexer server 角色的这端，我们首先需要生成证书：

    cd /etc/pki/tls
    sudo openssl req -x509 -batch -nodes -days 3650 -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt

然后把证书发送到准备运行 logstash-forwarder 的 shipper 端服务器上去：

    scp private/logstash-forwarder.key root@target_server_ip:/etc/pki/tls/private
    scp certs/logstash-forwarder.crt root@target_server_ip:/etc/pki/tls/certs

```
./bin/plugin install logstash-input-beats
```

然后创建 logstash 的配置文件，内容如下：

```
input {
    beats {
        port => 5044
    }
}
output {
    elasticsearch {
        hosts => "localhost:9200"
        sniffing => true
        manage_template => false
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
        document_type => "%{[@metadata][type]}"
    }
}
```

## shipper 端配置

我们现在登入到我们需要传送 log 的机器上，我们已在之前的步骤中发送了 logstash 的 crt 过来。

### logstash-forwarder 安装

首先，我们需要安装 logstash-forwarder 软件。官方都已经提供了软件仓库可用。在 Redhat 机器上只需要添加一个 */etc/yum.repos.d/logstash-forwarder.repo*，内容如下：

```ini
[logstash-forwarder]
name=logstash-forwarder
baseurl=http://packages.elasticsearch.org/logstash-forwarder/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1
```

然后运行安装命令即可：

    sudo yum install -y logstash-forwarder

你可以从我提供的 gist 中下载已经更改的 init script 或者使用 rpm 中提供的脚本 [logstash-forwader](https://gist.github.com/ae30a4c1a1f342df1274.git).

### logstash-forwarder 配置

logstash-forwarder 的配置文件是纯 JSON 格式。因为其轻量级的设计目的，所以可配置项很少。下面是一个 */etc/logstash-forwarder* 配置示例：

```json
{
  "network": {
    "servers": [ "10.18.10.2:5000" ],
      "timeout": 15,
      "ssl ca" : "/etc/pki/tls/certs/logstash-forwarder.crt"
      "ssl key": "/etc/pki/tls/private/logstash-forwarder.key"
  },
  "files": [
    {
      "paths": [
        "/var/log/message",
      "/var/log/secure"
        ],
      "fields": { "type": "syslog" }
    }
  ]
}
```

我们已完成了配置，当 `sudo service logstash-forwarder start` 之后，你就可以在 kibana 上看到你的日志了

### logstash-forwarder 配置说明

配置中，主要包括下面几个可用配置项：

* network.servers: 用来指定远端(即 logstash indexer 角色)服务器的 IP 地址和端口。这里可以写数组，但是 logstash-forwarder 只会随机选一台作为对端发送数据，一直到对端故障，才会重选其他服务器。
* network.ssl\*: 网络交互中使用的 SSL 证书路径。
* files.\*.paths: 读取的文件路径。
  logstash-forwarder 只支持两种输入，一种就是示例中用的文件方式，和 logstash 一样也支持 glob 路径，即 `"/var/log/*.log"` 这样的写法；一种是标准输入，写法为 `"paths": [ "-" ]`
* files.\*.fields: 给每条日志添加的固定字段，相当于 logstash 里的 `add_field` 参数。
  注意示例中添加的是 **type** 字段。在 logstash-forwarder 里添加的字段是优先于 LogStash::Inputs::Lumberjack 配置里定义的字段的。所以，在本例中，即便你在 indexer 上定义 type 为 "anything"。事件的实际 type 依然是这里添加的 "syslog"。这也意味着，你在 indexer 上如果做后续判断，应该是这样：

```
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
    }
  }
}
```

