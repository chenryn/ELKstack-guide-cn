# 输出到 Statsd

## 基础知识

Statsd 最早是 2008 年 Flickr 公司用 Perl 写的针对 Graphite、datadog 等监控数据后端存储开发的前端网络应用，2011 年 Etsy 公司用 node.js 重构。用于接收(默认 UDP)、写入、读取和聚合时间序列数据，包括即时值和累积值等。

Graphite 是用 Python 模仿 RRDtools 写的时间序列数据库套件。包括三个部分：

* carbon: 是一个Twisted守护进程，监听处理数据；
* whisper: 存储时间序列的数据库；
* webapp: 一个用 Django 框架实现的网页应用。

### Graphite 安装简介

通过如下几步安装 Graphite：

1. 安装 cairo 和 pycairo 库

```
# yum -y install cairo pycairo
```

2. pip 安装

```
# yum install python-devel python-pip
# pip install django django-tagging carbon whisper graphite-web uwsgi
```

3. 配置 Graphite

```
# cd /opt/graphite/webapp/graphite
# cp local_settings.py.example local_settings.py
# python manage.py syncdb
```

修改 `local_settings.py` 中的 `DATABASE` 为设置的 db 信息。

4. 启动 cabon

```
# cd /opt/graphite/conf/
# cp carbon.conf.example carbon.conf
# cp storage-schemas.conf.example storage-schemas.conf
# cd /opt/graphite/
# ./bin/carbon-cache.py start
```

### statsd 安装简介

1. Graphite 地址设置

```
# cd /opt/
# git clone git://github.com/etsy/statsd.git
# cd /opt/statsd
# cp exampleConfig.js Config.js
```

根据 Graphite 服务器地址，修改Config.js 中的配置如下：

```
{
  graphitePort: 2003,
  graphiteHost: "10.10.10.124",
  port: 8125,
  backends: [ "./backends/graphite" ]
}
```

2. uwsgi 配置

```
cd /opt/graphite/webapp/graphite
cat > wsgi_graphite.xml <<EOF
<uwsgi>
  <socket>0.0.0.0:8630</socket>
  <workers>2</workers>
  <processes>2</processes>
  <listen>100</listen>
  <chdir>/opt/graphite/webapp/graphite</chdir>
  <pythonpath>..</pythonpath>
  <module>wsgi</module>
  <pidfile>graphite.pid</pidfile>
  <master>true</master>
  <enable-threads>true</enable-threads>
  <logdate>true</logdate>
  <daemonize>/var/log/uwsgi_graphite.log</daemonize>
</uwsgi>
EOF
cp /opt/graphite/conf/graphite.wsgi /opt/graphite/webapp/graphite/wsgi.py
```

3. nginx 的 uwsgi 配置

```
cat > /usr/local/nginx/conf/conf.d/graphite.conf <<EOF
server {
  listen 8081;
  server_name graphite;
  
  access_log /opt/graphite/storage/log/webapp/access.log ;
  error_log /opt/graphite/storage/log/webapp/error.log ;
  
  location / {
    uwsgi_pass 0.0.0.0:8630;
    include uwsgi_params;
    proxy_connect_timeout 300;
    proxy_send_timeout 300;
    proxy_read_timeout 300;
  }
}
EOF
```

4. 启动

```
# uwsgi -x /opt/graphite/webapp/graphite/wsgi_graphite.xml
# systemctl nginx reload
```

5. 数据测试

```
echo "test.logstash.num:100|c" | nc -w 1 -u $IP $port
```

如果安装配置是正常的，在graphite的左侧metrics->stats->test->logstash->num的表，statsd里面多了numStats等数据。

## 配置示例

```
output {
    statsd {
        host => "statsdserver.domain.com"
        namespace => "logstash"
        sender => "%{host}"
        increment => ["httpd.response.%{status}"]
    }
}
```

## 解释

Graphite 以树状结构存储监控数据，所以 statsd 也是如此。所以发送给 statsd 的数据的 key 也一定得是 "first.second.tree.four" 这样的形式。而在 *logstash-output-statsd* 插件中，就会以三个配置参数来拼接成这种形式：

```
    namespace.sender.metric
```

其中 namespace 和 sender 都是直接设置的，而 metric 又分为好几个不同的参数可以分别设置。statsd 支持的 metric 类型如下：

### metric 类型

* increment
增量，一个计量周期内，某个数字接收了多少次，比如 nginx 的 status 状态码。

示例语法：`increment => ["nginx.status.%{status}"]`

* decrement

语法同 increment。

* count
对数字的计数，比如，每秒接收一个数字，一个计量周期内，所有数字的和，比如nginx 的 body_bytes_sent。

示例语法：`count => {"nginx.bytes" => "%{bytes}"}`

* gauge

语法同 count。

* set

语法同 count。

* timing
时间范围内，某种数字的最大值，最小值，平均值，比如 nginx 的响应时间 request_time。

语法同 count。

关于这些 metric 类型的详细说明，请阅读 statsd 文档：<https://github.com/etsy/statsd/blob/master/docs/metric_types.md>。

## 推荐阅读

* Etsy 发布 nodejs 版本 statsd 的博客：[Measure Anything, Measure Everything](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/)
* Flickr 发布 statsd 的博客：[Counting & Timing](http://code.flickr.net/2008/10/27/counting-timing/)
