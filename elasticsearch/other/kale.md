# Kale 系统

Kale 系统是 Etsy 公司开源的一个监控分析系统。Kale 分为两个部分：skyline 和 oculus。skyline 负责对时序数据进行概率分布校验，对校验失败率超过阈值的时序数据发报警；oculus 负责给被报警的时序，找出趋势相似的其他时序作为关联性参考。 

看到“相似”两个字，你一定想到了。没错，oculus 组件，就是利用了 Elasticsearch 的相似度打分。

oculus 中，为 Elasticsearch 的 `org.elasticsearch.script.ExecutableScript` 扩展了 `DTW` 和 `Euclidian` 两种 NativeScript。可以在界面上选择用其中某一种算法来做相似度打分：

![](https://codeascraft.com/wp-content/uploads/2013/06/results_screenshot.jpeg?w=300)

然后相似度最高的几个时序图就依次排列出来了。

Euclidian 即欧几里得距离，是时序相似度计算里最基础的方式。

DWT 即动态时间规整(Dynamic Time Warping)，也是时序相似度计算的常用方式，它和欧几里得距离的差别在于，欧几里得距离要求比对的时序数据是一一对应的，而动态时间规整计算的时序数据并不要求长度相等。在运维监控来说，也就是延后一定时间发生的相近趋势也可以以很高的打分项排名靠前。

不过，oculus 插件仅更新到支持 Elasticsearch-0.90.3 版本为止。Etsy 性能优化团队在 Oreilly 2015 大会上透露，他们内部已经根据 Kale 的经验教训，重新开发了 Kale 2.0 版。会在年内开源放出来。大家一起期待吧！

## 参考阅读

<http://codeascraft.com/2013/06/11/introducing-kale/>
