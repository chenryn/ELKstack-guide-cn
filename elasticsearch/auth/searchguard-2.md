# searchguard 在 Elasticsearch 2.x 上的运用

*本节作者：wdh*

本节内容基于以下软件版本：

* elasticsearch 2.1.1-1
* kibana 4.3.1
* logstash 2.1.1-1

searchguard 2.x 更新后跟 shield 配置上很相似，相比之前的版本简洁很多。

searchguard 优点有：

* 节点之间通过 SSL/TLS 传输
* 支持 JDK SSL 和 Open SSL
* 支持热载入，不需要重启服务
* 支持 kibana4 及 logstash 的配置
* 可以控制不同的用户访问不同的权限
* 配置简单

## 安装

1. 安装search-guard-ssl

```bash
bin/plugin install com.floragunn/search-guard-ssl/2.1.1.5
```

2. 安装search-guard-2

```bash
bin/plugin install com.floragunn/search-guard-2/2.1.1.0-alpha2
```

3. 配置 elasticsearch 支持 ssl

elasticsearch.yml增加以下配置：

```yaml
#############################################################################################
#                                     SEARCH GUARD SSL                                      #
#                                       Configuration                                       #
#############################################################################################

#This will likely change with Elasticsearch 2.2, see [PR 14108](https://github.com/elastic/elasticsearch/pull/14108)
security.manager.enabled: false

#############################################################################################
# Transport layer SSL                                                                       #
#                                                                                           #
#############################################################################################

# Enable or disable node-to-node ssl encryption (default: true)
#searchguard.ssl.transport.enabled: false
# JKS or PKCS12 (default: JKS)
#searchguard.ssl.transport.keystore_type: PKCS12
# Relative path to the keystore file (mandatory, this stores the server certificates), must be placed under the config/ dir
searchguard.ssl.transport.keystore_filepath: node0-keystore.jks
# Alias name (default: first alias which could be found)
searchguard.ssl.transport.keystore_alias: my_alias
# Keystore password (default: changeit)
searchguard.ssl.transport.keystore_password: changeit

# JKS or PKCS12 (default: JKS)
#searchguard.ssl.transport.truststore_type: PKCS12
# Relative path to the truststore file (mandatory, this stores the client/root certificates), must be placed under the config/ dir
searchguard.ssl.transport.truststore_filepath: truststore.jks
# Alias name (default: first alias which could be found)
searchguard.ssl.transport.truststore_alias: my_alias
# Truststore password (default: changeit)
searchguard.ssl.transport.truststore_password: changeit
# Enforce hostname verification (default: true)
#searchguard.ssl.transport.enforce_hostname_verification: true
# If hostname verification specify if hostname should be resolved (default: true)
#searchguard.ssl.transport.resolve_hostname: true
# Use native Open SSL instead of JDK SSL if available (default: true)
#searchguard.ssl.transport.enable_openssl_if_available: false

#############################################################################################
# HTTP/REST layer SSL                                                                       #
#                                                                                           #
#############################################################################################
# Enable or disable rest layer security - https, (default: false)
#searchguard.ssl.http.enabled: true
# JKS or PKCS12 (default: JKS)
#searchguard.ssl.http.keystore_type: PKCS12
# Relative path to the keystore file (this stores the server certificates), must be placed under the config/ dir
#searchguard.ssl.http.keystore_filepath: keystore_https_node1.jks
# Alias name (default: first alias which could be found)
#searchguard.ssl.http.keystore_alias: my_alias
# Keystore password (default: changeit)
#searchguard.ssl.http.keystore_password: changeit
# Do the clients (typically the browser or the proxy) have to authenticate themself to the http server, default is false
#searchguard.ssl.http.enforce_clientauth: false
# JKS or PKCS12 (default: JKS)
#searchguard.ssl.http.truststore_type: PKCS12
# Relative path to the truststore file (this stores the client certificates), must be placed under the config/ dir
#searchguard.ssl.http.truststore_filepath: truststore_https.jks
# Alias name (default: first alias which could be found)
#searchguard.ssl.http.truststore_alias: my_alias
# Truststore password (default: changeit)
#searchguard.ssl.http.truststore_password: changeit
# Use native Open SSL instead of JDK SSL if available (default: true)
#searchguard.ssl.http.enable_openssl_if_available: false
```

4. 增加 searchguard 的管理员帐号配置，同样在 elasticsearch.yml 中，增加以下配置：

```yaml
security.manager.enabled: false
searchguard.authcz.admin_dn:
  - "CN=admin,OU=client,O=client,l=tEst,C=De" #DN
```

5. 重启 elasticsearch

将 node 证书和根证书放在 elasticsearch 配置文件目录下，证书可用 openssl 生成，[修改了下官方提供的脚本](https://github.com/wdh-001/searchguard/pki-scripts/example.sh)。

*注意：证书中的 client 的 DN 及 server 的 oid，证书不正确会导致 es 服务起不来。（我曾经用 ejbca 生成证书不能使用）*

## 配置

searchguard 主要有5个配置文件在 `plugins/search-guard-2/sgconfig` 下：

1. sg\_config.yml:

主配置文件不需要做改动

2. sg\_internal\_users.yml:

user 文件，定义用户。对于 ELK 我们需要一个 kibana 登录用户和一个 logstash 用户：

```yaml
kibana4:
  hash: $2a$12$xZOcnwYPYQ3zIadnlQIJ0eNhX1ngwMkTN.oMwkKxoGvDVPn4/6XtO
  #password is: kirk
  roles:
    - kibana4
logstash:
  hash: $2a$12$xZOcnwYPYQ3zIadnlQIJ0eNhX1ngwMkTN.oMwkKxoGvDVPn4/6XtO
```

密码可用[plugins/search-guard-2/tools/hash.sh](https://github.com/wdh-001/searchguard/blob/master/tools/hash.sh)生成

3. sg\_roles.yml:

权限配置文件，这里提供 kibana4 和 logstash 的权限，以下是我修改后的内容，可自行修改该部分内容（插件安装自带的配置文件中权限不够，kibana 会登录不了，shield 中同样的权限却是够了）：

```yaml
sg_kibana4:
  cluster:
      - cluster:monitor/nodes/info
      - cluster:monitor/health
  indices:
    '*':
      '*':
        - indices:admin/mappings/fields/get
        - indices:admin/validate/query
        - indices:data/read/search
        - indices:data/read/msearch
        - indices:admin/get
        - indices:data/read/field_stats
    '?kibana':
      '*':
        - indices:admin/exists
        - indices:admin/mapping/put
        - indices:admin/mappings/fields/get
        - indices:admin/refresh
        - indices:admin/validate/query
        - indices:data/read/get
sg_logstash:
  cluster:
    - indices:admin/template/get
    - indices:admin/template/put
  indices:
    'logstash-*':
      '*':
        - WRITE
        - indices:data/write/bulk
        - indices:data/write/delete
        - indices:data/write/update
        - indices:data/read/search
        - indices:data/read/scroll
        - CREATE_INDEX
```

4. sg\_roles\_mapping.yml:

定义用户的映射关系，添加 kibana 及 logstash 用户对应的映射：

```yaml
sg_logstash:
  users:
    - logstash
sg_kibana4:
  backendroles:
    - kibana
  users:
    - kibana4
```

5. sg\_action\_groups.yml:

定义权限

## 加载配置并启用

```bash
plugins/search-guard-2/tools/sgadmin.sh -cd plugins/search-guard-2/sgconfig/ -ks plugins/search-guard-2/sgconfig/admin-keystore.jks -ts plugins/search-guard-2/sgconfig/truststore.jks  -nhnv
```

（如修改了密码，则需要使用plugins/search-guard-2/tools/sgadmin.sh -h查看对应参数）

注意证书路径，将生成的admin证书和根证书放在sgconfig目录下。

最后，可以尝试登录啦！

登录界面会有验证

帐号：kibana4 密码：kirk

更多的权限配置可以自己研究。

请参考

https://github.com/floragunncom/search-guard/tree/2.1.1


