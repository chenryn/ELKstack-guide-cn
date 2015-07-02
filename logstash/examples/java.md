# Java 日志

之前在 codec 章节，曾经提到过，对 Java 日志，除了使用 multiline 做多行日志合并以外，还可以直接通过 log4j 写入到 logstash 里。本节就讲述如何在 Java 应用环境做到这点。

## Log4J

首先，需要配置 Java 应用的 Log4J 设置，启动一个内置的 `SocketAppender`。修改应用的 `log4j.xml` 配置文件，添加如下配置段：

```
<appender name="LOGSTASH" class="org.apache.log4j.net.SocketAppender">
    <param name="RemoteHost" value="logstash_hostname" />
    <param name="ReconnectionDelay" value="60000" />
    <param name="LocationInfo" value="true" />
    <param name="Threshold" value="DEBUG" />
</appender>
```

然后把这个新定义的 appender 对象加入到 root logger 里，可以跟其他已有 logger 共存：

```
<root>
    <level value="INFO"/>
    <appender-ref ref="OTHERPLACE"/>
    <appender-ref ref="LOGSTASH"/>
</root>
```

如果是 `log4j.properties` 配置文件，则对应配置如下：

```
log4j.rootLogger=DEBUG, logstash

###SocketAppender###
log4j.appender.logstash=org.apache.log4j.net.SocketAppender
log4j.appender.logstash.Port=4560
log4j.appender.logstash.RemoteHost=logstash_hostname
log4j.appender.logstash.ReconnectionDelay=60000
log4j.appender.logstash.LocationInfo=true
```

Log4J 会持续尝试连接你配置的 *logstash_hostname* 这个地址，建立连接后，即开始发送日志数据。

## Logstash

Java 应用端的配置完成以后，开始设置 Logstash 的接收端。配置如下所示。其中 4560 端口是 Log4J SocketAppender 的默认对端端口。

```
input {
  log4j {
    type => "log4j-json"
    port => 4560
  }
}
```

### 异常堆栈测试验证

运行起来 logstash 后，编写如下一个简单 log4j 程序：

```
import org.apache.log4j.Logger;
public class HelloExample{
        final static Logger logger = Logger.getLogger(HelloExample.class);
        public static void main(String[] args) {
                HelloExample obj = new HelloExample();
                try{
                        obj.divide();
                }catch(ArithmeticException ex){
                        logger.error("Sorry, something wrong!", ex);
                }
        }
        private void divide(){
                int i = 10 /0;
        }
}
```

编译运行：

```
# javac -cp ./logstash-1.5.0.rc2/vendor/bundle/jruby/1.9/gems/logstash-input-log4j-0.1.3-java/lib/log4j/log4j/1.2.17/log4j-1.2.17.jar HelloExample.java
# java -cp .:./logstash-1.5.0.rc2/vendor/bundle/jruby/1.9/gems/logstash-input-log4j-0.1.3-java/lib/log4j/log4j/1.2.17/log4j-1.2.17.jar HelloExample
```

即可在 logstash 的终端输出看到如下事件记录：

```
{
        "message" => "Sorry, something wrong!",
       "@version" => "1",
     "@timestamp" => "2015-07-02T13:24:45.727Z",
           "type" => "log4j-json",
           "host" => "127.0.0.1:52420",
           "path" => "HelloExample",
       "priority" => "ERROR",
    "logger_name" => "HelloExample",
         "thread" => "main",
          "class" => "HelloExample",
           "file" => "HelloExample.java:9",
         "method" => "main",
    "stack_trace" => "java.lang.ArithmeticException: / by zero\n\tat HelloExample.divide(HelloExample.java:13)\n\tat HelloExample.main(HelloExample.java:7)"
}
```

可以看到，异常堆栈直接就记录在单行内了。

### JSON Event layout

如果无法采用 socketappender ，必须使用文件方式的，其实 Log4J 有一个 layout 特性，用来控制日志输出的格式。和 Nginx 日志自己拼接 JSON 输出类似，也可以通过 layout 功能，记录成 JSON 格式。推荐使用：<https://github.com/logstash/log4j-jsonevent-layout>
