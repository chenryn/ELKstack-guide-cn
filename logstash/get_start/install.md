# 安装

## 下载

Logstash 从 1.5 版本开始，将核心代码和插件代码完全剥离，并重构了插件架构逻辑，所有插件都以标准的 Ruby Gem 包形式发布。不过，为了方便大家从 1.4 版本过度，目前，官方依然发布打包有所有官方维护的插件在内的软件包。只是不再发布类似原先 1.4 时代的 `logstash-contrib.tar.gz` 这样的软件包了。依然要使用社区插件的读者，请阅读稍后插件安装章节。

直接下载官方发布的二进制包的，可以访问 <https://www.elastic.co/downloads/logstash> 页面找对应操作系统和版本，点击下载即可。不过更推荐使用软件仓库完成安装。

## 安装

如果你必须得在一些很老的操作系统上运行 Logstash，那你只能用源代码包部署了，记住要自己提前安装好 Java：

```
yum install java-1.8.0-openjdk
export JAVA_HOME=/usr/java
```

软件仓库的配置，主要两大平台如下：

### Debian 平台

```bash
wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | apt-key add -
cat >> /etc/apt/sources.list <<EOF
deb http://packages.elasticsearch.org/logstash/5.0/debian stable main
EOF
apt-get update
apt-get install logstash
```

### Redhat 平台

```
rpm --import http://packages.elasticsearch.org/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/logstash.repo <<EOF
[logstash-5.0]
name=logstash repository for 5.0.x packages
baseurl=http://packages.elasticsearch.org/logstash/5.0/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1
EOF
yum clean all
yum install logstash
```
