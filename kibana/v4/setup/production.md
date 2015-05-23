Kibana 是让你从 5601 端口访问的网页应用。你需要做的只是打开浏览器，然后输入你运行 Kibana 的机器地址然后加上端口号。比如说：`localhost:5601` 或者 `http://YOURDOMAIN.com:5601`。

当你访问 Kibana 的时候，默认会加载 Discover 页以及默认的索引模式。时间选择器默认为最近 15 分钟。而搜索条件是全部匹配(*)。

如果你没看到任何文档，尝试放宽时间选择器范围。如果还没有，可能你确实没往 Elasticsearch 里写进数据。

当你准备在生产环境使用 Kibana 的时候，比起在本机运行，你需要多考虑一些：

* 在哪运行 kibana
* 是否需要加密 Kibana 出入的流量
* 是否需要控制访问数据的权限

## 部署的考虑

你怎么部署 Kibana 取决于你的运用场景。如果就是自己用，在本机运行 Kibana 然后配置一下指向到任意你想交互的 Elasticsearch 实例即可。如果你有一大批 Kibana 重度用户，可能你需要部署多个 Kibana 实例，指向同一个 Elasticsearch ，然后前面加一个负载均衡。

虽然 Kibana 不是资源密集型的应用，我们依然建议你单独用一个节点来运行 Kibana，而不是泡在 Elasticsearch 节点上。

## 配置 Kibana 和 shield 一起工作

如果你在用 Shield 做 Elasticsearch 用户认证，你需要给 Kibana 提供用户凭证，这样它才能访问 `.kibana` 索引。Kibana 用户需要由权限访问 `.kibana` 索引里以下操作：

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

更多配置 Shield 的内容，请阅读 [Shield with Kibana 4](https://www.elasticsearch.org/guide/en/shield/current/_shield_with_kibana_4.html)。

要配置 Kibana 的凭证，设置 `kibana.yml` 里的 `kibana_elasticsearch_username` 和 `kibana_elasticsearch_password` 选项即可：

    # If your Elasticsearch is protected with basic auth:
    kibana_elasticsearch_username: kibana4
    kibana_elasticsearch_password: kibana4

## 开启 ssl

Kibana 同时支持对客户端请求以及 Kibana 服务器发往 Elasticsearch 的请求做 SSL 加密。

要加密浏览器到 Kibana 服务器之间的通信，配置 `kibana.yml` 里的 `ssl_key_file` 和 `ssl_cert_file` 参数：

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

你可以用 [Elasticsearch Shield](http://www.elasticsearch.org/overview/shield/) 来控制用户通过 Kibana 可以访问到的 Elasticsearch 数据。Shield 提供了索引级别的访问控制。如果一个用户没被许可运行这个请求，那么它在 Kibana 可视化界面上只能看到一个空白。

要配置 Kibana 使用 Shield，你要位 Kibana 创建一个或者多个 Shield 角色(role)，以 `kibana4` 作为开头的默认角色。更详细的做法，请阅读 [Using Shield with Kibana 4](http://www.elasticsearch.org/guide/en/shield/current/_shield_with_kibana_4.html)。
