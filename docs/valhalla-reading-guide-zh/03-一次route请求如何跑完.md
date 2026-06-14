# 一次 route 请求如何在 Valhalla 里跑完
这一章解决的问题：用一条真实源码调用链理解 route。
读完这一章，你应该能回答：用户坐标如何变成导航结果。
建议先读：`src/tyr/actor.cc`、`src/loki/route_action.cc`、`src/thor/route_action.cc`、`src/odin/worker.cc`。
不建议一开始读：所有 A* 队列细节和所有 serializer 字段。

## 1. 先用人话讲完整链路
用户给 Valhalla 一组 `locations`、一个 `costing` 和输出格式。Valhalla 先把 JSON/PBF 请求解析成内部 `Api`；然后 Loki 检查请求并把经纬度落到路网候选边；Thor 根据这些候选边和 Sif 的代价模型搜索路径；Odin 把路径变成 maneuver 和文字指令；最后 Tyr serializer 返回 JSON 或 PBF。这个顺序由 `actor_t::route` 直接证明：ParseApi -> `loki_worker.route` -> `thor_worker.route` -> `odin_worker.narrate`。源码证据：`src/tyr/actor.cc:97-118`。

## 2. 文字版时序图
```text
Client
  -> Tyr actor_t::route: 解析 request_str 到 Api
  -> Loki loki_worker_t::route: 校验 locations/costing，做位置关联准备
  -> Thor thor_worker_t::route: 调整 locations，解析 costing，按 depart/arrive 搜索路径
  -> Sif parse_costing/DynamicCost: 给 Thor 搜索提供边和转向代价
  -> Odin odin_worker_t::narrate: DirectionsBuilder 生成 directions
  -> Tyr serializeDirections: 按格式序列化 route 结果
```

## 3. 源码调用链
| 顺序 | 文件 | 函数/类 | 做什么 | 输入 | 输出 | 证据 |
|---|---|---|---|---|---|---|
| 1 | `src/tyr/actor.cc` | `actor_t::route` | route 总编排 | request string | `Api` 和 bytes | `src/tyr/actor.cc:97-118` |
| 2 | `src/tyr/actor.cc` | `ParseApi` | 解析请求并设置 `Options::route` | JSON/PBF | `Api` | `src/tyr/actor.cc:110` |
| 3 | `src/loki/route_action.cc` | `loki_worker_t::route` | 校验并定位请求位置 | `Api` | 带候选的 `Api` | `src/loki/route_action.cc:51` |
| 4 | `src/thor/route_action.cc` | `thor_worker_t::route` | 搜索路径 | 带候选的 `Api` | route/trip path | `src/thor/route_action.cc:364-383` |
| 5 | `valhalla/sif/dynamiccost.h` | `DynamicCost` | 抽象交通方式代价 | graph edge/transition | cost | `valhalla/sif/dynamiccost.h:250` |
| 6 | `src/odin/worker.cc` | `odin_worker_t::narrate` | 生成 directions 并序列化 | 带路径的 `Api` | JSON/PBF 字节 | `src/odin/worker.cc:28-38` |

## 4. 关键对象在链路中如何流动
| 对象 | 在哪里创建 | 被谁消费 | 表示什么 | 证据 |
|---|---|---|---|---|
| `Api` | `actor_t::route` 中 dummy 或外部传入 | Loki/Thor/Odin/serializer | 一次请求和响应的总容器 | `src/tyr/actor.cc:105-118` |
| `Options` | `ParseApi` 写入 | Loki/Thor/serializer | action、locations、costing、format | `src/tyr/actor.cc:110` |
| costing | Thor 中 `parse_costing` | path algorithm | 交通方式代价模型 | `src/thor/route_action.cc:373` |
| GraphTile | Mjolnir 离线构建，Baldr 运行时读取 | Loki/Thor/Sif | 路网图数据分片 | `valhalla/baldr/graphtile.h:140` |

## 5. 第一次读 route 流程应该按什么顺序读代码
| 顺序 | 文件 | 为什么读 | 读到什么程度可以停 |
|---|---|---|---|
| 1 | `src/tyr/actor.cc` | 找到主调用链 | 能复述四步调用即可 |
| 2 | `valhalla/tyr/actor.h` | 看有哪些 action | 知道 route/matrix/isochrone/trace 是同一 actor 面 |
| 3 | `src/loki/route_action.cc` | 看坐标进入图前的处理 | 看到校验和 init_route 即可 |
| 4 | `src/thor/route_action.cc` | 看搜索入口 | 看到 depart/arrive 分支即可 |
| 5 | `valhalla/sif/dynamiccost.h` | 理解 costing 接口 | 关注边代价和转向代价 |
| 6 | `src/odin/worker.cc` | 看输出如何产生 | 理解 Build + serializeDirections |

## 6. 容易误解的点
1. Loki 不是路径搜索，它主要在路径搜索前处理 locations。
2. Sif 不是独立请求 worker，而是 Thor 搜索时的代价模型来源。
3. Odin 不重新找路，它消费 Thor 产生的路径。
4. GraphTile 通常不是 route 请求现场构建的，而是 Mjolnir 离线构建、Baldr 在线读取。
5. `Api` 既承载请求也逐步承载响应，不要把它理解成只读输入。

## 7. 自测问题
1. `actor_t::route` 中哪个调用负责解析请求？
2. Loki 之后 `Api` 应该多出哪类信息？
3. Thor 为什么要调用 `parse_costing`？
4. arrive_by 和 depart_at 在哪里分支？
5. Odin 输出前调用哪个 builder？
6. serializer 属于 Odin 还是 Tyr？
7. GraphTile 从哪里来？
8. 如果 route 返回路径不合理，应该先在 Loki、Thor 还是 Odin 打断点？

## 本章小结
route 主线先不要读散：Tyr 编排，Loki 关联，Thor 搜索，Sif 定价，Odin 讲路，Tyr 序列化。

## 下一步读什么
读 Loki、Thor、Sif 三章；它们决定“落到哪条边”和“为什么选这条路”。

## 源码证据
- `src/tyr/actor.cc:97-118`
- `src/loki/route_action.cc:51`
- `src/thor/route_action.cc:364-383`
- `src/odin/worker.cc:28-38`
- `valhalla/sif/dynamiccost.h:250`
