# 07 Bot 与 Agent

## Q: octo-server 是否把 Agent 当作一等参与者？

结论：
README 明确说 Lobster orchestration 是 first class，OpenClaw-powered digital doubles 的 routing、session、tool-call execution 内置在 server 中，Agents 被视为一等会话参与者。

证据：
- 来源: README.md#L39-L43

说明：
这说明 octo-server 不只是普通业务后端，还承载 Agent 参与会话的产品能力。

边界：
具体运行时 orchestration 现已部分由 octo-fleet 承担，见 runtime 移除说明。

---

## Q: Bot/Agent 相关模块有哪些？

结论：
Bot/Agent 相关模块包括 robot、bot_mention、botfather、bot_api、app_bot、bot_provision、botidentity，以及 agentmailgateway 等。

证据：
- 来源: internal/modules.go#L24-L33
- 来源: internal/modules.go#L63-L75

说明：
这些模块分别覆盖机器人主体、机器人提及、BotFather 管理、Bot API、App Bot、bot provisioning、bot identity 和 agent mail gateway 等能力。

---

## Q: App Bot 管理 API 分哪些范围？

结论：
App Bot 管理 API 分平台管理员 API 和 Space 管理员 API；两者都要求登录，Space API 还需要 space admin 检查。

证据：
- 来源: modules/app_bot/app_bot.go#L116-L154
- 来源: modules/app_bot/app_bot.go#L156-L168

说明：
平台侧路径是 `/v1/admin/app_bot`，Space 侧路径是 `/v1/space/:space_id/app_bot`。

---

## Q: App Bot 是否有跨租户访问防护？

结论：
`botInRouteScope` 用来确认 bot 是否属于当前路由范围，防止平台路由读取或轮换任意 space bot token 的跨租户 IDOR。

证据：
- 来源: modules/app_bot/app_bot.go#L171-L180

说明：
这是 App Bot 管理里的一个重要权限边界：平台 route 只能管理 platform-scoped bots，space route 只能管理对应 space 的 bots。

---

## Q: Bot API 的认证身份有哪些上下文字段？

结论：
Bot API 认证后会使用 robot_id、bot_kind、robot、app_bot_scope、app_bot_space_id 等上下文字段表达身份。

证据：
- 来源: modules/bot_api/auth.go#L16-L23

说明：
这些上下文字段让后续 handler 能知道当前请求来自哪类 bot、哪个 robot 或 app bot scope。
