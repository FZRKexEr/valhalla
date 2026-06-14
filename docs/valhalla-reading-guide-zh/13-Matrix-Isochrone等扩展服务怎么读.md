# Matrix-Isochrone等扩展服务怎么读
这一章解决的问题：route 之外的服务应该如何从主链路迁移阅读。
读完这一章，你应该能回答：matrix、isochrone、trace 与 route 共享哪些层。
建议先读：`valhalla/tyr/actor.h`、`src/tyr/actor.cc`、`src/thor/matrix_action.cc`、`src/thor/isochrone_action.cc`。
不建议一开始读：序列化输出的每个 JSON 字段。

## 服务入口地图
`actor_t` 公开 `route`、`matrix`、`optimized_route`、`isochrone`、`trace_route` 等方法，说明扩展服务共享 Tyr 入口面。源码证据：`valhalla/tyr/actor.h:58-128`。Matrix 在 Thor 有 `matrix(Api&)`，Isochrone 在 Thor 有 action serializer；测试中 `test/gurka/test_matrix.cc` 和 `test/isochrone.cc` 直接覆盖这些业务。

## 阅读方法
1. 先在 `valhalla/tyr/actor.h` 找 action 名称。
2. 到 `src/tyr/actor.cc` 看是否调用 Loki、Thor、Odin。
3. 到 `valhalla/thor/worker.h` 看 Thor worker 是否有对应方法。
4. 到 `src/thor/*_action.cc` 看算法入口。
5. 到 `src/tyr/*_serializer.cc` 看输出。

## 本章小结
扩展服务不是重新发明整套系统，而是复用请求解析、位置关联、图读取和 costing，只是在 Thor 算法和 Tyr 序列化处出现分叉。
## 下一步读什么
读 Thor、Loki、Tyr 三章。
## 自测问题
1. Matrix 是否需要位置关联？
2. Isochrone 输出路径还是等时圈？
3. trace_route 与 Meili 有什么关系？
## 源码证据
- `valhalla/tyr/actor.h:58-128`
- `valhalla/thor/worker.h:54-79`
- `src/thor/matrix_action.cc:124-156`
- `src/thor/isochrone_action.cc:46`
