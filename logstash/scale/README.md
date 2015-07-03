# 扩展方案

之前章节中，讲述的都是单个 logstash 进程，如何配置实现对数据的读取、解析和输出处理。但是在生产环境中，从每台应用服务器运行 logstash 进程并将数据直接发送到 Elasticsearch 里，显然不是第一选择：第一，过多的客户端连接对 Elasticsearch 是一种额外的压力；第二，网络抖动会影响到 logstash 进程，进而影响生产应用；第三，运维人员未必愿意在生产服务器上部署 Java，或者让 logstash 跟业务代码争夺 Java 资源。

所以，在实际运用中，logstash 进程会被分为两个不同的角色。运行在应用服务器上的，尽量减轻运行压力，只做读取和转发，这个角色叫做 shipper；运行在独立服务器上，完成数据解析处理，负责写入 Elasticsearch 的角色，叫 indexer。

![](http://www.infoq.com/resource/articles/review-the-logstash-book/en/resources/2fig2.jpg)

logstash 作为无状态的软件，配合消息队列系统，可以很轻松的做到线性扩展。本节首先会介绍最常见的两个消息队列与 logstash 的配合。

此外，logstash 作为一个框架式的项目，并不排斥，甚至欢迎与其他类似软件进行混搭式的运行。本节也会介绍一些其他日志处理框架以及如何和 logstash 共存的方式（ 《logstashbook》也同样有类似内容）。希望大家各取所长，做好最适合自己的日志处理系统。
