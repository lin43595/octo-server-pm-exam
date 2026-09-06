# Source Index

目标仓库：Mininglamp-OSS/octo-server  
当前分析 commit: 49dc9fd

## 已核验关键来源

- README.md：项目定位、REST/WebSocket、Lobster、WuKongIM、Quickstart、部署关系
- configs/tsdd.yaml：基础配置、Webhook 安全、WuKongIM、DB、外网、文件服务、内置账户等配置段
- internal/modules.go：实际启用模块 import 列表
- modules/app_bot/app_bot.go：App Bot 管理 API、权限范围、跨租户防护
- modules/bot_api/auth.go：Bot API token 前缀分流，bf_ user bot 与 app_ app bot

## 引用规则

所有产品/代码结论必须使用：

`来源: <相对路径>#L<起>-L<止>`

如果没有证据，必须回答“不确定”，不得编造路径和行号。
- 深水区源码入口索引：`knowledge/10-deep-dive-source-map.md`
