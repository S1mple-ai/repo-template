# AGENTS.md — AI Agent 协作规范（v5 定稿）

> 适用范围：本仓库及对应服务的全部 AI Agent 操作。模板通用，建仓时按项目填空。
> **本模板仓是公开仓库，全仓不得出现任何真实服务器信息（见第十三章）。**

## 一、项目信息

| 项 | 值 |
|---|---|
| 项目名 | <待填> |
| 用途 | <工作 / 生活·目标管理> |
| 技术栈 | <待填> |
| 生产域名 | <待填> |
| 服务器 | <待填> |
| 服务名（systemd） | <待填> |
| 部署目录 | <如 /opt/xxx> |
| 日志命令 | `journalctl -u <svc> -f`（或项目实际日志路径） |
| 回滚方式 | 见部署规范第 5 条（必须写具体命令且实测过） |

## 二、分支规则

- `main`：受保护，**禁止直接 push**，仅通过 PR 合并
- `dev`：日常集成分支
- `feature/*`：功能分支，从 dev 拉出
- 流程：feature → dev →（确认后）→ main

## 二·补、私有仓无分支保护的纪律约束（B2 模式）

> 背景：GitHub Free 的**私有仓不能开分支保护规则**（公开仓可以）。本仓为私有仓，保护规则开不了。

- **「禁止直推 main」是纪律约束，不是技术强制。** push 不会被服务器拒绝，全靠 Agent 自觉。
- **main 操作双报告制**：WorkBuddy 任何对 main 的操作（合并 PR、回滚、直接提交文件）——
  1. 动手前：单独报告意图（改什么、为什么、影响哪些文件），等确认；
  2. 动完后：单独报告结果，并附审计证据——拉 `GET /repos/<owner>/<repo>/commits?sha=main&per_page=5`，把真实 commit 历史（sha、作者、时间、消息）原样写进人话报告。
- **CI 的定位**：CI 在 PR 上跑，能发现语法错误、密钥入库、明显 lint 问题，但**拦不住直推 main**。CI 绿 ≠ main 安全，不要误判。
- 静默修改 main 视为违反红线，等同第 2 条「禁止直接 push 到 main」。

## 二·补 2、依赖关系

> 建仓时按实际服务填写。**任何修改本服务对外接口的 PR，必须逐一检查下列调用方是否受影响**，检查结果写进人话变更报告。

| 方向 | 调用方 → 被调方 | 方式 |
|---|---|---|
| 正向 | 本服务 → <被依赖服务> | <调用方式> |
| 反向 | <依赖方 A> → 本服务 | <端口/接口> |
| 反向 | <依赖方 B> → 本服务 | <端口/接口> |

> 反向依赖意味着：改接口、改端口、改鉴权前必须确认调用方，否则会连带打挂别的服务。

## 三、提交规范

- 格式：`<type>(<scope>): <描述>`
- type：feat / fix / docs / refactor / test / chore
- 示例：`feat(order): 支持按人分单`
- 禁止提交：`.env`、`secrets/`、`*.key`、`*.pem`、任何密钥

## 四、红线（绝对禁止）

1. 禁止读取/修改/提交 `.env`、`secrets/`、`*.key`、`*.pem`
2. 禁止直接 push 到 main
3. 禁止删除数据库、清空表
4. 禁止修改 GitHub 分支保护规则
5. 禁止在代码中硬编码密钥
6. 禁止未经确认操作生产环境高风险变更

### GitHub Administration 禁用清单（与第十二章 REST 清单一致）

1. 删除仓库（DELETE /repos/*）
2. 修改分支保护（*/branches/*/protection）
3. 修改 Secrets（*/actions/secrets/*）
4. 修改 Collaborators（*/collaborators/*）
5. 修改仓库可见性（PATCH /repos/* {private}）
6. 修改 Variables（*/actions/variables/*）
7. 修改 Webhooks（*/hooks/*）

> 即使老板聊天口头要求，也必须**单独反问确认**，不得与其他任务混合执行。

## 五、人话变更报告（硬性要求）

**任何风险等级（含低）都不能省略，每次变更后必须输出：**

```
【变更目的】一句话
【影响范围】哪些功能/页面/接口
【风险等级】低 / 中 / 高
【测试结果】怎么验证的 + 真实证据（状态码/日志/接口返回）
【回滚方式】具体命令
```

## 六、三级风险分层

| 等级 | 行为 |
|---|---|
| 低 | 自动执行，**但必须在 WorkBuddy 聊天里发通知，禁止静默** |
| 中 | 聊天确认后执行 |
| 高 | 明确确认 + 二次确认 |

**加码规则**：涉及**生产环境、DB schema、域名/证书、支付、对外接口**的，无论判定多低，**一律至少按中风险走**。

## 七、部署规范

1. **部署前**：出部署清单（改哪些文件、执行哪些命令）
2. **部署中**：记录执行的命令
3. **部署后**：验证服务正常（`systemctl is-active` + `curl` 活接口 + 日志）
4. **保留上一版**：备份到 `/opt/backup/<日期>/`
5. **回滚命令必须实测**：回滚方式不能只写"恢复备份"，必须写**具体命令**；且部署前至少验证过一次备份可恢复（备份文件存在 + md5 一致）

## 八、密钥管理规范

| 项 | 规定 |
|---|---|
| .env 位置 | 各服务目录下，**权限 600**（实际路径以 SSH 实测为准，勿信记忆） |
| 引用 | systemd `EnvironmentFile=/path/.env`，代码里只读 `process.env` / `os.environ` |
| Agent 能做 | 创建/修改 `EnvironmentFile` 引用、改 unit 文件、设系统环境变量 |
| Agent 禁止 | **读取 .env 内容**、提交 .env 入库、把密钥写进代码/日志/对话 |
| 唯一例外 | key 名清点：`grep -oE '^[A-Za-z_]+=' .env \| sed 's/=$//'`（禁止 `cut -d= -f1` 整文件读入） |

## 九、Token 权限管理

| 项 | 规定 |
|---|---|
| Token 类型 | Fine-grained，最大授权（Contents/PR/Issues/Administration/Workflows/Secrets/Variables/Webhooks 读写） |
| 存放 | 仅 Windows 用户级环境变量 `GITHUB_PERSONAL_ACCESS_TOKEN`，任何配置文件/代码里不得出现明文 |
| 禁用清单 | 见第四章 Administration 禁用清单 + 第十二章 REST 清单 |
| 执行规则 | 清单内操作**即使老板口头要求也必须单独反问确认，不得混在其他任务里执行** |
| **紧急止血开关** | 在 `mcp.json` github 条目 headers 加 `"X-MCP-Readonly": "true"`，对 GitHub MCP **一键全锁只读**（连建仓都禁）。这是出事时的熔断机制，**平时不开**；启用后记得移除，否则 MCP 持续只读 |

## 十、服务器服务映射（附录，建仓时填）

> **本模板为公开仓，下表只允许占位符。真实值只放私有仓的 AGENTS.md 或本地笔记（见第十三章 13.2）。**

### <服务器A IP>（<域名A>）

| 服务名 | 端口 | 部署目录 | 日志 | GitHub 仓库 |
|---|---|---|---|---|
| <服务名> | <端口> | <部署目录> | <日志命令> | <建仓时填> |

### <服务器B IP>（<域名B>）

| 服务名 | 端口 | 部署目录 | 日志 | GitHub 仓库 |
|---|---|---|---|---|
| <服务名> | <端口> | <部署目录> | <日志命令> | <建仓时填> |

## 十一、环境变量与密钥（项目级）

- 服务器 `.env`：权限 600，不入库，`.gitignore` 强制忽略
- 代码引用：一律 `process.env.XXX` / `os.environ["XXX"]`
- 本机凭证：环境变量（如 `GITHUB_PERSONAL_ACCESS_TOKEN`），配置文件只写 `${VAR}` 引用或直接由脚本从环境读取

## 十二、GitHub 操作路径与约束

| 项 | 规定 |
|---|---|
| **主路径** | 所有 GitHub 操作**优先走 REST 直调**（环境变量 token + 脚本）。MCP 工具为可选路径，**不依赖** |
| 原因 | WorkBuddy MCP 层对 custom-mcp headers 存在 OAuth 劫持问题（daemon 400 invalid_token），REST 直调实测稳定 |

### REST 层禁用清单（即使老板口头要求，也必须单独反问确认，不得直接执行）

```
1. 删除仓库           DELETE /repos/{owner}/{repo}
2. 修改分支保护       PUT/PATCH/DELETE /repos/*/branches/*/protection
3. 修改 Secrets       PUT/DELETE /repos/*/actions/secrets/*
4. 修改 Variables     PUT/DELETE /repos/*/actions/variables/*
5. 修改 Collaborators PUT/DELETE /repos/*/collaborators/*
6. 修改可见性         PATCH /repos/*  {private:...}
7. 修改 Webhooks      POST/PATCH/DELETE /repos/*/hooks/*
8. 绕过 secret scanning push protection（推送时带 allow-secret-scanning 或等价参数）
```

### mcp-tool-policy.json 的实际作用范围（防误解）

- 文件位置：`~/.workbuddy/mcp-tool-policy.json`，内容 `{"version":1,"disabledTools":{"github":["delete_file"]}}`
- **作用机制**：仅在 MCP 工具列表注入会话时过滤（不暴露被禁工具）
- MCP 层唯一 DESTRUCTIVE 工具是 `delete_file`；删仓库/密钥/Webhook 类工具远程端点本就不暴露，**这些只能走 REST，由本清单约束**
- policy 文件保留：MCP 修好后自动生效，不修也无害

## 十三、公开仓库特别约束

本节适用于**所有公开仓库**（包括本模板仓）。私有仓库不受本节约束。

### 13.1 禁止提交的内容（硬红线）

以下内容**永不进入公开仓库**，无论当前版本还是 Git 历史：

1. **服务器 IP**（含 IPv4/IPv6，包括内网 IP）
2. **真实域名**（含子域名）
3. **部署路径**（如 `/opt/xxx`、`/srv/xxx`）
4. **端口映射**（服务名与端口的对应关系）
5. **日志路径**（如 `/tmp/xxx.log`、`journalctl -u xxx` 的服务名）
6. **服务名**（systemd unit 名、pm2 进程名）
7. **任何密钥、token、pem 路径**（红线已有，此处重申）
8. **内部服务调用关系**（如 A 调 B 的地址和端口）

### 13.2 服务器信息的正确存放位置

- **私有仓**：可放真实值
- **公开仓**：只能放占位符，形如 `<服务器A IP>`、`<域名>`、`<部署目录>`
- **本地笔记**：真实值的权威源，不依赖任何仓库

### 13.3 commit 前强制检查

公开仓的任何 commit 前，必须执行：

```bash
grep -rEn '([0-9]{1,3}\.){3}[0-9]{1,3}|[a-z0-9-]+\.(cn|com|net|top)|/opt/|/srv/|AKLT|sk-|ghp_' .
```

检出任何命中，逐个确认是真实值还是占位符/文档示例。真实值一律改为占位符后再提交。

### 13.4 Git 历史也要干净

- 即使当前版本已清值，历史 commit 里的旧 blob 仍可访问
- 误提交真实值后，唯一彻底方案是重写历史或重建仓库
- 不要把「新 commit 覆盖旧值」当作清值方案

### 13.5 万一泄露

1. 立即转私有（止损）
2. 重建仓库（清历史）
3. 通知用户：服务器侧要收紧安全组/防火墙（IP 已公开）
4. 记录事件：时间、范围、处理动作

---

_v5 2026-09-28（占位符版 + 第十三章公开仓约束，重建后首版）。_
