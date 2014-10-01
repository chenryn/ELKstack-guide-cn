# Auth WebUI in Mojolicious

社区已经有用 nodejs 或者 rubyonrails 写的 kibana-auth 方案了。不过我这两种语言都不太擅长，只会写一点点 Perl5 代码，所以我选择用 [Mojolicious](http://mojolicio.us) web 开发框架来实现我自己的 kibana 认证鉴权。

整套方案的代码以 `kbnauth` 子目录形式存在于我的 [kibana 仓库](https://github.com/chenryn/kibana)中，如果你不想用这套认证方案，照旧使用 `src` 子目录即可。事实上，`kbnauth/public/` 目录下的静态文件我都是通过软连接方式指到 `src/`下的。

### 特性

* 全局透明代理

  和 nodejs 实现的那套方案不同，我这里并没有使用 `__es/` 这样附加的路径。所有发往 Elasticsearch 的请求都是通过这个方案来控制。除了使用 `config.js.ep` 模版来定制 `elasticsearch` 地址设置以外，方案还会伪造 `/_nodes` 请求的响应体，伪造的响应体中永远只有运行着认证方案的这台服务器的 IP 地址。

  *这么做的原因是我的 kibana 升级了 elasticjs 版本，新版本默认会通过这个 API 获取 nodes 列表，然后浏览器直接轮询多个 IP 获取响应。*

  **注意：Mojolicious 有一个环境变量叫 `max_message_size`，默认是 10MB，即只允许代理响应大小在 10MB 以内的数据。我在 `script/kbnauth` 启动脚本中把它修改成了 0，即不限制。如果你有这方面的需求，可以修改成任意你想要的阈值。**

* 使用 `kibana-auth` elasticsearch 索引做鉴权

  因为所有的请求都会发往代理服务器(即运行着认证鉴权方案的服务器)，每个用户都可以有自己的仪表板空间(没错，这招是从 kibana-proxy 项目学来的，每个用户使用单独的 `kibana-int-$username` 索引保存自己的仪表板设置)。而本方案还提供另一个高级功能：还可以通过另一个新的索引 `kibana-auth` 来指定每个用户所能访问的 Elasticsearch 集群地址和索引列表。

  给用户 "sri" 添加鉴权信息的命令如下：

  ```
  $ curl -XPOST http://127.0.0.1:9200/kibana-auth/indices/sri -d '{
    "prefix":["logstash-sri","logstash-ops"],
    "server":"192.168.0.2:9200"
  }'
   ```

  这就意味着用户 "sri" 能访问的，是存在 "192.168.0.2:9200" 上的 "logstash-sri-YYYY.MM.dd" 或者 "logstash-ops-YYYY.MM.dd" 索引。

  **小贴士：所以你在 `kbn_auth.conf` 里配置的 eshost/esport，其实并不意味着 kibana 数据的来源，而是认证方案用来请求 `kibana-auth` 信息的地址！**

* 使用 [Authen::Simple](https://metacpan.org/pod/Authen::Simple) 框架做认证

  Authen::Simple 是一个很棒的认证框架，支持非常多的认证方法。比如：LDAP, DBI, SSH, Kerberos, PAM, SMB, NIS, PAM, ActiveDirectory 等。

  默认使用的是 Passwd 方法。也就是用 `htpasswd` 命令行在本地生成一个 `.htpasswd` 文件存用户名密码。

  如果要使用其他方法，比如用 LDAP 认证，只需要配置 `kbn_auth.conf` 文件就行了：

  ```perl
    authen => {
      LDAP => {
        host   => 'ad.company.com',
        binddn => 'proxyuser@company.com',
        bindpw => 'secret',
        basedn => 'cn=users,dc=company,dc=com',
        filter => '(&(objectClass=organizationalPerson)(objectClass=user)(sAMAccountName=%s))'
      },
    }
  ```

  可以同时使用多种认证方式，但请确保每种都是有效可用的。某一个认证服务器连接超时也会影响到其他认证方式超时。

## 安装

该方案代码只有两个依赖：Mojolicious 框架和 Authen::Simple 框架。我们可以通过 cpanm 部署：

```
curl http://xrl.us/cpanm -o /usr/local/bin/cpanm
chmod +x /usr/local/bin/cpanm
cpanm Mojolicious Authen::Simple::Passwd
```

如果你需要使用其他认证方法，每个方法都需要另外单独安装。比如使用 LDAP 部署，就再运行一行：`cpanm Authen::Simple::LDAP` 就可以了。

*小贴士：如果你是在一个新 RHEL 系统上初次运行代码，你可能会发现有报错说找不到 `Digest::SHA` 模块。这个模块其实是 Perl 核心模块，但是 RedHat 公司把所有的 Perl 核心模块单独打包成了 `perl-core.rpm`，所以你得先运行一下 `yum install -y perl-core` 才行。我讨厌 RedHat！*

## 运行

```
cd kbnauth
# 开发环境监听 3000 端口，使用单进程的 morbo 服务器调试
morbo script/kbnauth
# 生产环境监听 80 端口，使用高性能的 hypnotoad 服务器, 具体端口在 kbn_auth.conf 中定义
hypnotoad script/kbnauth
```

现在，打开浏览器，就可以通过默认的用户名/密码："sri/secr3t" 登录进去了。(sri 是 Mojolicious 框架的作者，感谢他为 Perl5 社区提供这么高效的 web 开发框架)

注意：这时候你虽然认证通过进去了 kibana 页面，但是还没有赋权。按照上面提到的 `kibana-auth` 命令操作，才算全部完成。
