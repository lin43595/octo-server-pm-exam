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
MySQL 是主要持久化数据库；Redis 用于缓存、认证注册、验证码、限流等能力。

---

## Q: App Bot 的认证信息如何跨实例生效？

结论：
App Bot 初始化时会建立共享 Redis auth registry，并从 DB warm up；使用共享 Redis 而不是进程内 map，是为了让 token revocation 在所有副本上立即生效。

证据：
- 来源: modules/app_bot/app_bot.go#L92-L101
- 来源: modules/app_bot/app_bot.go#L941-L950

说明：
这说明 Redis 不只是普通缓存，还承担 App Bot 认证 registry 的跨副本一致性。

---

## Q: 邮箱验证码相关 Redis key 有哪些？

结论：
邮箱验证码使用 `emailcode:` 前缀，并有发送状态、频率限制、失败计数、锁定等 key：`emailcode-status`、`email_rate_limit`、`email_verify_fail`、`email_verify_lock`。

证据：
- 来源: modules/base/common/service_email.go#L28-L59

说明：
这些 key 把不同 CodeType 纳入 keyspace，避免普通用户验证码和管理控制台 MFA 相互串扰。

---

## Q: 邮箱发送频率限制如何表达？

结论：
邮箱验证码发送有 1 分钟 cooldown；`EmailSendRateLimitRetryAfter` 会读取 Redis TTL，并把剩余秒数转换为客户端可行动的 retry hint，同时避免暴露 Redis 错误。

证据：
- 来源: modules/base/common/service_email.go#L119-L145

说明：
这回答了 Redis 在限流/验证码状态里的用途。

---

## Q: 短信验证码缓存 key 是什么？

结论：
短信验证码缓存 key 前缀是 `smscode:`。

证据：
- 来源: modules/base/common/const.go#L27-L30

说明：
这是短信验证码存储/缓存相关的最小可核验证据。

---

## Q: Agent Mail Gateway 有没有本地缓存？

结论：
Agent Mail Gateway 使用 LRU cache 记录 provisioned 身份，默认容量为 4096；同时使用 singleflight 合并并发 provisioning。

证据：
- 来源: modules/agentmailgateway/gateway.go#L45-L50
- 来源: modules/agentmailgateway/gateway.go#L72-L95
- 来源: modules/agentmailgateway/gateway.go#L97-L124

说明：
这不是 Redis，而是进程内 LRU cache，用于降低重复 provisioning 成本。

---

## Q: Agent Mail Gateway 为什么禁止共享缓存复用私有响应？

结论：
每个响应都设置 `Cache-Control: private, no-store`，并设置 `Vary: token, X-Space-ID, X-Octo-Mailbox-ID`，避免上游缓存把一个用户的私有 mail 响应复用给另一个调用方。

证据：
- 来源: modules/agentmailgateway/gateway.go#L409-L415

说明：
这属于外部依赖/缓存边界里的安全约束。

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
