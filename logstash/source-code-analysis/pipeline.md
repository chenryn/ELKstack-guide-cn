# pipeline

在一开始，就介绍过，Logstash 对日志的处理，从 input 到 output，就像在 Linux 命令行上的管道操作一样。事实上，在 Logstash 中，对此有一个专门的名词，叫 Pipeline。

Pipeline 的代码加载路径如下：

> `bin/logstash` -> `logstash-core/lib/logstash/runner.rb` -> `logstash-core/lib/logstash/agent.rb` -> `logstash-core/lib/logstash/pipeline.rb`

Logstash 从 2.2 版开始对 pipeline 做了大幅重构，目前最新 5.0 版的 `pipeline.rb`，可以归纳成下面这么一段缩略版的代码：

```
    # 初始化阶段
    @config = grammar.parse(configstr)
    code = @config.compile
    eval(code)

    queue = LogStash::Util::WrappedSynchronousQueue.new
    @input_queue_client = queue.write_client
    @filter_queue_client = queue.read_client

    # 启动指标计数器
    @filter_queue_client.set_events_metric()
    @filter_queue_client.set_pipeline_metric()

    # 运行
    LogStash::Util.set_thread_name("[#{pipeline_id}]-pipeline-manager")

    # 启动输入插件
    @inputs.each do |input|
        input.register
        @input_threads << Thread.new do
            LogStash::Util::set_thread_name("[#{pipeline_id}]<#{input.class.config_name}")
            plugin.run(@input_queue)
        end
    end

    @outputs.each {|o| o.register }
    @filters.each {|f| f.register }

    max_inflight = batch_size * pipeline_workers
    pipeline_workers.times do |t|
        @worker_threads << Thread.new do
            LogStash::Util.set_thread_name("[#{pipeline_id}]>worker#{t}")
            @filter_queue_client.set_batch_dimensions(batch_size, batch_delay)
            while true
                batch = @filter_queue_client.take_batch

                # 开始过滤
                batch.each do |event|
                    filter_func(event).each do |e|
                        batch.merge(e)
                    end
                end
                # 计数
                @filter_queue_client.add_filtered_metrics(batch)

                # 开始输出
                output_events_map = Hash.new { |h,k|  h[k] = [] }
                batch.each do |event|
                    output_func(event).each do |output|
                        output_events_map[output].push(event)
                    end
                end
                output_events_map.each do |output, events|
                    output.multi_receive(events)
                end
                @filter_queue_client.add_output_metrics(batch)

                # 释放
                @filter_queue_client.close_batch(batch)
            end
        end
    end

    # 运行
    @input_threads.each(&:join)
```

整个缩略版，可以了解到一个关键信息，对我们理解 Logstash 原理是最有用的：`queue` 是一个固定大小为 0 的多线程同步队列。filter 和 output 插件，则在相同的 `pipeline_worker` 线程中运行，该线程每次批量获取数据，也批量传递给 filter 和 output 插件。

由于 input 到 filter 之间有唯一的队列，任意一个 filter 或者 output 发生堵塞，都会一直堵塞到最前端的接收。这也是 logstash-input-heartbeat 的理论基础。

这个全新的 NG pipeline 是从 2.2 版开始发布的，当时也导致 logstash-output-elasticsearch 的 ESClient 数量比过去大幅增加，对写入 Elasticsearch 的性能是不利的。随后官方意识到这个问题，并大举重构了 logstash-output-elasticsearch 的实现，改成了一个整体连接池的方式，代码见：<https://github.com/logstash-plugins/logstash-output-elasticsearch/commit/06a47535111881b2bc6c9dbd3908e664e4852476>。相关的新配置参数，在之前插件介绍中已经讲过。
