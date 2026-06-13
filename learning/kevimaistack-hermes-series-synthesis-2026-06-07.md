---
title: "KevinAIStack Hermes 系列学习综合：个人 AI Stack 架构"
date: 2026-06-07
category: learning
source: "KevinAIStack 微信公众号系列文章（8篇）"
status: synthesized
---

# 个人 AI Stack 架构综合

> 基于 KevinAIStack 2026年5-6月发布的 8 篇 Hermes Agent 实践文章的综合学习记录。
> 核心原则：信息是为了打开思路，而非没有就停止研究。

---

## 一、架构总览

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         PERSONAL AI STACK 架构                          │
├─────────────────────────────────────────────────────────────────────────┤
│  入口层 (Gateway)                                                        │
│  ├── 渠道: 飞书 DM / 群聊 / 企业微信 / Webhook / CLI                     │
│  ├── 权限: 用户allowlist → 群allowlist → 命令allowlist → 工具allowlist   │
│  └── 审批: approvals.mode (DM可宽松，群聊需 smart/manual)                │
├─────────────────────────────────────────────────────────────────────────┤
│  情报层 (Tier 1 — Discovery)                                             │
│  ├── Search: Tavily / TinyFish MCP                                       │
│  ├── Fetch: web_extract → 静态页面                                       │
│  ├── Agent API: 反爬/动态页面 (TinyFish付费层)                           │
│  └── Platform API: 中文社媒专用通道                                      │
├─────────────────────────────────────────────────────────────────────────┤
│  执行层 (Tier 2 — Execution)                                             │
│  ├── 默认: Playwright 本地 Chromium (无状态)                             │
│  └── 生产: CDP 持久浏览器 (CloakBrowser / Kimi Bridge)                   │
│      ├── 持久 profile / 人工接管 / 指纹处理                               │
│      └── 6条操作规则: 复用/隔离/人工/低频/遇停/诚实                      │
├─────────────────────────────────────────────────────────────────────────┤
│  模型层 (Provider)                                                       │
│  ├── 主模型: Kimi-for-coding / MiniMax-M3 fallback                       │
│  ├── 辅助: MiniMax-M3 (compression等)                                    │
│  └── Prompt cache 优化: 稳定前缀 + 复用session = 高命中率                │
├─────────────────────────────────────────────────────────────────────────┤
│  记忆层 (Memory — 路由系统，非仓库)                                       │
│  ├── L0: MEMORY / USER → 启动热记忆                                      │
│  ├── L1: Session Search → 历史对话检索                                   │
│  ├── L2: Hindsight → 跨会话长期事实/语义检索                             │
│  ├── L3: Context Offloading → 长任务输出外置                             │
│  └── L4: Gateway Preflight → 任务前强制检查清单                          │
├─────────────────────────────────────────────────────────────────────────┤
│  任务层 (Execution)                                                      │
│  ├── delegate_task: 短任务分派                                           │
│  ├── kanban: 持久任务队列 (执行系统，非协作房间)                         │
│  └── cron: 定时任务                                                      │
├─────────────────────────────────────────────────────────────────────────┤
│  协作层 (Collaboration)                                                  │
│  ├── Phase 1: 对外一个 Bot，对内多角色协议                               │
│  ├── Phase 2: 多 Agent 房间 (Coordinator/Researcher/Builder/Reviewer)    │
│  └── 五大能力: 共享上下文/发言权调度/角色边界/可见争论/人类闸门           │
├─────────────────────────────────────────────────────────────────────────┤
│  观测层 (Observability)                                                  │
│  ├── logs: ~/.hermes/logs/gateway.log                                    │
│  ├── request_dump: 实际API请求追踪                                       │
│  └── session_search: 历史会话检索                                        │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 二、逐层关键洞察

### 2.1 入口层：Gateway 是 API Gateway，不是消息转发层

- **企业 IM 接入是权限工程，不是消息工程**
- 四层 allowlist：用户 → 群/频道 → 命令 → 工具
- DM vs 群聊风险差异：群聊是公共场，需 `require_mention: true`
- 审批模式：`smart` 或 `manual` 为群聊推荐，`false`（YOLO）仅限可信 DM

### 2.2 情报层：四层递进策略

```
Search 定位 → Fetch 抓普通页 → Agent API 处理反爬 → Platform API / 人工处理中文社媒
```

- **Search** (~0.7s): 发现 URL，免费额度 10K credits/月
- **Fetch** (~1s/URL): 提取静态内容，免费额度 10K credits/月
- **Agent API** (~50s+): 真实浏览器处理反爬/动态页面，付费
- **Platform API**: 小红书/知乎/微博/公众号专用通道

**常见情报层失败（看起来像"模型不聪明"）：**
- Search 返回旧文档 / Fetch 抓到摘要页
- 反爬失败却不报告 → 模型"假装"读过
- 多来源内容没有去重
- 未区分官方文档、论坛、社媒、营销号

### 2.3 执行层：浏览器工具后端选择

| 场景 | Tier 1 (web_extract) | Tier 2 (browser CDP) |
|:---|:---|:---|
| 读公开博客 | ✅ | Overkill |
| 查产品价格 | ✅ | Overkill |
| 操作已登录仪表盘 | ❌ | ✅ |
| 提交带 session cookie 的表单 | ❌ | ✅ |
| 人工验证工作流 | ❌ | ✅ (with human handoff) |

**CUA 三要素：** 看见真实界面、理解边界（何时自动/人工/只读）、留下证据

### 2.4 模型层：Prompt Cache 是 Agent 场景的成本杀手

**MiniMax-M3 实测（1周 3.3B tokens）：**
- Cache 命中率：92%（加权平均），峰值 ~99.9%
- 输入:输出:缓存读 = 1:0.09:11.6
- 缓存读按 ~0.1× 折扣计费
- **关键：不是 M3 便宜，是 Agent 场景天然适合 prompt cache**

**三要素复刻：**
1. OpenAI 兼容端点（`api.minimaxi.com/v1`）
2. 稳定 system prompt 和工具 schema
3. 复用 session（避免不必要的 `/reset`）

**约束对齐观察：** M3 是唯一主动引用 SOUL.md 名称并按 Change Control 流程执行的模型

### 2.5 记忆层：路由系统，非仓库

**五层框架：**

| 层 | 解决 | 不解决 |
|:---|:---|:---|
| MEMORY/USER | 启动热记忆 | 不能保证主动调用外部工具 |
| Session Search | 历史对话 | 不查就不会自动进入答案 |
| External Provider | 跨会话长期事实 | 不能替代任务前路由和事实核验 |
| Context Offloading | 长任务输出外置 | 不能自动判断哪些信息该进当前答案 |
| Gateway/Preflight | 任务前强制检查 | 不能替代来源真实性和人工边界 |

**核心幻觉：** "写进 MEMORY 就会去查" — 模型不是规则引擎，MEMORY 只是背景提示

**Preflight 检查清单（推荐实现）：**

| 任务类型 | 检查项 |
|:---|:---|
| 写作 | 写作偏好、系列上下文、外部事实来源 |
| 代码续写 | 项目目标/边界、上次进度、不可改文件、必跑测试、历史坑 |
| 命令/接入方案 | 当天官方文档、区分"官方支持"vs"社区方案"、明确日期版本 |

### 2.6 任务层：Kanban 是任务队列，不是协作房间

| 维度 | 任务队列 (Kanban) | Agent Team |
|:---|:---|:---|
| 核心 | 拆任务、并发、汇总 | 共享空间中的协作过程 |
| 上下文 | 各干各的，prompt 隔离 | 共享讨论过程 |
| 交互 | 单向流转 | 可见的争论、验证、追问 |
| 人类角色 | 节点介入 | 随时插话、批准、否决 |

**MVP 角色设计：**
- Coordinator: 拆任务、控节奏、收敛结论
- Researcher: 搜索、抓取、整理证据
- Builder: 生成草稿、代码、文件
- Reviewer: 挑错、找风险、要求补证据
- Human: 随时插话、批准、否决、改方向

### 2.7 协作层：多 Agent 阶段化

| 阶段 | 架构 | 时机 |
|:---|:---|:---|
| Phase 1 | 对外一个 Bot，对内多角色协议 | 生产环境 — 稳定、可审计 |
| Phase 2 | 多个真实 bot 带 @ 路由 | 实验性 — 解决上下文/权限/恢复后 |

**CUA 在 Agent Team 中的定位：**
- API 可用 → 用 API
- 可结构化 → 结构化
- 能让人确认 → 让人确认
- CUA 仅用于：无 API、无结构化、低风险、可回滚、可人工盯住

---

## 三、本轮学习来源清单

| # | 文章标题 | 核心主题 | 吸收状态 |
|:---|:---|:---|:---|
| 1 | MiniMax-M3直接塞进生产级Hermes | 成本分析、prompt cache | ✅ 已吸收 |
| 2 | MiniMax-M3一周实测：49元套餐扛住3.3亿Token | 量化验证、cache 命中率 | ✅ 已吸收 |
| 3 | 给 Hermes 接入 Hindsight 记忆系统 | 记忆层实战、6条排障 | ✅ 已吸收 |
| 4 | Hermes 浏览器工具升级：从 Playwright 到 CloakBrowser | CUA 实践、两段式架构 | ✅ 已吸收 |
| 5 | 我把 Hermes 接进企业微信和飞书后 | Gateway 治理、权限工程 | ✅ 已吸收 |
| 6 | 我把 Hermes 的搜索和抓取换成 TinyFish | 情报层四层架构 | ✅ 已吸收 |
| 7 | Hermes Agent 记忆层深度分析 | 记忆路由系统、Preflight | ✅ 已吸收 |
| 8 | 本地 Agent Team Runtime | 多 Agent 协作、三层模型 | ✅ 已吸收 |

---

## 四、开放跟踪项（需后续研究/验证）

> 信息是为了打开思路，以下项目无需立即执行，但应在遇到相关场景时主动回顾

| # | 项目 | 来源文章 | 状态 | 后续触发条件 |
|:---|:---|:---|:---|:---|
| 1 | **Preflight 任务前检查机制** | #7 | 概念已吸收，待实现 | 下次写作/代码/命令任务时，主动添加检查清单 |
| 2 | **TinyFish 设为默认搜索后端** | #6 | `mcp_servers.tinyfish.enabled: true` 已配置，但 `web.backend: tavily` | 当 Tavily 搜索质量不满足时切换 |
| 3 | **CDP 持久浏览器部署** | #4 | 概念已吸收，未部署 | 需要操作已登录仪表盘或复杂表单时 |
| 4 | **Agent Team 房间原型** | #8 | 概念已吸收，未实验 | 任务复杂度超过单 Agent 处理能力时 |
| 5 | ** approvals.mode 群聊适配** | #5 | 当前 `false`，DM-only 风险可控 | 扩展飞书群聊接入时必改 |
| 6 | **Hindsight 轻量 retain 后端** | #3 | 概念已吸收，未配置独立后端 | retain 超时再次出现 |
| 7 | **MiniMax-M3 辅助任务全切** | #1/#2 | compression 已切，vision/web_extract 未切 | 主模型验证满 1 周后 |
| 8 | **中文社媒 Platform API** | #6 | 未探索 | 需要抓取小红书/微博深度内容时 |

---

## 五、已沉淀的技能位置

| 技能文件 | 新增/修改内容 |
|:---|:---|
| `~/.hermes/skills/hermes-agent/SKILL.md` | Two-tier intelligence、Browser Tool Backends、Gateway governance、Memory layer governance、Agent Team vs Task Queue |
| `~/.hermes/skills/llm-provider-integration/provider-quirks.md` | MiniMax-M3 cache 优化、约束对齐、生产 rollout、文档导航 |

---

*综合日期: 2026-06-07*
*学习原则: 信息为打开思路，非停止研究之理由*

---

## 六、当前配置风险点与调整命令

### 风险 1: approvals.mode = false（全局 YOLO）

**现状**: 任何白名单用户可免确认触发任何工具
**影响**: 当前仅飞书 DM + 2 用户，风险可控；扩展群聊时必须调整
**调整命令**:
```bash
# 方案 A: 智能审批（推荐）
hermes config set approvals.mode smart

# 方案 B: 手动审批（更安全，但交互频繁）
hermes config set approvals.mode manual
```
**生效**: 需 `/reset` 或重启 gateway

---

### 风险 2: web.backend = tavily（TinyFish 未启用为默认）

**现状**: `mcp_servers.tinyfish.enabled: true` 已配置，但搜索仍走 Tavily
**影响**: TinyFish 的 Search/Fetch 免费额度未充分利用
**调整命令**:
```bash
# 查看当前配置
hermes config | grep -A2 'web:'

# 如需切换（确认 Tavily 不满足后再执行）
hermes config set web.backend tinyfish
hermes config set extract_backend tinyfish
```
**注意**: TinyFish Agent API 为付费层，免费额度仅覆盖 Search/Fetch

---

### 风险 3: browser.cdp_url = ''（无持久浏览器）

**现状**: 使用默认 Playwright，每会话新开浏览器，无持久状态
**影响**: 无法操作已登录页面
**调整命令**:
```bash
# 1. 启动 CloakBrowser（需先安装）
# cloakbrowser --cdp-port=9222 --profile-dir=~/.cloak/hermes

# 2. 配置 Hermes 连接
hermes config set browser.cdp_url http://127.0.0.1:9222
```
**前提**: 需先安装 CloakBrowser 或 Kimi Bridge

---

### 风险 4: 辅助任务 provider 未统一

**现状**: compression 已切 MiniMax-M3，vision/web_extract 仍为 auto
**影响**: 可能 fallback 到不稳定或高成本后端
**调整命令**:
```bash
# 统一辅助任务到已验证的 MiniMax-M3
hermes config set auxiliary.vision.provider minimax-cn
hermes config set auxiliary.vision.model MiniMax-M3
hermes config set auxiliary.web_extract.provider minimax-cn
hermes config set auxiliary.web_extract.model MiniMax-M3
```
**前提**: 需确认主模型 M3 稳定运行满 1 周（Phase 1 完成）

---

### 风险 5: Hindsight retain 后端未独立配置

**现状**: retain/consolidation 可能调用主模型（Kimi/MiniMax），存在超时风险
**影响**: 长输入时 retain 可能 300s+ 超时
**调整命令**:
```bash
# 在 ~/.hindsight/profiles/hermes.env 中配置轻量后端
# HINDSIGHT_LLM_MODEL=gpt-4o-mini 或等效 Flash 模型
# HINDSIGHT_LLM_BASE_URL=...
# HINDSIGHT_LLM_API_KEY=*** 然后重启 hindsight daemon
```
**注意**: 需确认轻量模型可用且成本可控

---

*配置调整日期: 2026-06-07*
*执行原则: 按需调整，非全量执行*
