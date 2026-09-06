# 01 认证与身份

## Q: octo-server 请求处理里有哪些认证路径？

结论：
octo-server 的高层请求流程包含 token、cookie、DH-sealed WebSocket frame 三类认证入口。

证据：
- 来源: README.md#L85-L91

说明：
README 在“每个请求做什么”里写明第一步 Authenticate 包含 token / cookie / DH-sealed WebSocket frame。

边界：
这是高层说明；具体中间件和握手细节还需要继续补源码引用。

---

## Q: Bot API 如何区分 User Bot 和 App Bot？

结论：
Bot API 按 token 前缀分流：`app_` 走 App Bot 认证；否则走 User Bot 认证，注释说明 bf_ token 或 legacy tokens 走 robot 表。

证据：
- 来源: modules/bot_api/auth.go#L10-L14
- 来源: modules/bot_api/auth.go#L25-L41

说明：
`BotKindUser` 对应 user bot，`BotKindApp` 对应 app bot；`authBot()` 通过 `strings.HasPrefix(token, "app_")` 区分。

边界：
User Bot 的 robot 表查询和 App Bot 认证细节还需继续补充。

---

## Q: App Bot 的身份前缀是什么？

结论：
App Bot token 前缀是 `app_`，App Bot UID 前缀是 `app_`，UID 后缀是 `_bot`。

证据：
- 来源: modules/app_bot/app_bot.go#L29-L36

说明：
这些常量定义了 App Bot 的 token 和 UID 命名约束。
