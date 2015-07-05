# pipeline

在一开始，就介绍过，Logstash 对日志的处理，从 input 到 output，就像在 Linux 命令行上的管道操作一样。事实上，在 Logstash 中，对此有一个专门的名词，叫 Pipeline。

Pipeline 的代码加载路径如下：

> `bin/logstash` -> `lib/logstash/runner.rb` -> `lib/logstash/agent.rb` -> `lib/logstash/pipeline.rb`

这个最关键的 `pipeline.rb`，可以归纳成下面这么一段缩略版的代码：

```
    @config = grammar.parse(configstr)
    code = @config.compile
    eval(code)

    @input_to_filter = SizedQueue.new(20)
    @filter_to_output = SizedQueue.new(20)
    @settings = {
      "filter-workers" => 1,
    }

    @input_threads = []
    @inputs.each do |input|
      @input_threads << Thread.new { input.run(@input_to_filter) }
    end

    @filter_threads = @settings["filter-workers"].times.collect do
      Thread.new { 
        begin
          while true
            event = @input_to_filter.pop
            events = []
            filter(event) { |newevent| events << newevent }
            events.each { |event| @filter_to_output.push(event) }
          end
        end
      }
    end
    @flusher_lock = Mutex.new
    @flusher_thread = Thread.new { Stud.interval(5) { @flusher_lock.synchronize { @input_to_filter.push(FLUSH_EVENT) } } }

    @output_threads = [
      Thread.new {
        @outputs.each(&:worker_setup)
        while true
          event = @filter_to_output.pop
          output(event)
        end
      }
    ]

    @input_threads.each(&:join)
    @filter_threads.each(&:join)
    @output_threads.each(&:join)
```

整个缩略版，可以了解到一个关键信息，对我们理解 Logstash 原理是最有用的：`@input_to_filter` 和 `@filter_to_output`，都是一个固定大小为 20 的线程安全的数组。所以，任意一个 filter 或者 output 发生堵塞，都会一直堵塞到最前端的接收。这也是 logstash-input-heartbeat 的理论基础。
