alias的几点应用
==================

*本节作者：childe*

# 索引更改名字时, 无缝过渡

## 情景1

用Logstash采集当前的所有nginx日志, 放入ES, 索引名叫nginx-YYYY.MM.DD.

后来又增加了apache日志, 希望能放在同一个索引里面,统一叫web-YYYY.MM.DD.

我们只要把Logstash配置更改一下,然后重启, 数据就会写入新的索引名字下. 但是同一天的索引就会被分成了2个, kibana上面就不好配置了.

### 如此实现

1. 今天是2015.07.28. 我们为nginx-2015.07.28建一个alias叫做web-2015.07.28, 之前的所有nginx日志也如此照做.  
2. kibana中把dashboard配置的索引名改成web-YYYY.MM.DD
3. 将logstash里面的elasticsearch的配置改成web-YYYY.MM.DD, 重启.
4. 无缝切换实现.

## 情景2

用Logstash采集当前的所有nginx日志, 放入ES, 索引名叫nginx-YYYY.MM.DD.

某天(2015.07.28)希望能够按月建立索引, 索引名改成nginx-YYYY.MM.

**[注意]** 像情景1中那样新建一个叫nginx-2015.07的alias, 并指向本月的其他的索引是不行的. 因为一个alias指向了多个索引, 写这个alias的时候, ES不可能知道写入哪个真正的索引.

### 如此实现

1.  新建索引nginx-2015.07, 以及他的alias: nginx-2015.07.29, nginx-2015.07.30  ... 等.
	新建索引nginx-2015.08, 以及他的alias: nginx-2015.08.01, nginx-2015.08.02  ... 等.
2. 等到第二天, 将logstash配置更改为nginx-YYYY.MM, 重启.
3. 如果索引只保留10天(一般来说, 不可能永久保存), 在10天之后的某天, 将kibana配置更改为 `[nginx-]YYYY.MM`

### 缺点

第二步, 第三步需要记得手工操作, 或者写一个crontab定时任务.

### 另外一种思路

1. 新建一个叫nginx-2015.07的alias, 并指向本月的其他的索引. 
	这个时候不能马上更改logstash配置写入nginx-2015.07. 因为一个alias指向了多个索引, 写这个alias的时候, ES不可能知道写入哪个真正的索引.
2. 新建索引nginx-2015.08, 以及他的alias: nginx-2015.08.01, nginx-2015.08.02  ... 等.
3. 把kibana配置改为 `[nginx-]YYYY.MM`. 
4. 到2015.08.01这天, 或者之后的某天, 更改logstash配置, 把elasticsearch的配置更改为nginx-YYYY.MM. 重启.

### 缺点

1. 7月份的索引还是按天建立.
2. 第四步需要记得手工操作.

情景2的操作有些麻烦, 这些都是为了"无缝"这个大前提.  我们不希望用户(kibana的使用者)感觉到任何变化, 更不能让使用者感受到数据有缺失.

如果用户愿意妥协, 那ELK的管理者就可以马上简单粗暴的把数据写到nginx-2015.07, 并把kibana配置改过去. 这样一来, 用户可能就不能方便的看到之前的数据了.

# 按域名配置kibana的dashboard

nginx日志中有个字段是domain, 各个业务部门在Kibana中需要看且只看自己域名下的日志. 

可以在logstash中按域名切分索引,但ES的metadata会成倍的增长,带来极大的负担.

也可以放在nginx-YYYY.MM.DD一个索引中, 在kibana加一个filter过滤. 但这个关键的filter和其它所有filter排在一起,是很容易很不小心清掉的.

我们可以在template中配置, 对每一个域名做一个alias. 

```
"aliases" : {
      "{index}-www.corp.com" : {
        "filter" : {
          "term" : {
            "domain" : "www.corp.com"
          }
        }
      },
      "{index}-abc.corp.com" : {
        "filter" : {
          "term" : {
            "domain" : "abc.corp.com"
          }
        }
      }
}
```

当新的索引nginx-2015.07.28生成的时候, 会有nginx-2015.07.28-www.corp.com和nginx-2015.07.28-abc.corp.com等alias指向nginx-2015.07.28.

在kibana配置dashboard的时候, 就可以直接用alias做配置了.

在实际使用中, 可能还需要一个crontab定时的查询是否有新的域名加入, 自动对新域名做当天的alias, 并把它加入template.
