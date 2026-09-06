# 09 构建与发布

## Q: octo-server 本地构建命令是什么？

结论：
README 给出的本地构建命令是 `go build -o octo-server .`。

证据：
- 来源: README.md#L45-L52

说明：
这是最基础的 standalone binary 构建方式。

---

## Q: octo-server 本地如何指定配置启动？

结论：
README 给出的启动方式是 `./octo-server --config ./configs/tsdd.yaml`。

证据：
- 来源: README.md#L45-L52

说明：
配置文件路径是 `configs/tsdd.yaml`，考试如果问配置从哪里来，可以和 03 配置知识库一起回答。

---

## Q: 默认开发环境为什么不一定能直接跑起来？

结论：
默认开发配置期望本地 WuKongIM 实例和 MySQL-compatible database；如果没有这些外部依赖，服务不能完整启动。

证据：
- 来源: README.md#L54-L58

说明：
考试要求“不要求把它跑起来”，因为重点是 Agent 是否读得懂代码，而不是部署完整依赖。

---

## Q: 官方一键部署在哪里？

结论：
官方 OOTB deployment 在 `Mininglamp-OSS/octo-deployment`；它包含 server、admin、web、matter、smart-summary、WuKongIM、MySQL、Redis、MinIO、nginx 等一整套组件。

证据：
- 来源: README.md#L60-L63

说明：
如果需要完整部署，不应只看 octo-server 单仓库，而要看 octo-deployment。

---

## Q: 旧 docker compose 栈还能作为部署事实来源吗？

结论：
不能。README 明确说旧的 `docker/octo/` 和 `docker/tsdd/` compose stacks 已退役，官方 OOTB deployment 才是 single source of truth。

证据：
- 来源: README.md#L60-L66

说明：
考试如果问部署关系，可以回答：octo-server 是后端单仓库；完整一键部署看 octo-deployment。
