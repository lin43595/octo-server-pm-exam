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

---

## Q: Dockerfile 是怎么构建镜像的？

结论：
`Dockerfile` 是多阶段构建：第一阶段使用 `golang:1.25` 下载依赖并在仓库内 `go build` 生成静态二进制 `app`；第二阶段使用 `alpine:3.21` 作为运行镜像，复制 app、assets、configs，并以 `/home/app` 为 entrypoint。

证据：
- 来源: Dockerfile#L11-L32
- 来源: Dockerfile#L35-L48

说明：
这个 Dockerfile 适合从源码直接构建运行镜像。

---

## Q: Dockerfile 如何处理没有 git tag 的 OSS 构建？

结论：
`Dockerfile` 对 `git describe --tags --abbrev=0` 增加 `2>/dev/null || echo dev` fallback；注释说明 OSS repo 可能 tags 被剥离，首次构建时 `git describe` 会失败，因此用 `dev` 保证构建通过。

证据：
- 来源: Dockerfile#L1-L9
- 来源: Dockerfile#L26-L32

说明：
这是 OSS 发布环境和内部构建环境的差异处理。

---

## Q: Dockerfile.ghcr 和 Dockerfile 有什么区别？

结论：
`Dockerfile.ghcr` 不在镜像内编译 Go 源码，而是基于 `debian:bookworm-slim`，安装 ca-certificates/tzdata，复制 assets、configs，以及预构建的 `linux_${TARGETARCH}` 二进制为 `main`，最后 `CMD ["/app/main"]`。

证据：
- 来源: Dockerfile.ghcr#L1-L17

说明：
二者核心区别：`Dockerfile` 源码内构建；`Dockerfile.ghcr` 消费预构建产物，更像发布流水线产物打包。
