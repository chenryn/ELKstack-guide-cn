# plugin

## input 中的 codec 

上一节大家可能注意到了，整个 pipeline 非常简单，无非就是一个多线程的线程间数据读写。但是，之前介绍的 codec 在哪里？我们可以看到 filter 阶段的 pop 操作，但是 Event 又是怎么进 `@input_to_filter` 的呢？这两个问题，并不在 pipeline 中完成，而是 plugin 中。

Logstash 从 1.5 开始，把各个 plugin 拆分成了单独的 gem，主代码里只留下了几个 `base.rb` 类。所以，要了解详细情况，我们需要阅读一个实际跑数据的插件，比如 `vendor/bundle/jruby/1.9/gems/logstash-input-file-0.1.6/lib/logstash/inputs/file.rb`。

可以看到其中最关键的读取数据部分代码如下：

```
    hostname = Socket.gethostname
    @tail.subscribe do |path, line|
      @logger.debug? && @logger.debug("Received line", :path => path, :text => line)
      @codec.decode(line) do |event|
        decorate(event)
        event["host"] = hostname if !event.include?("host")
        event["path"] = path
        queue << event
      end
    end
```

这里两个关键函数：`@codec.decode(line)` 和 `decorate(event)`。

@codec 在 `base.rb` 中默认为 plain，那么我们就继续看 `vendor/bundle/jruby/1.9/gems/logstash-codec-plain-0.1.5/lib/logstash/codecs/plain.rb` 的相关部分：

```
  def register
    @converter = LogStash::Util::Charset.new(@charset)
    @converter.logger = @logger
  end
  public
  def decode(data)
    yield LogStash::Event.new("message" => @converter.convert(data))
  end # def decode
```

超简短。就是在这个 `@codec.decode(line)` 里，生成了 `LogStash::Event` 对象。那么，我们通过 `output { codec => rubydebug }` 看到的除了 message 字段以外的那些数据，又是怎么来的呢？

继续看 `lib/logstash/event.rb` 的内容：

```
  CHAR_PLUS = "+"
  TIMESTAMP = "@timestamp"
  VERSION = "@version"
  VERSION_ONE = "1"
  TIMESTAMP_FAILURE_TAG = "_timestampparsefailure"
  TIMESTAMP_FAILURE_FIELD = "_@timestamp"

  public
  def initialize(data = {})
    @logger = Cabin::Channel.get(LogStash)
    @cancelled = false
    @data = data
    @accessors = LogStash::Util::Accessors.new(data)
    @data[VERSION] ||= VERSION_ONE
    @data[TIMESTAMP] = init_timestamp(@data[TIMESTAMP])
```

现在就清楚了，这个特殊的 `@timestamp` 是在 event 对象初始化的时候加上的，其实现为：

```
  private
  def init_timestamp(o)
    begin
      timestamp = o ? LogStash::Timestamp.coerce(o) : LogStash::Timestamp.now
```

再看 `lib/logstash/timestamp.rb` 中的实现：

```
  class Timestamp
    def initialize(time = Time.new)
      @time = time.utc
    end
    def self.now
      Timestamp.new(::Time.now)
    end
```

这就是我们看到 Logstash 生成的事件总是 UTC 时区时间的原因。

至于如果一开始就传入了 `@timestamp` 数据的处理，则是这样：

```
    JODA_ISO8601_PARSER = org.joda.time.format.ISODateTimeFormat.dateTimeParser
    UTC = org.joda.time.DateTimeZone.forID("UTC")
    def self.parse_iso8601(t)
      millis = JODA_ISO8601_PARSER.parseMillis(t)
      LogStash::Timestamp.at(millis / 1000, (millis % 1000) * 1000)

    def self.coerce(time)
      case time
      when String
        LogStash::Timestamp.parse_iso8601(time)
```

同样会利用 joda 库做一次解析，还是转换成 UTC 时区。

## output 中的 worker

我们知道，logstash 中，命令行的 `-w` 参数是设置 filter 插件的线程数的，而在 output 插件配置中，很多都另外有一个 `workers` 选项。这是从 `lib/logstash/ouput/base.rb` 中继承来的。其在 pipeline 中，是通过这行 `@outputs.each(&:worker_setup)` 调用的。

`worker_setup` 函数实现如下：

```
      define_singleton_method(:handle, method(:handle_worker))
      @worker_queue = SizedQueue.new(20)
      @worker_plugins = @workers.times.map { self.class.new(params.merge("workers" => 1, "codec" => @codec.clone)) }
      @worker_plugins.map.with_index do |plugin, i|
        Thread.new(original_params, @worker_queue) do |params, queue|
          LogStash::Util::set_thread_name(">#{self.class.config_name}.#{i}")
          plugin.register
          while true
            event = queue.pop
            plugin.handle(event)
          end
        end
      end
```

这个单例方法 `handle_worker` 的实现是：

```
  def handle_worker(event)
    @worker_queue.push(event)
  end
```

注意第一行，这里，对可以运行多线程，且确实配置了多线程的 output 插件，logstash 是给每个多线程插件单独准备一个依然固定大小 20 的线程安全数组。然后，pipeline 从 `@filter_to_output` 拿到的数据，再推进单个插件的 `@worker_queue` 里，由多线程来获取。

