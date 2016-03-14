# 镜像备份

*本节作者：李宏旭*

大多数公司在使用 Elasticsearch 之前，都已经维护有一套 Hadoop 系统。因此，在实时数据慢慢变得冷却，不再被经常使用的时候，一个需求自然而然的就出现了：怎么把 Elasticsearch 索引数据快速转移到 HDFS 上，以解决 Elasticsearch 上的磁盘空间；而在我们需要的时候，又可以较快的从 HDFS 上把索引恢复回来继续使用呢？

Elasticsearch 为此提供了 snapshot 接口。通过这个接口，我们可以快速导入导出索引镜像到本地磁盘，网络磁盘，当然也包括 HDFS。

## HDFS 插件安装配置

下载[repository-hdfs插件](https://github.com/elastic/elasticsearch-hadoop/tree/master/repository-hdfs)，通过标准的 elasticsearch plugin 安装命令安装：

```
bin/plugin install elasticsearch/elasticsearch-repository-hdfs/2.2.0
```

然后在 elasticsearch.yml 中增加以下配置：

```
# repository 配置
hdfs:
    uri:"hdfs://<host>:<port>"(默认port为8020)
    #Hadoop file-system URI
    path:"some/path"
    #path with the file-system where data is stored/loaded
    conf.hdfs_config:"/hadoop/hadoop-2.5.2/etc/hadoop/hdfs-site.xml"
    conf.hadoop_config:"/hadoop/hadoop-2.5.2/etc/hadoop/core-site.xml"
    load_defaults:"true"
    #whether to load the default Hadoop configuration (default) or not
    compress:"false"
    # optional - whether to compress the metadata or not (default)
    chunk_size:"10mb"
    # optional - chunk size (disabled by default)
# 禁用 jsm
security.manager.enabled: false
```
默认情况下，Elasticsearch 为了安全考虑会在运行 JVM 的时候执行 JSM。出于 Hadoop 和 HDFS 客户端权限问题，所以需要禁用 JSM。将 `elasticsearch.yml` 中的 `security.manager.enabled` 设置为 `false`。

将插件安装好，配置修改完毕后，需要重启 Elasticsearch 服务。没有重启节点插件可能会执行失败。

**注意：Elasticsearch 集群的每个节点都要执行以上步骤！**

## Hadoop 配置

本节内容基于Hadoop版本：2.5.2，假定其配置文件目录：hadoop-2.5.2/etc/Hadoop。注意，安装hadoop集群需要建立主机互信，互信方法请自行查询，很简单。

相关配置文件如下：

### mapred-site.xml.template

默认没有 mapred-site.xml 文件，复制 mapred-site.xml.template 一份，并把名字改为 mapred-site.xml，需要修改 3 处的 IP 为本机地址:

```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobtracker.http.address</name>
        <value>XX.XX.XX.XX:50030</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value> XX.XX.XX.XX:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value> XX.XX.XX.XX:19888</value>
    </property>
</configuration>
```

### yarn-site.xml

需要修改5处的IP为本机地址:

```
<configuration>

<!-- Site specific YARN configuration properties -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
        <property>
        <name>yarn.resourcemanager.address</name>
        <value> XX.XX.XX.XX:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value> XX.XX.XX.XX:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value> XX.XX.XX.XX:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value> XX.XX.XX.XX:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value> XX.XX.XX.XX:8088</value>
    </property>
</configuration>
```

### hadoop-env.sh

修改 jdk 路径和 jvm 内存配置，内存使用根据情况配置。

```
export JAVA_HOME=/usr/java/jdk1.7.0_79
export HADOOP_PORTMAP_OPTS="-Xmx512m $HADOOP_PORTMAP_OPTS"
export HADOOP_CLIENT_OPTS="-Xmx512m $HADOOP_CLIENT_OPTS"
```

### core-site.xml

临时目录及 hdfs 机器 IP 端口指定:

<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/soft/hadoop-2.5.2/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs:// XX.XX.XX.XX:9000</value>
    </property>
    <property>
        <name>io.file.buffer.size</name>
        <value>4096</value>
    </property>
</configuration>

### slaves

配置集群 IP 地址，集群有几个 IP 都要配置进去:

```
192.168.0.2
192.168.0.3
192.168.0.4
```

### hdfs-site.xml

namenode 和 datenode 数据存放路径及别名:

```
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/data01/hadoop/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/data01/hadoop/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

## 启动 Hadoop

格式化完成后也可使用 `sbin/start-all.sh` 启动，但有可能出现异常，建议按照顺序分开启动。

1. 首先需要格式化存储：
`bin/Hadoop namenode –format`
2. 启动start-dfs.sh
`sbin/start-dfs.sh`
3. 启动start-yarn.sh
`sbin/start-yarn.sh`

## 备份导出

### 创建快照仓库

```
curl -XPUT 'localhost:9200/_snapshot/backup' -d 
'{
    "type":"hdfs",
    "settings":{
        "path":"/test/repo",
        "uri":"hdfs://<uri>:<port>"
    }
}'
```

在这步可能会报错。通常是因为 hadoop 配置问题，更改好配置需要重新格式化文件系统:

在 hadoop 目录下执行 `bin/hadoop namenode -format`

### 索引快照

执行索引快照命令，可写入crontab，定时执行

```
curl -XPUT 'http://localhost:9200/_snapshot/backup/snapshot_1' -d
 '{"indices":"indices_01,indices_02"}'
```

### 备份恢复

`curl -XPOST "localhost:9200/_snapshot/backup/snapshot_1/_restore"`

### 备份删除

`curl -XDELETE "localhost:9200/_snapshot/backup/snapshot_1"`

### 查看仓库信息

`curl -XGET 'http://localhost:9200/_snapshot/backup?pretty'`

