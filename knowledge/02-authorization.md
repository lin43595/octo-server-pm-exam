# 02 鉴权模型

## Q: octo-server 的鉴权模型大概分几层？

结论：
从 README 的高层请求流程看，octo-server 在认证之后会做 Authorise，包含 org-aware RBAC、per-channel ACL、agent-identity gating 三类鉴权/门禁。

证据：
- 来源: README.md#L85-L91

说明：
这说明鉴权不是单一权限判断，而是组织级角色、频道级访问控制、Agent 身份门禁共同组成。

边界：
README 是高层描述，具体实现细节需要继续补查对应模块源码。

---

## Q: App Bot 管理接口是否需要登录？

结论：
App Bot 的平台管理员 API 和 Space 管理员 API 都通过 `ab.ctx.AuthMiddleware(r)` 注册，说明需要登录认证。

证据：
- 来源: modules/app_bot/app_bot.go#L116-L154

说明：
平台侧路径是 `/v1/admin/app_bot`，Space 侧路径是 `/v1/space/:space_id/app_bot`，两类管理 API 都挂了认证中间件。

---

## Q: Space App Bot 管理是否需要 Space 管理员权限？

结论：
Space App Bot 管理除了登录外，还需要检查当前登录用户是否是该 space 的 admin/owner。

证据：
- 来源: modules/app_bot/app_bot.go#L156-L168

说明：
代码查询 `space_member`，要求成员存在、状态有效，并且 `Role >= spaceRoleAdmin`。注释说明 0=member，1=admin，2=owner。

---

## Q: App Bot 是否有跨租户访问防护？

结论：
有。`botInRouteScope` 用来判断 bot 是否属于当前路由范围，防止平台路由读取或轮换任意 space bot token 的跨租户 IDOR。

证据：
- 来源: modules/app_bot/app_bot.go#L171-L180

说明：
这是 App Bot 管理里的一个重要权限边界：平台 route 只能管理 platform-scoped bots，space route 只能管理对应 space 的 bots。

---

## Q: Bot API 如何做身份门禁？

结论：
Bot API 使用统一认证中间件 `authBot()`，按 token 前缀区分 App Bot 和 User Bot；`app_` 走 App Bot 认证，否则走 User Bot 认证。

证据：
- 来源: modules/bot_api/auth.go#L25-L41

说明：
这属于 bot/agent 身份门禁的一部分。不同 bot 类型进入不同认证路径，后续权限边界也不同。
