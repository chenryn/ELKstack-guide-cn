# UserAgent 匹配归类

处理访问日志是 logstash 最常见的用途之一。在访问日志分析中，种类繁多还特别冗长的 User-Agent 是最难处理的一部分。为此，Logstash 提供了 filter/useragent 插件，帮助过滤归类。

## 配置示例

```
filter {
    useragent {
        target => "ua"
        source => "useragent"
    }
}
```
