---
title: Hermes 跨平台信息同步治理方案
created: 2026-06-05
updated: 2026-06-06
type: system
domain: hermes-ops
tags: [hermes, maintenance, routing, memory]
sources: []
confidence: medium
---

# Hermes 跨平台信息同步治理方案

## 基于官方机制 + 2026-06-05 诊断发现

---

## 一、问题本质

### 1.1 官方配置优先级 vs 源码实际实现

**官方文档声明**：
```
1. CLI 参数
2. ~/.hermes/config.yaml（主配置）
3. ~/.hermes/.env
4. 内置默认值
```

**源码实际**（hermes_cli/auth.py resolve_provider()）：
```
1. auth.json active_provider ← 实际最高优先级！
2. CLI 显式参数
3. 环境变量
4. 特定 provider API key
5. fallback: openrouter
```

**冲突点**：auth.json active_provider 静默覆盖 config.yaml，导致系统行为与配置声明不一致。

### 1.2 发现的多源冲突

| 信息层 | 网关端 | 桌面端 | 问题 |
|--------|--------|--------|------|
| 模型路由 | auth.json 优先 | auth.json 优先 | 统一但优先级错误 |
| 声明模型 | kimi-for-coding | kimi-for-coding | 一致 |
| 实际生效 | openai-codex | openai-codex | ❌ 覆盖声明 |
| Fallback | MiniMax-M3 | MiniMax-M3 | 已修复 |
| Hindsight | MiniMax-M3 | MiniMax-M3 | 已修复 |

### 1.3 记忆层冲突

- MEMORY.md：76% 使用率，含过时模型记录
- USER.md：历史偏好可能与新配置冲突
- Hindsight：14条过时记录已标注
- Skills：分散重复
- Wiki：与 Memory 边界模糊

---

## 二、官方机制解读

### 2.1 Memory 设计

> "Memory loads at session start only and never changes mid-session"
> "When full, the agent consolidates or replaces entries"
> **Agent-owned，not user-owned**

- MEMORY.md: 2,200 chars
- USER.md: 1,375 chars
- 变更立即持久化，但**下一会话才生效**

### 2.2 Config 设计

> "Secrets → .env. Everything else → config.yaml."
> "hermes config set automatically routes values to the right file"

- config.yaml: 非密钥配置
- .env: API keys
- auth.json: OAuth 状态（由 hermes auth 管理）

### 2.3 Skills 设计

> "Autonomous skill creation and self-improvement during use"

- Procedural memory（流程知识）
- 跨会话持久
- 自动改进

---

## 三、治理架构

### 3.1 配置层：Single Source of Truth

```
┌─────────────────────────────────────────┐
│         权威源：config.yaml              │
└─────────────────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ auth.json│ │ .env    │ │ Hindsight│
   │ (必须同步)│ │ (密钥)  │ │ (快照)   │
   └─────────┘ └─────────┘ └─────────┘
```

**规则**：
1. config.yaml 为唯一权威源
2. auth.json active_provider 必须同步 config.yaml
3. 禁止两者不一致

### 3.2 记忆层：四层分层

```
Layer 1: Built-in Memory（高频、短、当前）
  MEMORY.md: 环境事实、当前配置
  USER.md: 用户偏好、沟通风格

Layer 2: Skills（流程知识）
  可复用工作流、诊断步骤

Layer 3: Hindsight（长期事实）
  历史记录、跨会话检索

Layer 4: Wiki（结构化文档）
  项目文档、研究笔记
```

**存放规则**：

| 信息类型 | 存放位置 | 理由 |
|---------|---------|------|
| 当前模型配置 | MEMORY.md | 高频使用 |
| 用户偏好 | USER.md | 每会话注入 |
| 诊断流程 | Skills | 可复用 |
| 历史记录 | Hindsight | 自动管理 |
| 项目文档 | Wiki | 结构化 |
| 过时记录 | 标注[过时]或删除 | 避免干扰 |

### 3.3 跨平台同步

**网关端（Feishu）和桌面端（Desktop）**：

1. **配置共享**：同一 config.yaml / auth.json / .env
2. **运行时隔离**：gateway run vs Desktop app，环境变量可能不同
3. **状态同步**：
   - Memory：文件级共享，自动同步
   - Hindsight：数据库共享，自动同步
   - Skills：文件系统共享，自动同步
4. **冲突解决**：以 config.yaml 为准

---

## 四、自动化治理

### 4.1 配置变更流程

```
修改 config.yaml
    ↓
同步 auth.json active_provider
    ↓
更新 Hindsight 快照（如需要）
    ↓
运行 config_sync_check.py
    ↓
更新 Memory
    ↓
完成
```

### 4.2 定期维护

- **每6小时**：config_sync_check.py 自动运行
- **每会话开始**：一致性检查
- **每月**：Skills 去重、Hindsight 清理

---

## 五、关键原则

1. **官方优先**：官方 docs + GitHub source 为权威
2. **Live Verification**：不以 Memory 为状态真理
3. **Agent-Owned Memory**：Agent 负责维护，用户不手动管理
4. **Single Source of Truth**：config.yaml 为运行时权威

---

## 六、验证命令

```bash
# 配置一致性
python3 ~/.hermes/scripts/config_sync_check.py

# Provider 状态
hermes auth status kimi-coding
hermes auth status minimax-cn

# Config 检查
hermes config show

# Memory 状态
hermes memory status

# Hindsight 健康
curl http://127.0.0.1:9177/health

# Cron 任务
hermes cron list --all
```

---

*生成时间: 2026-06-05 | 版本: v1.0*
