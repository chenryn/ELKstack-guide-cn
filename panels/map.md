# map

状态：稳定

map 面板把 2 个字母的国家或地区代码转成地图上的阴影区域。目前可用的地图包括世界地图，美国地图和欧洲地图。

## 参数

* map
    显示哪个地图：world, usa, europe
* colors
    用来涂抹地图阴影的颜色数组。一旦设定好这 2 个颜色，阴影就会使用介于这 2 者之间的颜色。示例 [‘#A0E2E2’, ‘#265656’]
* size
    阴影区域的最大数量
* exclude
    排除的区域数组。示例 [‘US’,‘BR’,‘IN’]
* spyable
    设为假，不显示审查(inspect)按钮。

**请求(queries)**

* 请求对象
    这个对象描述本面板使用的请求。
  * queries.mode
    在可用请求中应该用哪些？可设选项有：`all, pinned, unpinned, selected`
  * queries.ids
    如果设为 `selected` 模式，具体被选的请求编号。
