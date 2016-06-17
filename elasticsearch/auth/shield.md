# Shield 权限管理

*本节作者：cameluo*

Shield 是 Elastic 公司官方发布的权限管理产品。其主要特性包括：

* 提供集群节点身份验证和集群数据访问身份验证
* 提供基于身份角色的细粒度资源和行为访问控制，细到索引级别的读写控制
* 提供节点间数据传输通道加密保护输出传输安全
* 提供审计功能
* 以插件的形式发布

## License管理策略

Shield 是一款商业产品，不过提供 30 天免费试用，试用期间是全功能的。过期后集群会不再正常工作。

## Shield架构

### 用户认证：

Shield 通过定义一套用户集合来认证用户，采用抽象的域方式定义用户集合，支持本地用户(esusers 域)和 LDAP 用户(含 AD)。

* Shield 提供工具 `./bin/shield/esusers` 用于创建和管理本地用户。
* 集成 LDAP 认证支持映射 LDAP 安全组到 Shield 角色，LDAP 安全组与 Shield 角色可以是多对多的关系。

Shield 支持定义多个认证域，采用order字段进行优先级排序。如一个本地域 esusers，order=1，加一个 LDAP 域，order=2。如果用户不再本地用户域中则在 LDAP 域中查找验证。

* `./config/shield/roles.yml` 文件中定义角色和角色的所拥有的权限。
* `./config/shield/group_to_role_mapping.yml` 文件中定义 LDAP 组到角色映射关系。

### 节点认证与通道加密：

使用SSL/TLS证书进行相互认证和通讯加密。加密是可选配置。如果不使用，shield 节点之间可以进行简单的密码验证（明文传输）。

### 授权：

Shield 采用 RBAC 授权模型，数据模型包含如下元素： 

* 受保护资源(Secured Resource)：控制用户访问的客体，包括 cluster、index/alias 等等。
* 权能(Priviliege)：用户可以对受保护资源执行的一种或多种操作，如 read, write 等。
* 许可(Permissions)：对被保护的资源拥有的一个或多个权能，如 read on the "products" index。
* 角色(Role)：命名的一组许可。
* 用户(Users)：用户实体，可以被赋予 0 个或多种角色，授权他们对被保护的资源执行各种权能。

### 审计：

增加认证尝试、授权失败等安全相关事件和活动日志。

## 安装


1. 安装 License 和 Shield 插件

```
bin/plugin -i elasticsearch/license/latest
bin/plugin -i elasticsearch/shield/latest
```

注意：初次运行 Shield 需要重新启动 ES 集群。后续更新 License(license.json 为 License 文件)就可以在线运行：

```
# curl -XPUT -u admin 'http://127.0.0.1:9200/_licenses' -d @license.json
```

2. 创建本地管理员：

```
./bin/shield/esusers useradd esadmin -r admin
```

## 配置

这里使用简单的配置先完成基本验证：使用纯本地用户认证或者使用本地认证 + 基本的 ldap 认证。

### ES 配置

在elasticsearch.yml中增加如下配置：

```
hostname_verification: false
#shield.ssl.keystore.path:          /app/elasticsearch/node01.jks
#shield.ssl.keystore.password:      xxxxxx
shield:
  authc:
    realms:
      default:
        type: esusers
        order: 1
      ldaprealm:
        type: ldap
        order: 2
        url: "ldap://ldap.example.com:389"
        bind_dn: "uid=ldapuser, ou=users, o=services, dc=example, dc=com"
        bind_password: changeme
        user_search:
          base_dn: "dc=example,dc=com"
          attribute: uid
        group_search:
          base_dn: "dc=example,dc=com"
        files:
          role_mapping: "/app/elasticsearch/shield/group_to_role_mapping.yml"
        unmapped_groups_as_roles: false 
```

### 角色配置

根据默认配置文件增减角色和访问控制权限。角色配置文件可以在线修改，保存后立即生效：

> ([INFO ][shield.authz.store       ] [Winky Man] updated roles (roles file [/opt/elasticsearch/config/shield/roles.yml] changed))

注意：如果需要集成kibana认证，用户角色也需要有访问'.kibana'索引的访问权限和cluster:monitor/nodes/info的访问权限，具体参照kibana4角色中的定义，否则用户通过kibana认证后仍然无法访问到数据索引

### 用户组与角色映射配置

根据默认配置文件增减用户、用户组与角色配置中定义角色的映射关系，可以灵活实现各种需求。LDAP组仅支持安全组，不支持动态组。这个配置文件可以在线修改，保存后立即生效：

> ([INFO ][shield.authc.ldap.support] [Vishanti] role mappings file [/opt/elasticsearch/config/shield/group_to_role_mapping.yml] changed for realm [ldap/ldaprealm]. updating mappings...)

## 测试

`curl -u username http://127.0.0.1:9200/`
