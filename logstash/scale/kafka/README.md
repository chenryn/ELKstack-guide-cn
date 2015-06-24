# 通过kafka传输

<https://github.com/joekiller/logstash-kafka>

插件已经正式合并进官方仓库，以下使用介绍基于**logstash 1.4相关版本**，1.5及以后版本的使用后续依照官方文档持续更新。

插件本身内容非常简单，其主要依赖同一作者写的 [jruby-kafka](https://github.com/joekiller/jruby-kafka) 模块。需要注意的是：**该模块仅支持 Kafka－0.8 版本。如果是使用 0.7 版本 kafka 的，将无法直接使 jruby-kafka 该模块和 logstash-kafka 插件。**

## 安装

* 安装按照官方文档完全自动化的安装.或是可以通过以下方式手动自己安装插件，不过重点注意的是 **kafka 的版本**，上面已经指出了。

> 1. 下载 logstash 并解压重命名为 `./logstash-1.4.0` 文件目录。

> 2. 下载 kafka 相关组件，以下示例选的为 [kafka_2.8.0-0.8.1.1-src](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.8.1.1/kafka-0.8.1.1-src.tgz)，并解压重命名为 `./kafka_2.8.0-0.8.1.1`。

> 3. 下载 logstash-kafka v0.4.2 从 [releases](https://github.com/joekiller/logstash-kafka/releases)，并解压重命名为 `./logstash-kafka-0.4.2`。

> 4. 从 `./kafka_2.8.0-0.8.1.1/libs` 目录下复制所有的 jar 文件拷贝到 `./logstash-1.4.0/vendor/jar/kafka_2.8.0-0.8.1.1/libs` 下，其中你需要创建 `kafka_2.8.0-0.8.1.1/libs` 相关文件夹及目录。

> 5. 分别复制 `./logstash-kafka-0.4.2/logstash` 里的 `inputs` 和 `outputs` 下的 `kafka.rb`，拷贝到对应的 `./logstash-1.4.0/lib/logstash` 里的 `inputs` 和 `outputs` 对应目录下。

> 6. 切换到 `./logstash-1.4.0` 目录下，现在需要运行 logstash-kafka 的 gembag.rb 脚本去安装 jruby-kafka 库，执行以下命令： `GEM_HOME=vendor/bundle/jruby/1.9 GEM_PATH= java -jar vendor/jar/jruby-complete-1.7.11.jar --1.9 ../logstash-kafka-0.4.2/gembag.rb ../logstash-kafka-0.4.2/logstash-kafka.gemspec`。

> 7. 现在可以使用 logstash-kafka 插件运行 logstash 了。例如：`bin/logstash agent -f logstash.conf`。
