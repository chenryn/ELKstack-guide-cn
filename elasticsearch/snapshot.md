# 镜像备份

大多数公司在使用 Elasticsearch 之前，都已经维护有一套 Hadoop 系统。因此，在实时数据慢慢变得冷却，不再被经常使用的时候，一个需求自然而然的就出现了：怎么把 Elasticsearch 索引数据快速转移到 HDFS 上，以解决 Elasticsearch 上的磁盘空间；而在我们需要的时候，又可以较快的从 HDFS 上把索引恢复回来继续使用呢？

Elasticsearch 为此提供了 snapshot 接口。通过这个接口，我们可以快速导入导出索引镜像到本地磁盘，网络磁盘，当然也包括 HDFS。

## 导出备份

## 导入恢复
