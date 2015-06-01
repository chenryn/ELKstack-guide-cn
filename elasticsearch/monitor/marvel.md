# marvel

marvel 是 Elastic.co 公司推出的官方监控方案，在非商业场合是可以免费使用的。如果在商业生产上使用，Elastic.co 公司的收费标准是：

* 前 5 个节点，每年 1000 美元
* 之后每增加 5 个节点，每年加收 250 美元

## 安装

marvel 是以 elasticsearch 的插件形式存在的，直接通过插件安装即可：

```
./bin/plugin -i elasticsearch/marvel/latest
```
