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
这说明 API 响应有统一封装概念，但具体响应结构、错误码字段、HTTP 状态码映射还需要继续补源码证据。

边界：
当前知识库还没有完整覆盖统一响应体结构和错误码分类。考试被问到具体字段时，应回答“不确定，需要补查 common / wkhttp / API handler 相关代码”。

---

## Q: API 问题回答时应该怎么处理不确定项？

结论：
如果只找到 README 的高层说明，但没有找到具体源码结构，Agent 必须明确说“不确定”，不能编造响应字段或错误码。

证据：
- 来源: README.md#L85-L91

说明：
考试红线是不允许编造引用。API 与错误约定这一块如果没有源码证据，应该诚实说明待补查。

---

## Q: 和文件上传相关的 API 有什么明确契约？

结论：
`GET /v1/file/upload-credentials` 会返回 `contentType` 和可能的 `contentDisposition`；浏览器 PUT 上传时必须带匹配请求头，否则对象存储签名校验会失败。

证据：
- 来源: configs/tsdd.yaml#L78-L86

说明：
这是一个很好的 API 契约例子：客户端必须按服务端签名时的 header 约定上传，否则会出现 `403 SignatureDoesNotMatch`。
