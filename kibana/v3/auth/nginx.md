# nginx 代理和简单权限验证

Kibana3 作为一个纯静态文件式的单页应用，可以运行在任意主机上，却要求所有用户的浏览器，都可以直连 ELasticsearch 集群。这对网络和数据安全都是极为不利的。所以，一般在生产环境的 Elastic Stack，都是采取 HTTP 代理层的方式来做一层防护。最简单的办法，就是使用 Nginx 代理配置。

ES 官方也提供了一个推荐配置：<https://github.com/elastic/kibana/blob/3.0/sample/nginx.conf>

```
server {
  listen                *:80;

  server_name           kibana.myhost.org;
  access_log            /var/log/nginx/kibana.myhost.org.access.log;

  location / {
    root  /usr/share/kibana3;
    index  index.html  index.htm;
  }

  location ~ ^/_aliases$ {
    proxy_pass http://127.0.0.1:9200;
    proxy_read_timeout 90;
  }
  location ~ ^/.*/_aliases$ {
    proxy_pass http://127.0.0.1:9200;
    proxy_read_timeout 90;
  }
  location ~ ^/_nodes$ {
    proxy_pass http://127.0.0.1:9200;
    proxy_read_timeout 90;
  }
  location ~ ^/.*/_search$ {
    proxy_pass http://127.0.0.1:9200;
    proxy_read_timeout 90;
  }
  location ~ ^/.*/_mapping {
    proxy_pass http://127.0.0.1:9200;
    proxy_read_timeout 90;
  }

  # Password protected end points
  location ~ ^/kibana-int/dashboard/.*$ {
    proxy_pass http://127.0.0.1:9200;
    proxy_read_timeout 90;
    limit_except GET {
      proxy_pass http://127.0.0.1:9200;
      auth_basic "Restricted";
      auth_basic_user_file /etc/nginx/conf.d/kibana.myhost.org.htpasswd;
    }
  }
  location ~ ^/kibana-int/temp.*$ {
    proxy_pass http://127.0.0.1:9200;
    proxy_read_timeout 90;
    limit_except GET {
      proxy_pass http://127.0.0.1:9200;
      auth_basic "Restricted";
      auth_basic_user_file /etc/nginx/conf.d/kibana.myhost.org.htpasswd;
    }
  }
}
```

由此也可以看到，Kibana3 实际往 ES 发起的请求，都有哪些。

然后，在同时运行了 Nginx 和 Elasticsearch 服务(建议为 client 角色)的这台服务器上，配置 `elasticsearch.yml` 为：

```
network.bind_host: 127.0.0.1
network.publish_host: 127.0.0.1
network.host: 127.0.0.1
```

其他 Elasticsearch 节点统一关闭 HTTP 服务，配置 `elasticsearch.yml`:

```
http.enabled: false
```

这样，所有人都只能从 Nginx 服务来查询 ES 服务了。从 Nginx 日志中，我们还可以审核查询请求的语法性能和合理性。

注意：改配置前提是 Logstash 采用 node/transport 协议写入数据，如果使用 http 协议的，请采用 iptables 防护，或新增 `/_bulk` 等写入接口的代理配置。
