# Valhalla核心概念词典
这一章解决的问题：第一次遇到术语时有一个人话入口。
读完这一章，你应该能回答：Api、Options、GraphTile、costing、candidate edge 分别是什么。
建议先读：`proto/api.proto`、`proto/options.proto`、`valhalla/baldr/graphtile.h`、`valhalla/sif/dynamiccost.h`。
不建议一开始读：所有字段枚举。

| 术语 | 人话解释 | 对应源码 | 为什么重要 |
|---|---|---|---|
| `Api` | 一次请求在模块间传递的总包裹，后续也会装响应。 | `src/tyr/actor.cc:105-118` | 串起 Loki/Thor/Odin。 |
| `Options` | 请求选项：action、locations、costing、format 等。 | `src/tyr/actor.cc:110` | 决定走 route 还是 matrix。 |
| `GraphTile` | 路网图的一块瓦片，在线按需读取。 | `valhalla/baldr/graphtile.h:140` | Loki/Thor 都依赖图数据。 |
| `GraphReader` | 读取和缓存 GraphTile 的对象。 | `valhalla/baldr/graphreader.h:436` | 避免算法直接管文件。 |
| `DynamicCost` | 不同交通方式如何给边/转向算代价的接口。 | `valhalla/sif/dynamiccost.h:250` | 决定 ETA 和选路偏好。 |
| candidate edge | 输入坐标附近可选的路网边。 | `src/loki/route_action.cc:51` 进入位置关联流程 | 决定起终点落在哪。 |
| maneuver | 给用户看的导航动作。 | `valhalla/odin/maneuver.h` | Odin 的核心输出之一。 |
| tile build | 从 OSM PBF 等源数据构建 routing tiles。 | `src/mjolnir/util.cc:631` | 没有 tiles 就无法在线 route。 |
| map matching | 把 GPS 轨迹匹配到路网上。 | `valhalla/meili/map_matcher.h` | trace_route/trace_attributes 相关。 |

## 本章小结
先用这些词建立心智模型，再去读模块文档。
## 下一步读什么
读 `03-一次route请求如何跑完.md`。
## 自测问题
1. `Api` 和 `Options` 有什么区别？
2. `GraphTile` 和 `GraphTileBuilder` 有什么区别？
3. costing 为什么会影响路线？
## 源码证据
见表格。
