# repo-template — AI Agent 协作规范模板验证仓

本仓库是 [AGENTS.md](AGENTS.md) v4 协作规范的**模板本体 + 流程验证仓**。

## 用途

1. 承载通用 `AGENTS.md` 模板（占位符保留），真实项目建仓时以本仓为起点复制定制
2. 验证完整协作流程：feature 分支 → PR → CI（最小 lint）→ 合并 → main 保护
3. 分支策略：`main`（保护，禁止直推，必须过 CI 状态检查）+ `dev` + `feature/*`

## 从本模板派生真实项目

1. 建新私有仓库（历史从零）
2. 复制本仓全部文件 → 填 AGENTS.md 第一章「项目信息」+ 第十章勾选服务行
3. 按项目技术栈替换 `.github/workflows/ci.yml` 的 lint 步骤
4. 开启 main 保护规则（每次需单独确认）

## 红线速查（详见 AGENTS.md 第四章、第十二章）

- 禁止直推 main；禁止提交 `.env` / `*.key` / `*.pem` / 任何密钥
- 删仓库 / 分支保护 / Secrets / Webhooks 等 8 项 REST 操作：即使口头要求也必须单独反问确认
- 每次变更必出人话变更报告（目的/影响/风险/测试证据/回滚）

## 流程验证

- 2026-09-28 feature/ci-verify：验证 PR → CI → 合并全流程（本行由 AI Agent 添加）。
