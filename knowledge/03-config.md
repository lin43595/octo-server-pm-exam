# 03 配置

## Q: octo-server 默认如何启动？

结论：
README 给出的本地启动方式是 clone 仓库后执行 `go build -o octo-server .`，再用 `./octo-server --config ./configs/tsdd.yaml` 启动。

证据：
- 来源: README.md#L45-L52

说明：
考试不要求跑起来，但构建与配置路径要能说明清楚。

---

## Q: 默认开发配置依赖什么？

结论：
默认开发配置期望本地 WuKongIM 实例和 MySQL-compatible database。

证据：
- 来源: README.md#L54-L58

说明：
这也解释了为什么考试要求“不需要跑起来”，重点是读懂代码。

---

## Q: configs/tsdd.yaml 里有哪些基础配置？

结论：
基础配置包含运行模式、管理员密码、API 监听地址、Webhook gRPC 监听地址、appName、rootDir、跨设备消息保存、欢迎语、手机号搜索、在线状态、群升级人数、事件池大小等。

证据：
- 来源: configs/tsdd.yaml#L1-L13

说明：
这些是服务启动和基础行为相关配置。

---

## Q: Webhook 安全配置是什么？

结论：
`webhookSecretKey` 用于配置入站 webhook 的 HMAC-SHA256 签名校验；配置后请求必须携带 `X-Signature-256`，格式为 `sha256=<hex(HMAC-SHA256(body, secret_key))>`；留空则不验证。

证据：
- 来源: configs/tsdd.yaml#L15-L19

说明：
这是 webhook 入站安全边界，考试如果问凭证/签名安全，可以引用这里。

---

## Q: WuKongIM 和 DB 配置在哪里？

结论：
WuKongIM 配置包含 `apiURL` 和 `managerToken`；DB 配置包含 MySQL 地址、Redis 地址/密码/TLS，以及异步任务 Redis 地址。

证据：
- 来源: configs/tsdd.yaml#L21-L34

说明：
octo-server 默认开发环境依赖 WuKongIM 和 MySQL，Redis 也在 DB 配置段里。

---

## Q: 文件服务支持哪些对象存储？

结论：
文件服务配置里列出了 MinIO、Tencent COS、Aliyun OSS、Qiniu、SeaweedFS 等存储形态；其中预签名 PUT/GET 能力因后端不同而不同。

证据：
- 来源: configs/tsdd.yaml#L69-L103

说明：
考试如果问对象存储或上传下载边界，可以引用这一段。
