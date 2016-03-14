# pipeline

在一开始，就介绍过，Logstash 对日志的处理，从 input 到 output，就像在 Linux 命令行上的管道操作一样。事实上，在 Logstash 中，对此有一个专门的名词，叫 Pipeline。

Pipeline 的代码加载路径如下：

> `bin/logstash` -> `lib/logstash/runner.rb` -> `lib/logstash/agent.rb` -> `lib/logstash/pipeline.rb`

Logstash 2.2 版对 pipeline 做了大幅重构，最新版的 `pipeline.rb`，可以归纳成下面这么一段缩略版的代码：

```
    @config = grammar.parse(configstr)
    code = @config.compile
    eval(code)

    @input_queue = LogStash::Util::WrappedSynchronousQueue.new
    LogStash::Util.set_thread_name("[#{pipeline_id}]-pipeline-manager")

    @inputs.each do |input|
        input.register
        @input_threads << Thread.new do
            LogStash::Util::set_thread_name("[#{pipeline_id}]<#{input.class.config_name}")
            plugin.run(@input_queue)
        end
    end
    @outputs.each {|o| o.register }
    @filters.each {|f| f.register }
    @settings[:pipeline_workers].times do |t|
        @worker_threads << Thread.new do
            LogStash::Util.set_thread_name("[#{pipeline_id}]>worker#{t}")
            while true
                input_batch = []
                batch_size.times do |t|
                    event = (t == 0) ? @input_queue.take : @input_queue.poll(batch_delay)
                    input_batch << event
                end
                input_batch.reduce([]) do |acc,e|
                    filtered = filter_func(e)
                    filtered.each {|fe| acc << fe unless fe.cancelled?}
                    acc
                end
                .reduce(Hash.new { |h, k| h[k] = []  }) do |acc, event|
                    outputs_for_event = output_func(event) || []
                    outputs_for_event.each { |output| acc[output] << event  }
                    acc
                end
                .each { |output, events| output.multi_receive(events)  }
            end
        end
    end
```

整个缩略版，可以了解到一个关键信息，对我们理解 Logstash 原理是最有用的：`@input_queue` 是一个固定大小为 0 的多线程同步队列。filter 和 output 插件，则在相同的 `pipeline_worker` 线程中运行，该线程每次批量获取数据，也批量传递给 filter 和 output 插件。

由于 input 到 filter 之间有唯一的队列，任意一个 filter 或者 output 发生堵塞，都会一直堵塞到最前端的接收。这也是 logstash-input-heartbeat 的理论基础。

注：2.2 版这种改造，导致 logstash-output-elasticsearch 的 ESClient 数量比过去大幅增加，对写入 Elasticsearch 的性能是不利的。目前官方已经意识到这个问题，正在实现一个多线程共享的 ESClient 对象。在此之前，建议大家谨慎使用。
