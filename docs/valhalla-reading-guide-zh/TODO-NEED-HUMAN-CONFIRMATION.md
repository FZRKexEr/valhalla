| 位置 | 不确定内容 | 为什么不确定 | 建议人工如何确认 |
|---|---|---|---|
| 多个模块文档 | 测试文件列表是否穷尽 | 本轮以代表性测试为主 | `rg "模块核心类|worker方法" test` 全量确认 |
| `01-整体架构地图.md` | HTTP 服务启动完整线程模型 | 本轮聚焦 actor in-process 链路 | 阅读 `src/tyr` 服务启动和 prime_server 配置 |
| Meili 章节 | trace_route 与 Meili/Thor map matcher 的完整边界 | 本轮只确认头文件和测试入口 | 追踪 `src/thor/trace_route_action.cc` 到 meili 调用 |
