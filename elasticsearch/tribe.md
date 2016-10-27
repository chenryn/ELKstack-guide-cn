# 多集群连接

当你的 ES 集群发展到一定规模，单集群不足以应对庞大的在线索引量级，或者由于业务隔离需求，都有可能划分成多个集群。这时候，另一个问题就出来了：可能其中有一部分数据，被分割在两个集群里，但是还是需要一起使用的。如果是自己写程序，当然可以初始化两个对象，分别连接两个集群，得到结果集后再自行合并。但是如果用 Elastic Stack 的，Kibana 可不支持同时连接两个集群地址，这时候，就要用到 ES 中一个特殊的角色：tribe 节点。

tribe 节点只需要提供集群自动发现方面的配置，连接上多个集群后，对外提供只读功能。`elasticsearch.yml` 配置示例如下：

```
tribe:
    1002:
        cluster.name: es1002
        discovery.zen.ping.timeout: 100s
        discovery.zen.ping.multicast.enabled: false
        discovery.zen.ping.unicast.hosts: ["10.19.0.22","10.19.0.24",10.19.0.21"]
    1003:
        cluster.name: es1003
        discovery.zen.ping.timeout: 100s
        discovery.zen.ping.multicast.enabled: false
        discovery.zen.ping.unicast.hosts: ["10.19.0.97","10.19.0.98","10.19.0.99","10.19.0.100"]
    blocks:
        write:    true
        metadata: true
    on_conflict: prefer_1003
```

注意这里的 `on_conflict` 设置，当多个集群内，索引名称有冲突的时候，tribe 节点默认会把请求轮询转发到各个集群上，这显然是不可以的。所以可以设置一个优先级，在索引名冲突的时候，偏向于转发给某一个集群。

以 tribe 配置启动的 Elasticsearch 服务，其日志输入如下：

```
[2015-06-18 18:05:51,983][INFO ][node                     ] [Manslaughter] version[1.5.1], pid[12846], build[5e38401/2015-04-09T13:41:35Z]
[2015-06-18 18:05:51,984][INFO ][node                     ] [Manslaughter] initializing ...
[2015-06-18 18:05:51,990][INFO ][plugins                  ] [Manslaughter] loaded [], sites []
[2015-06-18 18:05:54,891][INFO ][node                     ] [Manslaughter/1003] version[1.5.1], pid[12846], build[5e38401/2015-04-09T13:41:35Z]
[2015-06-18 18:05:54,891][INFO ][node                     ] [Manslaughter/1003] initializing ...
[2015-06-18 18:05:54,891][INFO ][plugins                  ] [Manslaughter/1003] loaded [], sites []
[2015-06-18 18:05:55,654][INFO ][node                     ] [Manslaughter/1003] initialized
[2015-06-18 18:05:55,655][INFO ][node                     ] [Manslaughter/1002] version[1.5.1], pid[12846], build[5e38401/2015-04-09T13:41:35Z]
[2015-06-18 18:05:55,655][INFO ][node                     ] [Manslaughter/1002] initializing ...
[2015-06-18 18:05:55,656][INFO ][plugins                  ] [Manslaughter/1002] loaded [], sites []
[2015-06-18 18:05:56,275][INFO ][node                     ] [Manslaughter/1002] initialized
[2015-06-18 18:05:56,285][INFO ][node                     ] [Manslaughter] initialized
[2015-06-18 18:05:56,286][INFO ][node                     ] [Manslaughter] starting ...
[2015-06-18 18:05:56,486][INFO ][transport                ] [Manslaughter] bound_address {inet[/0:0:0:0:0:0:0:0:9301]}, publish_address {inet[/10.19.0.100:9301]}
[2015-06-18 18:05:56,499][INFO ][discovery                ] [Manslaughter] elasticsearch/Oewo-L2fR3y2xsgpsoI4Og
[2015-06-18 18:05:56,499][WARN ][discovery                ] [Manslaughter] waited for 0s and no initial state was set by the discovery
[2015-06-18 18:05:56,529][INFO ][http                     ] [Manslaughter] bound_address {inet[/0:0:0:0:0:0:0:0:9201]}, publish_address {inet[/10.19.0.100:9201]}
[2015-06-18 18:05:56,530][INFO ][node                     ] [Manslaughter/1003] starting ...
[2015-06-18 18:05:56,603][INFO ][transport                ] [Manslaughter/1003] bound_address {inet[/0:0:0:0:0:0:0:0:9302]}, publish_address {inet[/10.19.0.100:9302]}
[2015-06-18 18:05:56,609][INFO ][discovery                ] [Manslaughter/1003] es1003/m1-cDaFTSoqqyC2iiQhECA
[2015-06-18 18:06:26,610][WARN ][discovery                ] [Manslaughter/1003] waited for 30s and no initial state was set by the discovery
[2015-06-18 18:06:26,610][INFO ][node                     ] [Manslaughter/1003] started
[2015-06-18 18:06:26,611][INFO ][node                     ] [Manslaughter/1002] starting ...
[2015-06-18 18:06:26,674][INFO ][transport                ] [Manslaughter/1002] bound_address {inet[/0:0:0:0:0:0:0:0:9303]}, publish_address {inet[/10.19.0.100:9303]}
[2015-06-18 18:06:26,676][INFO ][discovery                ] [Manslaughter/1002] es1002/4FPiRPh7TFyBk-BaPc_TLg
[2015-06-18 18:06:56,676][WARN ][discovery                ] [Manslaughter/1002] waited for 30s and no initial state was set by the discovery
[2015-06-18 18:06:56,677][INFO ][node                     ] [Manslaughter/1002] started
[2015-06-18 18:06:56,677][INFO ][node                     ] [Manslaughter] started
[2015-06-18 18:07:37,266][INFO ][cluster.service          ] [Manslaughter/1003] detected_master [10.19.0.97][jnA-rt2fS_22Mz9nYl5Ueg][localhost.localdomain][inet[/10.19.0.97:9300]]{max_local_storage_nodes=1, data=false, master=true}, added {[10.19.0.73][_S8ylz1OTv6Nyp1YoMRNGQ][esnode073.mweibo.bx.sinanode.com][inet[/10.19.0.73:9300]]{max_local_storage_nodes=1, master=false},}, reason: zen-disco-receive(from master [[10.19.0.97][jnA-rt2fS_22Mz9nYl5Ueg][localhost.localdomain][inet[/10.19.0.97:9300]]{max_local_storage_nodes=1, data=false, master=true}])
[2015-06-18 18:07:37,382][INFO ][tribe                    ] [Manslaughter] [1003] adding node [[10.19.0.73][_S8ylz1OTv6Nyp1YoMRNGQ][esnode073.mweibo.bx.sinanode.com][inet[/10.19.0.73:9300]]{max_local_storage_nodes=1, tribe.name=1003, master=false}]
[2015-06-18 18:07:37,391][INFO ][tribe                    ] [Manslaughter] [1003] adding node [[Manslaughter/1003][m1-cDaFTSoqqyC2iiQhECA][localhost.localdomain][inet[/10.19.0.100:9302]]{data=false, tribe.name=1003, client=true}]
[2015-06-18 18:07:37,393][INFO ][tribe                    ] [Manslaughter] [1003] adding node [[10.19.0.97][_mIrWKzZTYifp1xshngBew][esnode054.mweibo.bx.sinanode.com][inet[/10.19.0.54:9300]]{max_local_storage_nodes=1, tribe.name=1003, master=false}]
[2015-06-18 18:07:37,393][INFO ][tribe                    ] [Manslaughter] [1003] adding index [logstash-mweibo-vip-2015.06.15]
[2015-06-18 18:07:37,394][INFO ][tribe                    ] [Manslaughter] [1003] adding index [logstash-php-2015.06.08]
[2015-06-18 18:07:37,394][INFO ][tribe                    ] [Manslaughter] [1003] adding index [logstash-mweibo-vip-2015.06.16]
[2015-06-18 18:07:37,395][INFO ][tribe                    ] [Manslaughter] [1003] adding index [.kibana]
[2015-06-18 18:07:37,398][INFO ][tribe                    ] [Manslaughter] [1003] adding index [logstash-php-2015.06.14]
[2015-06-18 18:07:37,403][INFO ][tribe                    ] [Manslaughter] [1003] adding index [logstash-mweibo-vip-2015.06.10]
[2015-06-18 18:07:37,403][INFO ][tribe                    ] [Manslaughter] [1003] adding index [kibana-int]
[2015-06-18 18:07:37,404][INFO ][tribe                    ] [Manslaughter] [1003] adding index [logstash-mweibo-2015.06.13]
[2015-06-18 18:07:37,411][INFO ][cluster.service          ] [Manslaughter] added {[10.19.0.73][_S8ylz1OTv6Nyp1YoMRNGQ][esnode073.mweibo.bx.sinanode.com][inet[/10.19.0.73:9300]]{max_local_storage_nodes=1, tribe.name=1003, master=false},[10.19.0.97][jnA-rt2fS_22Mz9nYl5Ueg][localhost.localdomain][inet[/10.19.0.97:9300]]{max_local_storage_nodes=1, tribe.name=1003, data=false, master=true},}, reason: cluster event from 1003, zen-disco-receive(from master [[10.19.0.97][jnA-rt2fS_22Mz9nYl5Ueg][localhost.localdomain][inet[/10.19.0.97:9300]]{max_local_storage_nodes=1, data=false, master=true}])
[2015-06-18 18:08:07,316][INFO ][cluster.service          ] [Manslaughter/1002] detected_master [10.19.0.22][6qyQh9EURUyO7RBC_dXDow][localhost.localdomain][inet[/10.19.0.22:9300]]{max_local_storage_nodes=1, master=true}, added {[10.19.0.93][qAklY08iSsSfIf2vvu6Iyw][localhost.localdomain][inet[/10.19.0.93:9300]]{max_local_storage_nodes=1, master=false}])
[2015-06-18 18:08:07,350][INFO ][indices.breaker          ] [Manslaughter/1002] Updating settings parent: [PARENT,type=PARENT,limit=259489792/247.4mb,overhead=1.0], fielddata: [FIELDDATA,type=MEMORY,limit=155693875/148.4mb,overhead=1.03], request: [REQUEST,type=MEMORY,limit=103795916/98.9mb,overhead=1.0]
[2015-06-18 18:08:07,353][INFO ][tribe                    ] [Manslaughter] [1002] adding node [[10.19.0.93][qAklY08iSsSfIf2vvu6Iyw][localhost.localdomain][inet[/10.19.0.93:9300]]{max_local_storage_nodes=1, tribe.name=1002, master=false}]
[2015-06-18 18:08:07,357][INFO ][tribe                    ] [Manslaughter] [1002] adding node [[Manslaughter/1002][4FPiRPh7TFyBk-BaPc_TLg][localhost.localdomain][inet[/10.19.0.100:9303]]{data=false, tribe.name=1002, client=true}]
[2015-06-18 18:08:07,358][INFO ][tribe                    ] [Manslaughter] [1002] adding node [[10.19.0.22][tkrBsbnLTry0zzZEdbQR0A][localhost.localdomain][inet[/10.19.0.27:9300]]{max_local_storage_nodes=1, tribe.name=1002, master=false}]
[2015-06-18 18:08:07,358][INFO ][tribe                    ] [Manslaughter] [1002] adding index [test.yingju1-mweibo_client_downstream_success-2015.06.07]
[2015-06-18 18:08:07,363][INFO ][tribe                    ] [Manslaughter] [1002] adding index [logstash-mweibo_client_downstream_error-2015.06.02]
[2015-06-18 18:08:07,366][INFO ][tribe                    ] [Manslaughter] [1002] adding index [.kibana_5601]
[2015-06-18 18:08:07,377][INFO ][cluster.service          ] [Manslaughter] added {[10.19.0.22][6qyQh9EURUyO7RBC_dXDow][localhost.localdomain][inet[/10.19.0.22:9300]]{max_local_storage_nodes=1, tribe.name=1002, master=false},[10.19.0.93][l7nkk-H7S6GvMzWwGe0_CA][localhost.localdomain][inet[/10.19.0.93:9300]]{max_local_storage_nodes=1, tribe.name=1002, master=false},}, reason: cluster event from 1002, zen-disco-receive(from master [[10.19.0.22][6qyQh9EURUyO7RBC_dXDow][localhost.localdomain][inet[/10.19.0.22:9300]]{max_local_storage_nodes=1, master=true}])
[2015-06-18 18:08:13,208][DEBUG][discovery.zen.publish    ] [Manslaughter/1003] received cluster state version 782404
[2015-06-18 18:08:21,803][DEBUG][discovery.zen.publish    ] [Manslaughter/1003] received cluster state version 782405
[2015-06-18 18:08:33,229][DEBUG][discovery.zen.publish    ] [Manslaughter/1003] received cluster state version 782406
```

日志中可以明显看到，节点是如何分别连接上两个集群的。

最后，我们可以使用标准的 RESTful 接口来验证一下：

```
# curl 10.19.0.100:9201/_cat/indices?v
health status index                                                    pri rep docs.count docs.deleted store.size pri.store.size
green  open   test.yingju1-mweibo_client_downstream_success-2015.06.07  20   1   40692459            0    154.1gb           77gb
green  open   weibo-client-video-2015.06.19                              5   1          0            0       970b           575b
green  open   dpool-pc-weibo-2015.06.19                                 20   1          0            0      3.7kb          2.2kb
green  open   logstash-video-2015.06.16                                 27   0  149015413            0     13.4gb         13.4gb
```

不同集群的索引，都可以通过 tribe node 访问到了。
