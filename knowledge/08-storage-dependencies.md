# 08 存储与外部依赖

## Q: octo-server 默认开发环境依赖哪些外部服务？

结论：
默认开发配置期望本地 WuKongIM 实例和 MySQL-compatible database。

证据：
- 来源: README.md#L54-L58

说明：
这也是考试要求“不要求你把它跑起来”的原因：它需要 MySQL 和本地 WuKongIM 实例才能完整启动。

---

## Q: DB 配置里包含哪些内容？

结论：
DB 配置包含 MySQL 地址、Redis 地址、Redis 密码、Redis TLS 配置，以及异步任务 Redis 地址。

证据：
- 来源: configs/tsdd.yaml#L26-L34

说明：
MySQL 是主要持久化数据库；Redis 用于缓存或异步任务相关能力，具体缓存哪些内容还需要继续补源码证据。

边界：
当前知识库还没有完整梳理 Redis 在每个模块里的用途，被问到具体缓存项时应说“不确定，需要补查对应模块”。

---

## Q: 文件服务支持哪些对象存储？

结论：
文件服务配置中列出了 MinIO、Tencent COS、Aliyun OSS、Qiniu、SeaweedFS 等存储形态；不同后端对 presigned PUT / presigned GET 的支持不同。

证据：
- 来源: configs/tsdd.yaml#L69-L103

说明：
这说明 octo-server 的文件能力不是绑定单一对象存储，而是支持多种后端。

---

## Q: 浏览器直传文件有什么签名契约？

结论：
`GET /v1/file/upload-credentials` 返回 `contentType` 和可能的 `contentDisposition`；浏览器 PUT 上传时必须携带匹配 header，因为这些 header 会进入 SigV4 / OSS 签名校验。偏离会导致 `403 SignatureDoesNotMatch`。

证据：
- 来源: configs/tsdd.yaml#L78-L86

说明：
这是文件上传 API 的关键外部依赖契约：客户端必须按服务端签名时的 header 原样上传。

---

## Q: migration 执行顺序由什么决定？

结论：
migration 执行顺序由 SQL 文件名时间戳决定，而不是 `internal/modules.go` 里的 blank import 顺序。

证据：
- 来源: internal/modules.go#L1-L18

说明：
这是容易误解的点。`internal/modules.go` 中保留历史排序是为了可读性，不是 load-bearing；真正排序由 SQL 文件时间戳决定。
