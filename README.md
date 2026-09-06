# octo-server PM Exam Repository

AINOL Agent 实操考核 B 卷需求池仓库。

目标项目：Mininglamp-OSS/octo-server  
目标角色：octo-server 产品管家 Agent  
用途：产品问答、Bug/Feature 收集、PRD 撰写、Review 跟进、定时状态回报。

## Agent 能力

1. 产品问答：关键结论必须带 `来源: <相对路径>#L<起>-L<止>`。
2. 需求归档：把 Octo 群里的 Bug / Feature / Question 归档为 GitHub issue。
3. PRD 补全：Feature 类需求补 PRD，只写 What，不写 How。
4. Review 闭环：区分 done / wontfix / cannot-reproduce / need-info。
5. 定时扫描：只在 issue 有变化时回报 Octo 群；无变化不发群，只写 cron log。

## 红线

- 目标仓库 octo-server 只读。
- 不泄露 token / cookie / API key。
- 不编造源码引用。
- 不把“已修复 / 未复现 / 不做”混为一谈。
- 冻结后不修改 Agent 配置。
