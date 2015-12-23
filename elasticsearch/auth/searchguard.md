# Searchguard的部署与权限策略配置

*本节作者：elain*

##背景

当前es正在被各大互联网公司大量的使用，但目前安全方面还没有一个很成熟的方案，大部门都没有做安全认证或基于自身场景自己开发，没有一个好的开源方案
es官方推出了shield认证，试用了一番，很是方便，功能强大，文档也较全面，但最大的问题是收费的，我相信中国很多公司都不愿去花钱使用，所以随后在github
中找到了search-guard项目，接下来我们一起来了解并部署此项目到我们的ES环境中。

注：目前此项目只支持到1.6以下的es，1.7 还未支持，所以，我们在ES1.6下来部署此项目

##软件版本：

  * ES 1.6.0
  * kibana 4.0.2
  * CentOS 6.3

##功能特性：

  * 基于用户与角色的权限控制
  * 支持SSL/TLS方式安全认证
  * 支持LDAP认证
  * 支持最新的kibana4
  * 更多特性见官网介绍

官网地址：<http://floragunn.com/searchguard>

##目标：

实现用户访问es中日志需要登陆授权，不同用户访问不同索引，不授权的索引无法查看，分组控制不同rd查看各自业务的日志，


##部署


###download maven：
```
axel -n 10  http://mirror.bit.edu.cn/apache/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
tar zxvf apache-maven-3.3.3-bin.tar.gz
cd apache-maven-3.3.3/

#git search-guard and build
git clone -b es1.6 https://github.com/floragunncom/search-guard.git
cd search-guard ;/home/work/app/maven/bin/mvn package -DskipTests

#把编译好的包放到一个下载地址(方便于es集群使用，单台测试可不使用这种方案)：
http://www.elain.org/dl/search-guard-16-0.6-SNAPSHOT.zip
```
###在es上以插件方式安装编译好的包
```
cd /home/work/app/elasticsearch/plugins/
./bin/plugin -u http://www.elain.org/dl/search-guard-16-0.6-SNAPSHOT.zip -i search-guard
```
###elasticsearch.yml 添加
```
#################search-guard###################
searchguard.enabled: true
searchguard.key_path: /home/work/app/elasticsearch/keys
searchguard.auditlog.enabled: true
searchguard.allow_all_from_loopback: true #本地调试可打开，建议在线上关闭
searchguard.check_for_root: false
searchguard.http.enable_sessions: true

#配置认证方式
searchguard.authentication.authentication_backend.impl: com.floragunn.searchguard.authentication.backend.simple.SettingsBasedAuthenticationBackend
searchguard.authentication.authorizer.impl: com.floragunn.searchguard.authorization.simple.SettingsBasedAuthorizator
searchguard.authentication.http_authenticator.impl: com.floragunn.searchguard.authentication.http.basic.HTTPBasicAuthenticator

#配置用户名和密码
searchguard.authentication.settingsdb.user.admin: admin
searchguard.authentication.settingsdb.user.user1: 123
searchguard.authentication.settingsdb.user.user2: 123

#配置用户角色
searchguard.authentication.authorization.settingsdb.roles.admin: ["root"]
searchguard.authentication.authorization.settingsdb.roles.user1: ["user1"]
searchguard.authentication.authorization.settingsdb.roles.user2: ["user2"]

#配置角色权限（只读）
searchguard.actionrequestfilter.names: ["readonly","deny"]
searchguard.actionrequestfilter.readonly.allowed_actions: ["indices:data/read/*", "indices:admin/exists","indices:admin/mappings/*","indices:admin/validate/query"]
searchguard.actionrequestfilter.readonly.forbidden_actions: ["indices:data/write/*"]

#配置角色权限（禁止访问）
searchguard.actionrequestfilter.deny.allowed_actions: []
searchguard.actionrequestfilter.deny.forbidden_actions: ["indices:data/write/*"]
#################search-guard###################
```

###logging.yml 添加
```
logger.com.floragunn: DEBUG  #开启debug，方便调试
```
###创建key
```
echo 'be226fd1e6ddc74b' >/home/work/app/elasticsearch/keys/searchguard.key
```
重启elasticsearch

###配置权限策略如下 ：
```
curl -XPUT 'http://localhost:9200/searchguard/ac/ac?pretty' -d '
{"acl": [
    {
      "__Comment__": "Default is to execute all filters",
      "filters_bypass": [],
      "filters_execute": ["actionrequestfilter.deny"]
    }, //默认禁止访问
    {
      "__Comment__": "This means that every requestor (regardless of the requestors hostname and username) which has the root role can do anything",
      "roles": [
        "root"
      ],
      "filters_bypass": ["*"],
      "filters_execute": []
    }, // root角色完全权限
    {
      "__Comment__": "This means that for the user spock on index popstuff only the actionrequestfilter.readonly will be executed, no other",
      "users": ["user1"],
      "indices": ["index1-*","index2-*",".kibana"],
      "filters_bypass": ["actionrequestfilter.deny"],
      "filters_execute": ["actionrequestfilter.readonly"]
    }, //user1 用户只能访问index1-*,index2-* 索引，且只有只读权限 
    {
      "__Comment__": "This means that for the user spock on index popstuff only the actionrequestfilter.readonly will be executed, no other",
      "users": ["user2"],
      "indices": ["index3-*",".kibana"],
      "filters_bypass": ["actionrequestfilter.deny"],
      "filters_execute": ["actionrequestfilter.readonly"]
    } //user2 用户只能访问index3-* 索引，且只有只读权限 

  ]}}
```
###查看策略
```curl -XGET -uadmin:admin http://localhost:9200/searchguard/ac/ac ```

注：以上是我自己使用的策略，方便于不同用户访问不同索引，不授权的索引无法查看，分组控制不同rd查看各自业务的日志  

更多策略见：<https://github.com/floragunncom/search-guard/blob/es1.6/searchguard_config_example_1.yml>

更多配置与功能见：<https://github.com/floragunncom/search-guard/blob/es1.6/searchguard_config_template.yml>
