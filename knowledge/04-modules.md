# 04 业务模块清单

## Q: 实际启用模块应看哪里？

结论：
实际启用模块应优先看 `internal/modules.go` 的 blank import 列表，而不是只看 README 架构表。

证据：
- 来源: internal/modules.go#L22-L78

说明：
考试要求已提醒 README 架构表和实际目录可能不一致，以代码为准。

---

## Q: 当前 import 里有哪些核心业务模块？

结论：
当前 import 包含 user、group、channel、message、file、thread、space、notification、notify、webhook、openapi、search、statistics、sticker、workplace 等业务模块。

证据：
- 来源: internal/modules.go#L35-L63
- 来源: internal/modules.go#L75-L77

说明：
这些模块构成 octo-server 的主要业务能力，包括用户、群组、频道、消息、文件、通知、搜索和工作台等。

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

## Q: runtime 模块现在还在 octo-server 里负责 bot 编排吗？

结论：
runtime/bot orchestration 已由单独的 octo-fleet 服务负责；`modules/runtime` 已移除。

证据：
- 来源: internal/modules.go#L53-L57

说明：
这是一个容易被问到的点：不要把 runtime 仍归到 octo-server 当前职责里。

---

## Q: usersecret 模块负责什么？

结论：
`usersecret` 提供用户外部密钥别名表、write-only CRUD 和 resolve 能力；鉴权按 `bf_` bot token 反查 robot.creator_uid，运行期查 robot 表。

证据：
- 来源: internal/modules.go#L63-L66

说明：
这和考试红线“凭证不许出现在群里，也不许进 git”有关。Agent 可以管理密钥别名或检查可用性，但不能输出明文。

## 当前启用状态校准

注意：`modules/` 目录存在不等于当前启用。当前启用模块必须以 `internal/modules.go` 的 blank import 为准；截至本次检查，`botidentity`、`cardtrust`、`source` 目录存在但未被 `internal/modules.go` import。

来源: internal/modules.go#L22-L78
