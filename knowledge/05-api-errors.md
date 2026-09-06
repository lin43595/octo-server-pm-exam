# 05 API 与错误约定

## Q: octo-server 对外提供什么类型的 API？

结论：
octo-server 对外提供 REST + WebSocket API，供 octo-web、octo-admin 等客户端使用。

证据：
- 来源: README.md#L29-L37

说明：
README 明确说 octo-server 是 Go backend，提供 REST + WebSocket APIs，并承载业务编排和 Lobster agent 调度。

---

## Q: octo-server 每个请求的大致处理流程是什么？

结论：
请求大致经过认证、鉴权、执行业务逻辑、IM fan out、响应五个步骤。

证据：
- 来源: README.md#L85-L91

说明：
README 写明每个请求会 Authenticate、Authorise、Execute、Fan out、Respond。

---

## Q: 响应体是否有统一约定？

结论：
README 高层说明请求最终会返回 unified JSON envelope，或者 WebSocket frame，并带 tracing + metrics tags。

证据：
- 来源: README.md#L85-L91

说明：
这说明 API 响应有统一封装概念；更细字段要继续看错误渲染和 wkhttp/httperr 相关实现。

---

## Q: 错误码注册表的设计原则是什么？

结论：
错误码通过 `codes.Register` 注册；ID 是全局唯一稳定 i18n key；重复 ID 或非法 ID 会在注册阶段 panic；错误码包含 HTTPStatus、DefaultMessage、DefaultMessages、SafeDetailKeys、Internal 等元信息。

证据：
- 来源: pkg/i18n/codes/registry.go#L1-L18
- 来源: pkg/i18n/codes/registry.go#L45-L67
- 来源: pkg/i18n/codes/registry.go#L74-L104

说明：
这说明错误约定不是散落文案，而是有中心注册表、命名规则、HTTP status、详情白名单和内部错误保护。

---

## Q: 错误码 ID 的命名规则是什么？

结论：
错误码 ID 必须以 `err.shared.` 或 `err.server.` 开头，后面接小写字母、数字、下划线组成的点分 segment；不接受大写、空 segment 或其他命名空间。

证据：
- 来源: pkg/i18n/codes/registry.go#L28-L43

说明：
这可以回答“错误码怎么分类”：跨模块通用错误进 shared，业务专属错误进 server/module 命名空间。

---

## Q: HTTP 状态码在哪里定义？

结论：
每个注册错误码包含 `HTTPStatus` 字段，要求在 100-599 范围内；兼容期内 renderer 可能把响应头固定为 400，但 body 的 `error.http_status` 仍暴露真实 canonical HTTP status。

证据：
- 来源: pkg/i18n/codes/registry.go#L47-L57
- 来源: pkg/i18n/codes/registry.go#L92-L94

说明：
这解释了 HTTP 状态码映射的来源和兼容边界。

---

## Q: 通用 shared 错误码有哪些例子？

结论：
shared 错误码覆盖 auth required、token missing、token invalid、token expired、forbidden、rate limited、param invalid、not found、internal 等通用场景，并映射到 401/403/429/400/404/500 等 HTTP 状态。

证据：
- 来源: pkg/i18n/codes/shared.go#L21-L107

说明：
这些是跨 module 的通用错误，业务专属错误应归到 `pkg/errcode` 的 server 命名空间。

---

## Q: 错误详情如何避免泄露敏感信息？

结论：
`SafeDetailKeys` 是 details 字段白名单，renderer 只透传白名单内的 key，防止业务层误把 uid/token/raw_err 等泄露给客户端；Internal=true 的 5xx 错误会输出占位文案，避免内部 message 泄露。

证据：
- 来源: pkg/i18n/codes/registry.go#L11-L14
- 来源: pkg/i18n/codes/registry.go#L56-L59
- 来源: pkg/i18n/codes/shared.go#L96-L106

说明：
这和考试红线“凭证不许出现在群里，也不许进 git”一致：错误响应也不能透出敏感细节。

---

## Q: Bot API 错误码如何分类？

结论：
`pkg/errcode/bot_api.go` 把 Bot API 错误分成 validation(400)、permission/authorization(403)、not found(404) 等分类；部分外部适配器依赖真实 HTTP status，因此通过 status-preserving 路径保留 wire status。

证据：
- 来源: pkg/errcode/bot_api.go#L9-L20
- 来源: pkg/errcode/bot_api.go#L21-L113
- 来源: pkg/errcode/bot_api.go#L114-L205
- 来源: pkg/errcode/bot_api.go#L206-L230

说明：
这块能回答 Bot API 错误码如何分类，以及为什么某些外部 API 要保留真实 HTTP 状态。

---

## Q: 和文件上传相关的 API 有什么明确契约？

结论：
`GET /v1/file/upload-credentials` 会返回 `contentType` 和可能的 `contentDisposition`；浏览器 PUT 上传时必须带匹配请求头，否则对象存储签名校验会失败。

证据：
- 来源: configs/tsdd.yaml#L78-L86

说明：
这是一个明确 API 契约：客户端必须按服务端签名时的 header 约定上传，否则会出现 `403 SignatureDoesNotMatch`。
