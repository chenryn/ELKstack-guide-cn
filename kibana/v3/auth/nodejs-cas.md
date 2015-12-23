# 配置Kibana的CAS验证

*本节作者：childe*

我们公司用的是 CAS 单点登陆, 用如下工具将kibana集成到此单点登陆系统

## 准备工具

- nginx: 仅仅是为了记录日志, 不用也行
- nodejs: 为了跑 kibana-authentication-proxy
- kibana: <https://github.com/elasticsearch/kibana>
- kibana-authentication-proxy: <https://github.com/fangli/kibana-authentication-proxy>

## 配置

1. nginx 配置 8080 端口, 反向代理到 es 的 9200
2. git clone kibana-authentication-proxy
3. git clone kibana
4. 将 kibana 软链接到 kibana-authentication-proxy 目录下
5. 配置 kibana-authentication-proxy/config.js

   可能有如下参数需要调整:
   
   ```
   es_host  	#这里是nginx地址
   es_port   	#nginx的8080
   listen_port  	#node的监听端口, 80
   listen_host 	#node的绑定IP, 可以0.0.0.0
   cas_server_url  #CAS地址
   ```

6. 安装 kibana-authentication-proxy 的依赖, `npm install express`, 等
7. 运行 `node kibana-authentication-proxy/app.js`

## 原理

- app.js 里面 `app.get('/config.js', kibana3configjs)`; 返回了一个新的 config.js, 不是用的 kibana/config.js, 在这个配置里面,调用 ES 数据的 URL 前面加了一个 `__es` 的前缀
- 在 app.js 入口这里, 有两个关键的中间层(我也不知道叫什么)被注册: 一个是 `configureCas`, 一个是`configureESProxy`
- 一个请求来的时候, 会到 `configureCas` 判断是不是已经登陆到 CAS, 没有的话就转到 cas 登陆页面
- `configureESProxy` 在 lib/es-proxy.js 里,  会把 `__es` 打头的请求(其实就是请求 es 数据的请求)转发到真正的 es 接口那里(我们这里是 nginx)

## 请求路径

	node(80) <=> nginx(8080) <=> es(9200)

kibana-authentication-proxy 本身没有记录日志的代码, 而且转发 es 请求用的流式的(看起来), 并不能记录详细的 request body. 所以我们就用 nginx 又代理一层做日志了..

