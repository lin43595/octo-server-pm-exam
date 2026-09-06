# 06 IM 控制面

## Q: octo-server 和 WuKongIM 的关系是什么？

结论：
octo-server 是 OCTO 平台后端，驱动 WuKongIM IM core 做实时消息；README 也说 WuKongIM 通过薄控制面边界被驱动，使 IM core 可替换。

证据：
- 来源: README.md#L31-L37
- 来源: README.md#L41-L43

说明：
octo-server 不是底层 IM 内核本身，而是业务后端和控制面。WuKongIM 更偏实时消息内核。

---

## Q: 请求处理过程中 IM 相关动作在哪一步？

结论：
请求处理流程的 Fan out 阶段会把 IM 消息 enqueue 到 WuKongIM，并在频道需要外部桥接时触发 adapters。

证据：
- 来源: README.md#L85-L91

说明：
这说明业务请求执行后，消息分发由 WuKongIM 和 adapter 体系共同承接。

---

## Q: octo-server 是否直接承载所有实时消息能力？

结论：
不是。README 的表述是 octo-server 驱动 WuKongIM IM core，并作为控制面；实时消息核心能力由 WuKongIM 承担。

证据：
- 来源: README.md#L31-L37
- 来源: README.md#L41-L43

说明：
考试如果问“哪些走 server，哪些走 IM”，可以这样答：业务控制、账号/频道/消息编排等由 server 处理；底层实时消息投递由 WuKongIM 承担。更细边界需要继续补源码证据。

边界：
当前知识库还没有逐接口列出哪些操作走 server、哪些直连 IM。被问到具体接口时，应说“不确定，需要补查对应 module 和 WuKongIM client 调用”。

---

## Q: adapter 在 IM 流程里起什么作用？

结论：
请求处理的 fan out 阶段，在频道需要外部桥接时会触发 adapters。

证据：
- 来源: README.md#L85-L91

说明：
adapter 可理解为外部桥接/分发面的一部分，但具体 adapter 注册和触发规则需要继续补代码证据。

---

## Q: 为什么说 WuKongIM 是可替换的？

结论：
README 说明 WuKongIM 是通过 thin control-plane boundary 被驱动，因此 IM core remains swappable。

证据：
- 来源: README.md#L41-L43

说明：
这代表 octo-server 与 IM 内核之间有控制面边界，不把所有业务逻辑强耦合进 IM 内核。
