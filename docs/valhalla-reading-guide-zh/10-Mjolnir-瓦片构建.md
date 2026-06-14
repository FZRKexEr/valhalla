# Mjolnir 源码阅读指南
这一章解决的问题：带你用业务问题理解 mjolnir，再进入关键文件。
读完这一章，你应该能回答：mjolnir 负责什么、输入输出是什么、第一次读哪些文件。
建议先读：`valhalla/mjolnir/` 头文件，再读 `src/mjolnir/` 中同名实现。
不建议一开始读：边角参数、异常分支和性能微优化。

## 1. 这个模块解决什么问题
离线构建层：从 OSM PBF 等输入构建 Valhalla routing tiles。

这一步的业务意义是：把导航系统里的一个独立问题拆出来，避免 route 请求所有逻辑都堆在一个函数里。第一次读到这里，只要搞懂它和 Tyr/Loki/Thor/Odin/Mjolnir 之间的数据交接，不必马上读完所有类。

## 2. 它在整体链路里的位置
| 定位 | 说明 | 源码证据 |
|---|---|---|
| 模块职责 | 离线构建层：从 OSM PBF 等输入构建 Valhalla routing tiles。 | src/mjolnir/valhalla_build_tiles.cc:98 调 build_tile_set；src/mjolnir/util.cc:631 定义 build_tile_set。 |
| 代表测试 | 用测试确认模块不是“只被声明”。 | test/gurka/test_reproduce_tile_build.cc、test/gurka 构图工具。 |

## 3. 输入和输出
| 场景 | 输入 | 输出 | 关键类型/函数 | 源码证据 |
|---|---|---|---|---|
| 在线 route 相关 | `Api`、`Options`、locations、costing 或图数据 | 更新后的 `Api` / 路径 / 叙述 / 候选 | 见本模块 worker 或核心类型 | src/mjolnir/valhalla_build_tiles.cc:98 调 build_tile_set；src/mjolnir/util.cc:631 定义 build_tile_set。 |
| 扩展服务 | matrix、isochrone、trace 等 action | 对应 protobuf 字段或序列化结果 | worker action 函数 | `valhalla/tyr/actor.h` 声明多个 action |

## 4. 核心概念
### 模块边界
人话解释：先看这个模块“接收什么、产出什么”，不要先陷进内部算法。
对应源码：
- 文件：`valhalla/mjolnir/`、`src/mjolnir/`
- 类/结构体/函数：见证据列
- 相关测试：test/gurka/test_reproduce_tile_build.cc、test/gurka 构图工具。
为什么重要：模块边界决定你调试时应该在哪一层打断点。
常见误解：把目录名直接当成完整职责；本文只采用源码入口和测试能支撑的职责。

### 关键入口
人话解释：每个模块通常有一个最适合第一次阅读的入口文件。
对应源码：
- 文件：见下方推荐阅读顺序
- 类/结构体/函数：src/mjolnir/valhalla_build_tiles.cc:98 调 build_tile_set；src/mjolnir/util.cc:631 定义 build_tile_set。
- 相关测试：test/gurka/test_reproduce_tile_build.cc、test/gurka 构图工具。
为什么重要：入口能告诉你数据从哪里来、往哪里去。
常见误解：一上来读最底层工具类，结果看不出业务链路。

## 5. 关键流程
```text
Step 1: 从 Tyr 或构建命令进入模块。源码证据：src/mjolnir/valhalla_build_tiles.cc:98 调 build_tile_set；src/mjolnir/util.cc:631 定义 build_tile_set。
Step 2: 模块读取/修改 Api、GraphTile、DynamicCost 或构建期临时数据。
Step 3: 结果交给下游模块或写入 tile/response。
```

## 6. 推荐阅读顺序
| 顺序 | 文件 | 为什么先读它 | 读的时候关注什么 |
|---|---|---|---|
| 1 | `valhalla/mjolnir` | 先看公开类型 | 类名、输入输出 |
| 2 | `src/mjolnir` | 再看实现 | 主 action/worker |
| 3 | `test/gurka/test_reproduce_tile_build.cc、test/gurka 构图工具。` | 用测试验证理解 | 业务场景和边界条件 |
| 4 | `src/tyr/actor.cc` | 看谁调用它 | action 链路 |
| 5 | `proto/options.proto` | 看请求字段 | costing/action/locations |

## 7. 和其他模块的关系
| 上游模块 | 当前模块如何被调用 | 下游模块 | 当前模块调用它做什么 | 证据 |
|---|---|---|---|---|
| Tyr/构建 CLI | 在线请求由 actor 或 worker 编排；构建由 CLI 进入 | Baldr/Sif/Thor/Odin/Mjolnir 等 | 根据模块定位读下游 | src/mjolnir/valhalla_build_tiles.cc:98 调 build_tile_set；src/mjolnir/util.cc:631 定义 build_tile_set。 |

## 8. 读完本章应该能回答的问题
1. mjolnir 在 route 链路中是在线模块、离线模块还是工具模块？
2. 它的核心输入对象是什么？
3. 它的结果被谁消费？
4. 第一次调试应该在哪个文件打断点？
5. 哪些测试能帮助你确认理解？

## 9. 本章仍不确定的地方
TODO: NEED HUMAN CONFIRMATION：本章是第一轮静态阅读版，部分测试文件名和完整调用链需要后续按符号逐一补强。

## 本章小结
你可以先把 mjolnir 理解成：离线构建层：从 OSM PBF 等输入构建 Valhalla routing tiles。 先读入口，再读核心类型，最后读测试。

## 下一步读什么
读完本章后，回到 `03-一次route请求如何跑完.md`，把这个模块放回完整请求链路里。

## 自测问题
同第 8 节。

## 源码证据
- src/mjolnir/valhalla_build_tiles.cc:98 调 build_tile_set；src/mjolnir/util.cc:631 定义 build_tile_set。
- 代表测试：test/gurka/test_reproduce_tile_build.cc、test/gurka 构图工具。
