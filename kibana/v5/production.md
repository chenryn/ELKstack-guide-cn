# 生产环境部署

Kibana5 是是一个完整的 web 应用。使用时，你需要做的只是打开浏览器，然后输入你运行 Kibana 的机器地址然后加上端口号。比如说：`localhost:5601` 或者 `http://YOURDOMAIN.com:5601`。

但是当你准备在生产环境使用 Kibana5 的时候，比起在本机运行，就需要多考虑一些问题：

* 在哪运行 kibana
* 是否需要加密 Kibana 出入的流量
* 是否需要控制访问数据的权限

## Nginx 代理配置

因为 Kibana5 不再是 Kibana3 那种纯静态文件的单页应用，所以其服务器端是需要消耗计算资源的。因此，如果用户较多，Kibana5 确实有可能需要进行多点部署，这时候，就需要用 Nginx 做一层代理了。

和 Kibana3 相比，Kibana5 的 Nginx 代理配置倒是简单许多，因为所有流量都是统一配置的。下面是一段包含入口流量加密、简单权限控制的 Kibana5 代理配置：

```
upstream kibana5 {
    server 127.0.0.1:5601 fail_timeout=0;
}
server {
    listen               *:80;
    server_name          kibana_server;
    access_log           /var/log/nginx/kibana.srv-log-dev.log;
    error_log            /var/log/nginx/kibana.srv-log-dev.error.log;

    ssl                  on;
    ssl_certificate      /etc/nginx/ssl/all.crt;
    ssl_certificate_key  /etc/nginx/ssl/server.key;

    location / {
        root   /var/www/kibana;
        index  index.html  index.htm;
    }

    location ~ ^/kibana5/.* {
        proxy_pass           http://kibana5;
        rewrite              ^/kibana5/(.*)  /$1 break;
        proxy_set_header     X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header     Host            $host;
        auth_basic           "Restricted";
        auth_basic_user_file /etc/nginx/conf.d/kibana.myhost.org.htpasswd;
    }
}
```

如果用户够多，当然你可以单独跑一个 kibana5 集群，然后在 `upstream` 配置段中添加多个代理地址做负载均衡。

## 配置 Kibana 和 shield 一起工作

Nginx 只能加密和管理浏览器到服务器端的请求，而 Kibana5 到 ELasticsearch 集群的请求，就需要由 Elasticsearch 方面来完成了。如果你在用 Shield 做 Elasticsearch 用户认证，你需要给 Kibana 提供用户凭证，这样它才能访问 `.kibana` 索引。Kibana 用户需要由权限访问 `.kibana` 索引里以下操作：

    '.kibana':
          - indices:admin/create
          - indices:admin/exists
          - indices:admin/mapping/put
          - indices:admin/mappings/fields/get
          - indices:admin/refresh
          - indices:admin/validate/query
          - indices:data/read/get
          - indices:data/read/mget
          - indices:data/read/search
          - indices:data/write/delete
          - indices:data/write/index
          - indices:data/write/update
          - indices:admin/create

更多配置 Shield 的内容，请阅读官网的 [Shield with Kibana 4](https://www.elastic.co/guide/en/shield/current/_shield_with_kibana_4.html)。

要配置 Kibana 的凭证，设置 `kibana.yml` 里的 `kibana_elasticsearch_username` 和 `kibana_elasticsearch_password` 选项即可：

    # If your Elasticsearch is protected with basic auth:
    kibana_elasticsearch_username: kibana5
    kibana_elasticsearch_password: kibana5

## 开启 ssl

Kibana 同时支持对客户端请求以及 Kibana 服务器发往 Elasticsearch 的请求做 SSL 加密。

要加密浏览器(或者在 Nginx 代理情况下，Nginx 服务器)到 Kibana 服务器之间的通信，配置 `kibana.yml` 里的 `ssl_key_file` 和 `ssl_cert_file` 参数：

    # SSL for outgoing requests from the Kibana Server (PEM formatted)
    ssl_key_file: /path/to/your/server.key
    ssl_cert_file: /path/to/your/server.crt

如果你在用 Shield 或者其他提供 HTTPS 的代理服务器保护 Elasticsearch，你可以配置 Kibana 通过 HTTPS 方式访问 Elasticsearch，这样 Kibana 服务器和 Elasticsearch 之间的通信也是加密的。

要做到这点，你需要在 `kibana.yml` 里配置 Elasticsearch 的 URL 时指明是 HTTPS 协议：

    elasticsearch: "https://<your_elasticsearch_host>.com:9200"

如果你给 Elasticsearch 用的是自己签名的证书，请在 `kibana.yml` 里设定 `ca` 参数指明 PEM 文件位置，这也意味着开启了 `verify_ssl` 参数：

    # If you need to provide a CA certificate for your Elasticsarech instance, put
    # the path of the pem file here.
    ca: /path/to/your/ca/cacert.pem

## 控制访问权限

你可以用 [Elasticsearch Shield](http://www.elastic.co/overview/shield/) 来控制用户通过 Kibana 可以访问到的 Elasticsearch 数据。Shield 提供了索引级别的访问控制。如果一个用户没被许可运行这个请求，那么它在 Kibana 可视化界面上只能看到一个空白。

要配置 Kibana 使用 Shield，你要为 Kibana 创建一个或者多个 Shield 角色(role)，以 `kibana5` 作为开头的默认角色。更详细的做法，请阅读 [Using Shield with Kibana 4](http://www.elastic.co/guide/en/shield/current/_shield_with_kibana_4.html)。
