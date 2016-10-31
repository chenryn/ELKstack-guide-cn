# Puppet 自动部署 Elasticsearch

Elasticsearch 作为一个 Java 应用，本身的部署已经非常简单了。不过作为生产环境，还是有必要采用一些更标准化的方式进行集群的管理。Elasticsearch 官方提供并推荐使用 Puppet 方式部署和管理。其 Puppet 模块源码地址见：

<https://github.com/elastic/puppet-elasticsearch>

## 安装方法

和其他标准 Puppet Module 一样，puppet-elasticsearch 也可以通过 Puppet Forge 直接安装：

```
# puppet module install elasticsearch-elasticsearch
```

## 配置示例

安装好 Puppet 模块后，就可以使用了。模块提供几种 Puppet 资源，主要用法如下：

```
class { 'elasticsearch':
  version => '2.4.1',
  config => { 'cluster.name' => 'es1003' },
  java_install => true,
}
elasticsearch::instance { $fqdn:
  config => { 'node.name' => $fqdn },
  init_defaults => { 'ES_USER' => 'elasticsearch', 'ES_HEAP_SIZE' => $memorysize > 64 ? '31g' : $memorysize / 2 },
  datadir => [ '/data1/elasticsearch' ],
}
elasticsearch::template { 'templatename':
  host => $::ipaddress,
  port => 9200,
  content => '{"template":"*","settings":{"number_of_replicas":0}}'
}
```

示例中展示了三种资源：

* class: 配置具体安装的 Elasticsearch 软件版本，全集群公用的一些基础配置项。注意，puppet-elasticsearch 模块默认并不负责 Java 的安装，它只是调用操作系统对应的 Yum，Apt 工具，而 elasticsearch.rpm 或者 elasticsearch.deb 本身没有定义其他依赖(因为 java 版本太多了，定义起来不方便)。所以，如果依然要走 puppet-elasticsearch 配置来安装 Java 的话，需要额外开启 `java_install` 选项。该选项会使用另一个 Puppet Module —— [puppetlabs-java](https://forge.puppetlabs.com/puppetlabs/java) 来安装，默认安装的是 jdk。如果你要节省空间，可以再加一行 `java_package` 来明确指定软件全名。
* instance: 配置具体单个进程实例的配置。其中 `config` 和 `init_defaults` 选项在 class 和 instance 资源中都可以定义，实际运行时，会自动做一次合并，当然，instance 里的配置优先级高于 class 中的配置。此外，最重要的定义是数据目录的位置。有多快磁盘的，可以在这里定义一个数组。
* template: 模板是 Elasticsearch 创建索引映射和设置时的预定义方式。一般可以通过在 `config/templates/` 目录下放置 JSON 文件，或者通过 RESTful API 上传配置两种方式管理。而这里，单独提供了 template 资源，通过 puppet 来管理模板。`content` 选项中直接填入模板内容，或者使用 `file` 选项读取文件均可。

事实上，模块还提供了 plugin 和 script 资源管理这两方面的内容。考虑在 ELK 中，二者用的不是很多，本节就不单独介绍了。想了解的读者可以参考官方文档。
