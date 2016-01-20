# AIX 平台上的 Logstash-Forwarder-Java

*本节作者：鹏程*

在AIX环境下（IBM Power小型机的一种操作系统）,你无法使用logstash（因为IBM JDK没有实现相关方法），也无法使用logstash-forwarder，github上有个Logstash-forwarder再实现的项目就是解决这个问题的：<https://github.com/didfet/logstash-forwarder-java>

配置和Logstash-forwarder基本一致，但是注意有一个坑是需要关注的，作者也在他的github上提到了，就是：

```
the ssl ca parameter points to a java keystore containing the root certificate of the server, not a PEM file
```

不熟悉证书相关体系的读者可能不太清楚这个意思，换句话说，如果你还按照logstash-forwarder的配置方法配置shipper端，那么你将会得到一个诡异的java.io.IOException: Invalid keystore format  异常。

首先介绍下背景知识，摘录一段知乎上的讲解： @刘长元 from <http://www.zhihu.com/question/29620953>

> 把SSL系统比喻为工商局系统。
> 首先有SSL就有CA，certificate authority。证书局，用于制作、认证证书的第三方机构，我们假设营业执照非常难制作，就像身份证一样，需要有制证公司来提供，并且提供技术帮助工商局验证执照的真伪。然后CA是可以有多个的，也就是可以有多个制证公司，但工商局就只有一个，它来说那个制证公司是可信的，那些是假的，需要打击。在SSL的世界中，微软、Google和Mozilla扮演了一部分这个角色。也就是说，IE、Chrome、Firefox中内置有一些CA，经过这些CA颁发，验证过的证书都是可以信的，否则就会提示你不安全。
> 这也是为什么前几天Chrome决定屏蔽CNNIC的CA时，CNNIC那么遗憾了。
> 也因为内置的CA是相对固定的，所以当你决定要新建网站时，就需要购买这些内置CA颁发的证书来让用户看到你的域名前面是绿色的，而不是红色。而这个最大的卖证书的公司就是VeriSign如果你听说过的话，当然它被卖给了Symantec，这家伙不只出Ghost，还是个卖证书的公司。
> 
> 要开店的老板去申请营业执照的时候是需要交他的身份证的，然后办出来的营业执照上也会有他的照片和名字。身份证相当于私钥，营业执照就是证书，Ceritficate，.cer文件。
> 
> 然后关于私钥和公钥如何解释我没想好，而它们在数据加密层面，数据的流向是这样的。
> 
> 消息-->[公钥]-->加密后的信息-->[私钥]-->消息
> 
> 公钥是可以随便扔给谁的，他把消息加了密传给我。对了，可以这样理解，我有一个箱子，一把锁和一把钥匙，我把箱子和开着的锁给别人，他写了信放箱子里，锁上，然后传递回我手上的途中谁都是打不开箱子的，只有我可以用原来的钥匙打开，这就是SSL，公钥，私钥传递加密消息的方式。这里的密钥就是key文件。
> 
> 于是我们就有了.cer和.key文件。接下来说keystore
> 
> 不同的语言、工具序列SSL相关文件的格式和扩展名是不一样的。
> 其中Java系喜欢用keystore, truststore来干活，你看它的名字，Store，仓库，它里面存放着key和信任的CA，key和CA可以有多个。
> 这里的truststore就像你自己电脑的证书管理器一样，如果你打开Chrome的设置，找到HTTP SSL，就可以看到里面有很多CA，truststore就是干这个活儿的，它也里面也是存一个或多个CA让Tomcat或Java程序来调用。
> 而keystore就是用来存密钥文件的，可以存放多个。
> 
> 然后是PEM，它是由RFC1421至1424定义的一种数据格式。其实前面的.cert和.key文件都是PEM格式的，只不过在有些系统中（比如Windows）会根据扩展名不同而做不同的事。所以当你看到.pem文件时，它里面的内容可能是certificate也可能是key，也可能两个都有，要看具体情况。可以通过openssl查看。

看到这儿你就应该懂了，按照logstash-forwarder-java的作者设计，此时你的shipper端ssl ca这个域配置的应该是keystore，而不是PEM，因此需要从你生成的crt中创建出keystore（jks）文件，方法为：

```
keytool -importcert -trustcacerts -file logstash-forwarder.crt -alias ca -keystore keystore.jks
```

一个示例的shipper.conf为：

```
{
  "network": {
    "servers": [ "192.168.1.1:5043"],
    "ssl ca": "/mnt/disk12/logger/logstash/config/keystore.jks"
  },
  "files": [
    {
      "paths": [ "/mnt/disk12/logger/logstash/config/2.txt" ],
      "fields": { "type": "sadb" }
    }
  ]
}
```

*注意：server可以配置多个，这样agent如果一个logstash连接不上可以连接另外的。*

其余配置信息，请参考logstash-forwarder，它完全兼容。主要包括下面几个可用配置项：

* network.servers: 用来指定远端(即 logstash indexer 角色)服务器的 IP 地址和端口。这里可以写数组，但是 logstash-forwarder 只会随机选一台作为对端发送数据，一直到对端故障，才会重选其他服务器。
* network.ssl\*: 网络交互中使用的 SSL 证书路径。
* files.\*.paths: 读取的文件路径。
  logstash-forwarder 只支持两种输入，一种就是示例中用的文件方式，和 logstash 一样也支持 glob 路径，即 `"/var/log/*.log"` 这样的写法；一种是标准输入，写法为 `"paths": [ "-" ]`
* files.\*.fields: 给每条日志添加的固定字段，相当于 logstash 里的 `add_field` 参数。

配置好以后启动它即可：`nohup java -jar logstash-forwarder-java-0.2.3-SNAPSHOT.jar -quiet -config logforwarder.conf > logforwarder.log &`

*-quiet 参数可以大大减少不必要的日志量，如果遇到错误请打开-debug和-trace选项，得到相关信息后查证，未果时请转向该项目github的issue区，作者很热心*

测试通过环境：

* AIX版本
> 6100-04-06-1034

* Java版本
> java version "1.6.0"
> Java(TM) SE Runtime Environment (build pap6460sr14-20130705_01(SR14))
> IBM J9 VM (build 2.4, JRE 1.6.0 IBM J9 2.4 AIX ppc64-64 jvmap6460sr14-20130704_155156 (JIT enabled, AOT enabled)
> J9VM - 20130704_155156
> JIT - r9_20130517_38390
> GC - GA24_Java6_SR14_20130704_1138_B155156)
> JCL - 20130618_01
