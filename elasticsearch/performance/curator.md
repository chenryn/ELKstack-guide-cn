# curator

如果经过之前章节的一系列优化之后，数据确实超过了集群能承载的能力，除了拆分集群以外，最后就只剩下一个办法了：清除废旧索引。

为了更加方便的做清除数据，合并 segment，备份恢复等管理任务，Elasticsearch 在提供相关 API 的同时，另外准备了一个命令行工具，叫 curator 。curator 是 Python 程序，可以直接通过 pypi 库安装：

```
pip install elasticsearch-curator
```

*注意，是 elasticsearch-curator 不是 curator。PyPi 原先就有另一个项目叫这个名字*

## 参数介绍

和 Elastic Stack 里其他组件一样，curator 也是被 Elastic.co 收购的原开源社区周边。收编之后同样进行了一次重构，命令行参数从单字母风格改成了长单词风格。新版本的 curator 命令可用参数如下：

> Usage: curator [OPTIONS] COMMAND [ARGS]...

Options 包括:

  --host TEXT        Elasticsearch host.
  --url_prefix TEXT  Elasticsearch http url prefix.
  --port INTEGER     Elasticsearch port.
  --use_ssl          Connect to Elasticsearch through SSL.
  --http_auth TEXT   Use Basic Authentication ex: user:pass
  --timeout INTEGER  Connection timeout in seconds.
  --master-only      Only operate on elected master node.
  --dry-run          Do not perform any changes.
  --debug            Debug mode
  --loglevel TEXT    Log level
  --logfile TEXT     log file
  --logformat TEXT   Log output format [default|logstash].
  --version          Show the version and exit.
  --help             Show this message and exit.

Commands 包括:
  alias       Index Aliasing
  allocation  Index Allocation
  bloom       Disable bloom filter cache
  close       Close indices
  delete      Delete indices or snapshots
  open        Open indices
  optimize    Optimize Indices
  replicas    Replica Count Per-shard
  show        Show indices or snapshots
  snapshot    Take snapshots of indices (Backup)

针对具体的 Command，还可以继续使用 `--help` 查看该子命令的帮助。比如查看 *close* 子命令的帮助，输入 `curator close --help`，结果如下：

```
Usage: curator close [OPTIONS] COMMAND [ARGS]...

  Close indices

Options:
  --help  Show this message and exit.

Commands:
  indices  Index selection.
```

## 常用示例

在使用 1.4.0 以上版本的 Elasticsearch 前提下，curator 曾经主要的一个子命令 `bloom` 已经不再需要使用。所以，目前最常用的三个子命令，分别是 `close`, `delete` 和 `optimize`，示例如下：

```
curator --timeout 36000 --host 10.0.0.100 delete indices --older-than 5 --time-unit days --timestring '%Y.%m.%d' --prefix logstash-mweibo-nginx-
curator --timeout 36000 --host 10.0.0.100 delete indices --older-than 10 --time-unit days --timestring '%Y.%m.%d' --prefix logstash-mweibo-client- --exclude 'logstash-mweibo-client-2015.05.11'
curator --timeout 36000 --host 10.0.0.100 delete indices --older-than 30 --time-unit days --timestring '%Y.%m.%d' --regex '^logstash-mweibo-\d+'
curator --timeout 36000 --host 10.0.0.100 close indices --older-than 7 --time-unit days --timestring '%Y.%m.%d' --prefix logstash-
curator --timeout 36000 --host 10.0.0.100 optimize --max_num_segments 1 indices --older-than 1 --newer-than 7 --time-unit days --timestring '%Y.%m.%d' --prefix logstash-
```

这一顿任务，结果是：

*logstash-mweibo-nginx-yyyy.mm.dd* 索引保存最近 5 天，*logstash-mweibo-client-yyyy.mm.dd* 保存最近 10 天，*logstash-mweibo-yyyy.mm.dd* 索引保存最近 30 天；且所有七天前的 *logstash-\** 索引都暂时关闭不用；最后对所有非当日日志做 segment 合并优化。
