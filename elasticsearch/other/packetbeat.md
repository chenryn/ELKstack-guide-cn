# packetbeat

之前已经提到过，packetbeat 已经被 Elastic 公司收归旗下，未来就会替代 logstash-forwarder 成为 ELKstack 套件的一部分。那么，为什么 Elastic 公司如此看好 packetbeat 项目，不惜废掉 logstash-forwarder 呢？

packetbeat 和 logstash-forwarder 一样，也是一个 golang 写的开源项目。其特色在于，logstash-forwarder 的数据来源是文件或者标准输入，都是文本流。而packetbeat 采用 libpcap 库，抓取网络流量，识别其中的特定网络协议，自动按照协议规范，将网络流量包划分成事件字段，写入到 Elasticsearch 中。

对于很多 ELKstack 新手来说，面对的很可能就是几种常用数据流，而书写 logstash 正则是一个耗时耗力的重复劳动，文件落地本身又是多余操作，packetbeat 的运行方式，无疑是对新手入门极大的帮助。

目前 packetbeat 项目地址为：<https://github.com/elastic/packetbeat>。

## 安装部署

## 配置示例

## 效果

