# spark streaming 交互

Apache Spark 是一个高性能集群计算框架，其中 Spark Streaming 作为实时批处理组件，因为其简单易上手的特性深受喜爱。在 es-hadoop 2.1.0 版本之后，也新增了对 Spark 的支持，使得结合 ES 和 Spark 成为可能。

目前最新版本的 es-hadoop 是 2.1.0-Beta4。安装如下：

```
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.0.2-bin-cdh4.tgz
wget http://download.elasticsearch.org/hadoop/elasticsearch-hadoop-2.1.0.Beta4.zip
```

然后通过 `ADD_JARS=../elasticsearch-hadoop-2.1.0.Beta4/dist/elasticsearch-spark_2.10-2.1.0.Beta4.jar` 环境变量，把对应的 jar 包加入 Spark 的 jar 环境中。

下面是一段使用 spark streaming 接收 kafka 消息队列，然后写入 ES 的配置：

```
import org.apache.spark._
import org.apache.spark.streaming.kafka.KafkaUtils
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
import org.apache.spark.sql._
import org.elasticsearch.spark.sql._
import org.apache.spark.storage.StorageLevel
import org.apache.spark.Logging
import org.apache.log4j.{Level, Logger}

object Elastic {
  def main(args: Array[String]) {
    val numThreads = 1
    val zookeeperQuorum = "localhost:2181"
    val groupId = "test"
    val topic = Array("test").map((_, numThreads)).toMap
    val elasticResource = "apps/blog"

    val sc = new SparkConf()
                 .setMaster("local[*]")
                 .setAppName("Elastic Search Indexer App")

    sc.set("es.index.auto.create", "true")
    val ssc = new StreamingContext(sc, Seconds(10))
    ssc.checkpoint("checkpoint")
    val logs = KafkaUtils.createStream(ssc,
                                       zookeeperQuorum,
                                       groupId,
                                       topic,
                                       StorageLevel.MEMORY_AND_DISK_SER)
                         .map(_._2)
                         
    logs.foreachRDD { rdd =>
      val sc = rdd.context
      val sqlContext = new SQLContext(sc)
      val log = sqlContext.jsonRDD(rdd)
      log.saveToEs(elasticResource)
    }

    ssc.start()
    ssc.awaitTermination()

  }
}
```

注意，代码中使用了 spark SQL 提供的 `jsonRDD()` 方法，如果在对应的 kafka topic 里的数据，本身并不是已经处理好了的 JSON 数据的话，这里还需要自己写一写额外的处理函数，利用 `cast class` 来规范数据。
