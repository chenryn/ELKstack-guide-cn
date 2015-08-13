# 安装

## 下载

Logstash 从 1.5 版本开始，将核心代码和插件代码完全剥离，并重构了插件架构逻辑，所有插件都以标准的 Ruby Gem 包形式发布。不过，为了方便大家从 1.4 版本过度，目前，官方依然发布打包有所有官方维护的插件在内的软件包。只是不再发布类似原先 1.4 时代的 `logstash-contrib.tar.gz` 这样的软件包了。依然要使用社区插件的读者，请阅读稍后插件安装章节。

下载官方软件包的方式有以下几种：

* 压缩包方式

```
wget https://download.elastic.co/logstash/logstash/logstash-1.5.1.tar.gz
```

* Debian 平台

```
wget https://download.elastic.co/logstash/logstash/packages/debian/logstash_1.5.1-1_all.deb
```

* Redhat 平台

```
wget https://download.elastic.co/logstash/logstash/packages/centos/logstash-1.5.1-1.noarch.rpm
```

## 安装

上面这些包，你可能更偏向使用 `rpm`，`dpkg` 等软件包管理工具来安装 Logstash，开发者在软件包里预定义了一些依赖。比如，`logstash-1.5.1-1.narch` 就依赖于 `jre` 包。

另外，软件包里还包含有一些很有用的脚本程序，比如 `/etc/init.d/logstash`。

如果你必须得在一些很老的操作系统上运行 Logstash，那你只能用源代码包部署了，记住要自己提前安装好 Java：

```
yum install openjdk-jre
export JAVA_HOME=/usr/java
tar zxvf logstash-1.5.1.tar.gz
```

## 最佳实践

但是真正的建议是：如果可以，请用 Elasticsearch 官方仓库来直接安装 Logstash！

### Debian 平台

```bash
wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | apt-key add -
cat >> /etc/apt/sources.list <<EOF
deb http://packages.elasticsearch.org/logstash/1.5/debian stable main
EOF
apt-get update
apt-get install logstash
```

### Redhat 平台

```
rpm --import http://packages.elasticsearch.org/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/logstash.repo <<EOF
[logstash-1.5]
name=logstash repository for 1.5.x packages
baseurl=http://packages.elasticsearch.org/logstash/1.5/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1
EOF
yum clean all
yum install logstash
```
