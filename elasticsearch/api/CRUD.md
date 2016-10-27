# 增删改查

增删改查是数据库的基础操作方法。ES 虽然不是数据库，但是很多场合下，都被人们当做一个文档型 NoSQL 数据库在使用，原因自然是因为在接口和分布式架构层面的相似性。虽然在 Elastic Stack 场景下，数据的写入和查询，分别由 Logstash 和 Kibana 代劳，作为测试、调研和排错时的基本功，还是需要了解一下 ES 的增删改查用法的。

## 数据写入

ES 的一大特点，就是全 RESTful 接口处理 JSON 请求。所以，数据写入非常简单：

```
# curl -XPOST http://127.0.0.1:9200/logstash-2015.06.21/testlog -d '{
    "date" : "1434966686000",
    "user" : "chenlin7",
    "mesg" : "first message into Elasticsearch"
}'
```

命令返回响应结果为：

```
{"_index":"logstash-2015.06.21","_type":"testlog","_id":"AU4ew3h2nBE6n0qcyVJK","_version":1,"created":true}
```

## 数据获取

可以看到，在数据写入的时候，会返回该数据的 `_id`。这就是后续用来获取数据的关键：

```
# curl -XGET http://127.0.0.1:9200/logstash-2015.06.21/testlog/AU4ew3h2nBE6n0qcyVJK
```

命令返回响应结果为：

```
{"_index":"logstash-2015.06.21","_type":"testlog","_id":"AU4ew3h2nBE6n0qcyVJK","_version":1,"found":true,"_source":{
    "date" : "1434966686000",
    "user" : "chenlin7",
    "mesg" : "first message into Elasticsearch"
}}
```

这个 `_source` 里的内容，正是之前写入的数据。

如果觉得这个返回看起来有点太过麻烦，可以使用 `curl -XGET http://127.0.0.1:9200/logstash-2015.06.21/testlog/AU4ew3h2nBE6n0qcyVJK/_source` 来指明只获取源数据部分。

更进一步的，如果你只想看数据中的一部分字段内容，可以使用 `curl -XGET http://127.0.0.1:9200/logstash-2015.06.21/testlog/AU4ew3h2nBE6n0qcyVJK?fields=user,mesg` 来指明获取字段，结果如下：

```
{"_index":"logstash-2015.06.21","_type":"testlog","_id":"AU4ew3h2nBE6n0qcyVJK","_version":1,"found":true,"fields":{"user":["chenlin7"],"mesg":["first message into Elasticsearch"]}}
```

## 数据删除

要删除数据，修改发送的 HTTP 请求方法为 DELETE 即可：

```
# curl -XDELETE http://127.0.0.1:9200/logstash-2015.06.21/testlog/AU4ew3h2nBE6n0qcyVJK
```

删除不单针对单条数据，还可以删除整个整个索引。甚至可以用通配符。

```
# curl -XDELETE http://127.0.0.1:9200/logstash-2015.06.0*
```

在 Elasticsearch 2.x 之前，可以通过查询语句删除，也可以删除某个 `_type` 内的数据。现在都已经不再内置支持，改为 `Delete by Query` 插件。因为这种方式本身对性能影响较大！

## 数据更新

已经写过的数据，同样还是可以修改的。有两种办法，一种是全量提交，即指明 `_id` 再发送一次写入请求。

```
# curl -XPOST http://127.0.0.1:9200/logstash-2015.06.21/testlog/AU4ew3h2nBE6n0qcyVJK -d '{
    "date" : "1434966686000",
    "user" : "chenlin7",
    "mesg" " "first message into Elasticsearch but version 2"
}'
```

另一种是局部更新，使用 `/_update` 接口：

```
# curl -XPOST 'http://127.0.0.1:9200/logstash-2015.06.21/testlog/AU4ew3h2nBE6n0qcyVJK/_update' -d '{
    "doc" : {
        "user" : "someone"
    }
}'
```

或者

```
# curl -XPOST 'http://127.0.0.1:9200/logstash-2015.06.21/testlog/AU4ew3h2nBE6n0qcyVJK/_update' -d '{
    "script" : "ctx._source.user = \"someone\""
}'
```
