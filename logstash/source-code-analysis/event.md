## Logstash 中 Event 的生成

上一节大家可能注意到了，整个 pipeline 非常简单，无非就是一个多线程的线程间数据读写。但是，之前介绍的 codec 在哪里？这个问题，并不在 pipeline 中完成，而是 plugin 中。

Logstash 从 1.5 开始，把各个 plugin 拆分成了单独的 gem，主代码里只留下了几个 `base.rb` 类。所以，要了解详细情况，我们需要阅读一个实际跑数据的插件，比如 `vendor/bundle/jruby/1.9/gems/logstash-input-stdin-3.2.0/lib/logstash/inputs/stdin.rb`。

可以看到其中最关键的读取数据部分代码如下：

```
    @host = Socket.gethostname
    while !stop?
      if data = stdin_read
      @codec.decode(data) do |event|
        decorate(event)
        event.set("host", @host) if !event.include?("host")
        queue << event
      end
    end
```

这里两个关键函数：`@codec.decode(line)` 和 `decorate(event)`。

@codec 在 `stdin.rb` 中默认为 line，那么我们就继续看 `vendor/bundle/jruby/1.9/gems/logstash-codec-line-3.0.2/lib/logstash/codecs/line.rb` 的相关部分：

```
  def register
    require "logstash/util/buftok"
    @buffer = FileWatch::BufferedTokenizer.new(@delimiter)
    @converter = LogStash::Util::Charset.new(@charset)
    @converter.logger = @logger
  end
  public
  def decode(data)
    @buffer.extract(data).each do |line|
      yield LogStash::Event.new("message" => @converter.convert(line))
    end
  end # def decode
```

超简短。就是在这个 `@codec.decode(data)` 里，生成了 `LogStash::Event` 对象。那么，我们通过 `output { codec => rubydebug }` 看到的除了 message 字段以外的那些数据，又是怎么来的呢？尤其是那个 `@timestamp` 是怎么出来的？

在 5.0 之前，我们可以通过 `lib/logstash/event.rb` 看到相关属性的定义和操作。5.0 之后，Logstash 为了提高性能，对 Event 部分采用 Java 语言进行了重构，现在你在 `logstash-core-event-java/lib/logstash/event.rb` 里只能看到通过 JRuby 的专属 require 指令加载 jar 的语句了。

想要了解 Logstash::Event 的实际定义，需要去 Git 仓库下载，然后阅读 Java 源代码，你也可以直接通过网页阅读，地址是：<https://github.com/elastic/logstash/blob/master/logstash-core-event-java/src/main/java/org/logstash/Event.java>：

```
  public static final String METADATA = "@metadata";
  public static final String METADATA_BRACKETS = "[" + METADATA + "]";
  public static final String TIMESTAMP = "@timestamp";
  public static final String TIMESTAMP_FAILURE_TAG = "_timestampparsefailure";
  public static final String TIMESTAMP_FAILURE_FIELD = "_@timestamp";
  public static final String VERSION = "@version";
  public static final String VERSION_ONE = "1";

  public Event()
  {
      this.metadata = new HashMap<String, Object>();
      this.data = new HashMap<String, Object>();
      this.data.put(VERSION, VERSION_ONE);
      this.cancelled = false;
      this.timestamp = new Timestamp();
      this.data.put(TIMESTAMP, this.timestamp);
      this.accessors = new Accessors(this.data);
      this.metadata_accessors = new Accessors(this.metadata);
  }
```

现在就清楚了，这个特殊的 `@timestamp` 是在 event 对象初始化的时候加上的，其实现同样在这个 Java 源码中，见<https://github.com/elastic/logstash/blob/master/logstash-core-event-java/src/main/java/org/logstash/Timestamp.java>：

```
  public class Timestamp implements Cloneable {
    private DateTime time;
    public Timestamp() {
      this.time = new DateTime(DateTimeZone.UTC);
    }
  }
```

这就是我们看到 Logstash 生成的事件总是 UTC 时区时间的原因。

至于如果一开始就传入了 `@timestamp` 数据的处理，则是这样：

```
  public Timestamp(String iso8601) {
    this.time = ISODateTimeFormat.dateTimeParser().parseDateTime(iso8601).toDateTime(DateTimeZone.UTC);
  }
  public Timestamp(long epoch_milliseconds) {
    this.time = new DateTime(epoch_milliseconds, DateTimeZone.UTC);
  }
```

同样会利用 joda 库做一次解析，还是转换成 UTC 时区。
