# Evidence Index

## 仓库结构
| 路径 | 作用 | 证据 |
|---|---|---|
| `src/` | C++ 实现目录，按模块分子目录。 | `src/CMakeLists.txt` 将 tyr/odin/thor/loki 等源文件加入构建。 |
| `valhalla/` | 对外头文件目录，按模块分子目录。 | 例如 `valhalla/tyr/actor.h:14` 定义 actor_t，`valhalla/thor/worker.h:35` 定义 thor_worker_t。 |
| `test/` | C++、gurka、binding 测试。 | `test/actor.cc:24` 调 actor.route；`test/gurka/test_matrix.cc` 覆盖 matrix。 |
| `proto/` | Protobuf API/数据对象定义。 | `proto/options.proto`、`proto/api.proto` 定义请求对象。 |
| `docs/` | 用户文档，不等同源码阅读路线。 | `docs/docs` 下是站点文档。 |
| `CMakeLists.txt` | 顶层构建入口。 | 顶层包含项目和子目录配置。 |

## 主要模块
| 模块 | 初步职责 | 源码目录 | 头文件目录 | 测试目录 | 证据 |
|---|---|---|---|---|---|
| baldr | 基础数据层：GraphTile、GraphReader、GraphId 等运行时图数据访问。 | `src/baldr` | `valhalla/baldr` | test/graphtile.cc、test/graphreader.cc（若存在以仓库为准）；大量 gurka 测试间接覆盖。 | valhalla/baldr/graphtile.h:140 定义 GraphTile；valhalla/baldr/graphreader.h:436 定义 GraphReader。 |
| sif | 算法支撑层：DynamicCost 把不同交通方式的业务偏好转成边/转向代价。 | `src/sif` | `valhalla/sif` | test/costing.cc、test/gurka/test_*.cc 中按 costing 调 actor.route。 | valhalla/sif/dynamiccost.h:250 定义 DynamicCost；src/thor/route_action.cc:373 调用 parse_costing。 |
| loki | 位置关联层：检查请求，把经纬度关联到路网候选边。 | `src/loki` | `valhalla/loki` | test/worker_nullptr_tiles.cc、test/gurka locate/route 测试间接覆盖。 | valhalla/loki/worker.h:38 声明 route；src/loki/route_action.cc:51 实现 loki_worker_t::route。 |
| thor | 路径算法层：基于 Loki 的候选点和 Sif 的 costing 计算 route/matrix/isochrone/map matching 路径。 | `src/thor` | `valhalla/thor` | test/astar.cc、test/isochrone.cc、test/gurka/test_matrix.cc、test/mapmatch.cc。 | valhalla/thor/worker.h:54 声明 route/matrix/trace_route；src/thor/route_action.cc:364 实现 route。 |
| odin | 结果生成层：把 Thor 的 TripPath/TripLeg 转为 maneuver 和导航叙述。 | `src/odin` | `valhalla/odin` | test/gurka/test_instructions*.cc、test/narrative*.cc（若存在以仓库为准）。 | valhalla/odin/worker.h:16 定义 odin_worker_t；src/odin/worker.cc:28 narrate 调 DirectionsBuilder 并序列化。 |
| tyr | 请求编排层：actor_t 解析请求，顺序调用 Loki/Thor/Odin，并序列化结果。 | `src/tyr` | `valhalla/tyr` | test/actor.cc、test/bindings/python/test_actor.py、test/bindings/nodejs/tests.js。 | valhalla/tyr/actor.h:14 定义 actor_t；src/tyr/actor.cc:97 route 按 ParseApi->loki->thor->odin。 |
| mjolnir | 离线构建层：从 OSM PBF 等输入构建 Valhalla routing tiles。 | `src/mjolnir` | `valhalla/mjolnir` | test/gurka/test_reproduce_tile_build.cc、test/gurka 构图工具。 | src/mjolnir/valhalla_build_tiles.cc:98 调 build_tile_set；src/mjolnir/util.cc:631 定义 build_tile_set。 |
| meili | 地图匹配算法层：把 GPS trace 通过候选搜索、Viterbi、转移代价匹配到路网。 | `src/meili` | `valhalla/meili` | test/mapmatch.cc。 | valhalla/meili/map_matcher.h:19 定义 meili 命名空间 MapMatcher；valhalla/meili/viterbi_search.h:13 定义 Viterbi。 |
| midgard | 基础工具层：点、线、距离、格网等几何和通用工具。 | `src/midgard` | `valhalla/midgard` | test/point2.cc、test/encoded.cc（若存在以仓库为准）。 | valhalla/midgard/point2.h 与 valhalla/midgard/encoded.h 提供几何点和编码工具。 |

## 主要入口
| 入口类型 | 文件 | 函数/类 | 说明 | 证据 |
|---|---|---|---|---|
| in-process API | `src/tyr/actor.cc` | `actor_t::route` | 解析 route 请求并串起 loki/thor/odin。 | `src/tyr/actor.cc:97-118` |
| tile build CLI | `src/mjolnir/valhalla_build_tiles.cc` | `main` | 调用 `build_tile_set` 构建 tiles。 | `src/mjolnir/valhalla_build_tiles.cc:98` |
| Loki worker | `src/loki/route_action.cc` | `loki_worker_t::route` | 请求校验和位置关联入口。 | `src/loki/route_action.cc:51` |
| Thor worker | `src/thor/route_action.cc` | `thor_worker_t::route` | route 搜索入口。 | `src/thor/route_action.cc:364` |
| Odin worker | `src/odin/worker.cc` | `odin_worker_t::narrate` | 生成 directions 并序列化。 | `src/odin/worker.cc:28` |

## 关键数据结构
| 名称 | 所属模块 | 文件 | 初步解释 | 证据 |
|---|---|---|---|---|
| `Api` | proto | `proto/api.proto` | 一次请求在模块之间传递的总对象。 | `src/tyr/actor.cc:97` 参数和 ParseApi 使用。 |
| `Options` | proto | `proto/options.proto` | action、costing、locations 等请求选项。 | `src/tyr/actor.cc:110` ParseApi 写入 Options::route。 |
| `GraphTile` | baldr | `valhalla/baldr/graphtile.h` | 一个图瓦片的运行时视图。 | `valhalla/baldr/graphtile.h:140` |
| `GraphReader` | baldr | `valhalla/baldr/graphreader.h` | 按 GraphId/TileId 读取和缓存 GraphTile。 | `valhalla/baldr/graphreader.h:436` |
| `DynamicCost` | sif | `valhalla/sif/dynamiccost.h` | 交通方式代价模型基类。 | `valhalla/sif/dynamiccost.h:250` |
| `GraphTileBuilder` | mjolnir | `valhalla/mjolnir/graphtilebuilder.h` | 构建期可写 tile。 | `valhalla/mjolnir/graphtilebuilder.h:39` |

## 不确定项
| 问题 | 为什么不确定 | 下一步怎么验证 |
|---|---|---|
| HTTP 服务进程入口的完整线程模型 | 本轮优先确认 actor 链路，未完整展开 server/prime_server 配置。 | 继续读 `src/tyr/` 中服务启动相关文件和部署文档。 |
| 部分模块测试文件名 | 仓库测试很多，本文档先列代表性测试，未保证穷尽。 | 用 `rg` 按类名和 action 扫描全部 `test/`。 |
