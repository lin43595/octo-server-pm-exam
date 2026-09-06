# 10 深水区源码入口索引

本页用于处理知识库未完全展开的深水区问题。遇到以下问题时，必须先查对应源码；没有证据则回答：`我不确定，当前知识库没有足够证据。需要补充检查 <模块/文件>。`

## 1. token / cookie / WebSocket 认证路径

- 高层流程：`octo-server` README 说明每个请求会 Authenticate，包含 token / cookie / DH-sealed WebSocket frame。
  来源: README.md#L85-L91
- 普通登录 token / cookie 的底层认证中间件在 `octo-lib`，`octo-server` 通过 `ctx.AuthMiddleware()` 调用。
  来源: octo-lib/config/context.go#L139-L143
  来源: octo-lib/pkg/wkhttp/http.go#L303-L338
- Bot API token 在 `octo-server` 内处理：从 `Authorization: Bearer` 提取；`app_` 走 App Bot，其他走 User Bot / `bf_` 或 legacy token。
  来源: modules/bot_api/auth.go#L25-L41
  来源: modules/bot_api/auth.go#L143-L150
- App Bot 的 API token 和 IM WebSocket token 是同一个 token；轮换会同时影响 API auth 与 IM WebSocket 连接。
  来源: modules/bot_api/register.go#L478-L504

## 2. org RBAC / channel ACL / 逐接口权限边界

- 高层流程：认证后做 org-aware RBAC、per-channel ACL、agent-identity gating。
  来源: README.md#L85-L91
- App Bot 管理路由分平台管理员和 Space 管理员两类，均挂登录认证。
  来源: modules/app_bot/app_bot.go#L116-L154
- App Bot 有跨租户防护：平台路由只管 platform bot，space 路由只管对应 space bot。
  来源: modules/app_bot/app_bot.go#L171-L180
- 更细的频道/消息权限分散在各模块，必须按具体接口继续查，不能只凭 README 高层表回答。
  推荐入口：
  - `modules/messages_search/authz.go`
  - `modules/message/api_authorize.go`
  - `modules/*/authtree_guard.go`

## 3. octo-server 与 WuKongIM 分工

- `octo-server` 是业务后端、REST/WebSocket API 和 WuKongIM 控制面；它驱动 WuKongIM 做实时消息。
  来源: README.md#L29-L43
- `internal/im/` 是 WuKongIM 控制面 client，负责 channel / message / presence。
  来源: README.md#L75-L79
- 如果问题追问 WuKongIM 内部实现，需要去 `WuKongIM/WuKongIM` 仓库查；`octo-server` 侧只回答控制面边界。

## 4. modules/ 当前启用状态

- 当前启用模块以 `internal/modules.go` 的 blank import 为准。
  来源: internal/modules.go#L22-L78
- `modules/` 目录存在不等于当前启用。
- 截至本次检查，源码目录中存在但未被 `internal/modules.go` import 的模块：
  - `botidentity`
  - `cardtrust`
  - `source`
- 因此回答“当前启用模块”时必须以 `internal/modules.go` 为准，不得只按目录名判断。
