---
title: Hermes 本地关键系统跟进清单
created: 2026-06-06
updated: 2026-06-06
type: system
domain: hermes-ops
tags: [hermes, maintenance, tooling]
sources: []
confidence: medium
---

# Hermes 本地关键系统跟进清单

> **用途**：明确每个关键系统的跟进源、频率、触发条件和失败兜底，把"维护"从抽象承诺变成具体动作。
> **维护原则**：先官方 docs/GitHub → 本机 runtime 验证；外部实践只作发现层，不替代官方边界校验。
> **更新频率**：本清单本身每季度 review 一次，或当任一系统出现 breaking change 时立即更新。

---

## 系统总览

| # | 系统 | 当前版本/状态 | 风险等级 | 跟进频率 | 负责方式 |
|---|------|-------------|---------|---------|---------|
| 1 | Hermes Agent 主仓库 | v0.15.1 (2026.5.29)，已同步 origin/main | **高** | 每次 release | 手动 rebase + 验证 |
| 2 | yt-dlp | 2026.03.17 (brew) | **中高** | 每月检查 | brew upgrade + 功能验证 |
| 3 | notebooklm-py | v0.3.4-hermes.4 (fork，含安全审计) | **高** | 每次 upstream release | 强制安全审计后合并 |
| 4 | akshare | v1.18.64 (已安装) | 中 | 按需 | pip install -U |
| 5 | OpenAI SDK | v2.24.0 (Hermes 内置) | 中 | 每次 Hermes 更新时顺带检查 | Hermes 更新流程覆盖 |
| 6 | baoyu-skills 套件 | 24 个 skills，部分有 upstream | 中 | 每季度 | hermes skills update |
| 7 | Hermes 系统级 skills | 6 个（自维护） | 低 | 按需 | 自维护 |
| 8 | Gateway / 本地基础设施 | 127.0.0.1:8642，运行正常 | 中 | 出问题才看 | 监控告警 |

---

## 逐项明细

### 1. Hermes Agent 主仓库

| 属性 | 内容 |
|------|------|
| **跟进源** | GitHub: `NousResearch/hermes-agent` |
| **Release 频道** | <https://github.com/NousResearch/hermes-agent/releases> + 官方 Changelog <https://hermes-ai.net/changelog> |
| **本地路径** | `~/.hermes/hermes-agent` |
| **当前状态** | v0.15.1，fork 在 `foras910521-lab/hermes-agent`，origin/main 已同步 |
| **跟进频率** | **每次官方 release**（通常 2-4 周一次） |
| **触发条件** | GitHub release notification / 月度主动检查 / 功能异常怀疑是 upstream bug |
| **跟进方式** | 1. Watch GitHub release<br>2. 本地 `git fetch origin` → `git log HEAD..origin/main --oneline`<br>3. 阅读 changelog，判断 breaking change 风险<br>4. 创建 backup branch: `git branch backup/pre-update-$(date +%Y%m%d)`<br>5. Rebase 或 merge origin/main<br>6. 运行 `hermes --version` 验证<br>7. 执行 smoke test（启动 gateway，做一次简单对话） |
| **失败兜底** | - Backup branch 可立即回滚: `git reset --hard backup/pre-update-YYYYMMDD`<br>- 若 upstream 不可达，用现有 backup branch 继续运行<br>- 严重故障时参考 `~/.hermes/audits/hermes-full-heal-20260510/RETROSPECTIVE.md` |
| **特殊注意** | - 当前有多个 codex/* 分支和 backup 分支，更新前确认当前在 main<br>- 更新后检查 config.yaml 是否有新字段需要配置 |

---

### 2. yt-dlp

| 属性 | 内容 |
|------|------|
| **跟进源** | GitHub: `yt-dlp/yt-dlp` releases |
| **当前版本** | 2026.03.17 (via Homebrew) |
| **本地使用路径** | `/opt/homebrew/bin/yt-dlp` |
| **使用场景** | YouTube 内容下载（`youtube-content` skill + `baoyu-youtube-transcript` skill） |
| **跟进频率** | **每月检查一次**（yt-dlp 更新频繁，但 YouTube 接口变更时才需要紧急更新） |
| **触发条件** | - YouTube 下载失败且怀疑是 extractor 过时<br>- 每月 1 号主动检查 `brew outdated yt-dlp`<br>- GitHub release notification（可选） |
| **跟进方式** | `brew upgrade yt-dlp` → 用任意 YouTube URL 测试下载/转录功能 |
| **失败兜底** | - yt-dlp 有内置更新机制: `yt-dlp -U`（绕过 brew 直接更新二进制）<br>- 若 YouTube 全面封禁，切换至 `youtube-transcript-api` 纯 API 路径（已配置在 skills 中） |
| **特殊注意** | - 两个 skill 都依赖 yt-dlp，更新后需同时验证 `youtube-content` 和 `baoyu-youtube-transcript`<br>- yt-dlp 更新可能伴随 Python 依赖变更，注意检查 |

---

### 3. notebooklm-py（高优先级安全组件）

| 属性 | 内容 |
|------|------|
| **跟进源** | GitHub: `teng-lin/notebooklm-py` |
| **当前版本** | v0.3.4-hermes.4（fork 维护，含安全审计文档） |
| **本地路径** | Skill: `~/.hermes/skills/productivity/notebooklm/` |
| **风险等级** | **高** — 处理 Google 账号凭据，有安全审计要求 |
| **跟进频率** | **每次 upstream 发布新版本时**，但**必须经过安全审计** |
| **触发条件** | - upstream 新 release tag<br>- 功能异常需要 bugfix<br>- 安全公告 |
| **跟进方式** | **强制流程**：<br>1. 读取 `SECURITY_AUDIT.md` 中的 Upgrade Protocol<br>2. `git fetch upstream`<br>3. `git log --oneline HEAD..upstream/main -- src/`<br>4. `git diff HEAD..upstream/main -- src/`<br>5. 重新执行安全扫描（eval/exec/subprocess/hosts/env/file writes）<br>6. 若通过，合并并打新 tag `vX.Y.Z-hermes.N+1`<br>7. 更新 `SECURITY_AUDIT.md`<br>8. 重新登录并 `chmod 600 ~/.notebooklm/storage_state.json` |
| **失败兜底** | - 安全审计不通过 → **冻结升级**，继续使用当前版本<br>- 当前版本已稳定运行，不升级不影响现有功能<br>- 使用 burner Google account，主账号不受影响 |
| **特殊注意** | - **绝对禁止** `pip install -U notebooklm-py` 或 `hermes skills update notebooklm --force`<br>- 审计文档必须随版本同步更新<br>- `NOTEBOOKLM_REFRESH_CMD` 环境变量已配置，注意检查其安全性 |

---

### 4. akshare

| 属性 | 内容 |
|------|------|
| **跟进源** | GitHub: `akfamily/akshare` |
| **当前版本** | **v1.18.64**（已安装，2026-06-02） |
| **使用场景** | 股票数据获取（A股日线、实时行情等） |
| **跟进频率** | **按需** — 数据接口异常时检查更新 |
| **触发条件** | - 数据获取失败（如 `stock_zh_a_spot_em` 等接口返回异常）<br>- 需要使用新接口或新功能 |
| **跟进方式** | `pip install -U akshare` → 验证核心接口（如 `stock_zh_a_daily`） |
| **失败兜底** | - 数据源变更时回退到备用数据源<br>- 或切换到 `yfinance`/`alpha_vantage` 等备选 |
| **特殊注意** | - 东方财富部分接口（`stock_zh_a_spot_em`）可能因服务端限制返回空响应，属正常现象<br>- 新浪财经接口（`stock_zh_a_spot`）可能返回 HTML 而非 JSON，需关注 akshare 更新修复<br>- 当前 `stock_zh_a_daily` 等接口工作正常 |

---

### 5. OpenAI SDK / 模型 Provider 层

| 属性 | 内容 |
|------|------|
| **跟进源** | PyPI: `openai` + Hermes 内置依赖 |
| **当前版本** | v2.24.0（Hermes v0.15.1 内置） |
| **跟进频率** | **随 Hermes 更新顺带检查** |
| **触发条件** | - Hermes 更新后 API 行为变化<br>- OpenAI 平台公告新功能/弃用 |
| **跟进方式** | Hermes 更新流程已覆盖 — 更新后做一次模型调用 smoke test |
| **失败兜底** | - 多 provider 配置已就绪（kimi-coding → minimax-cn fallback）<br>- 单个 provider 故障时自动切换 |
| **特殊注意** | - config.yaml 中 `fallback_providers` 已配置，保持更新时检查有效性 |

---

### 6. baoyu-skills 套件

| 属性 | 内容 |
|------|------|
| **跟进源** | GitHub: `JimLiu/baoyu-skills`（部分 skill 可能有独立 upstream） |
| **当前版本** | 24 个 skills，2026-05-23 左右安装 |
| **高风险 skills** | `baoyu-danger-gemini-web`、`baoyu-danger-x-to-markdown`（名称带 danger，注意合规） |
| **跟进频率** | **每季度检查一次** |
| **触发条件** | - 功能异常<br>- upstream 重大更新<br>- 季度维护窗口 |
| **跟进方式** | `hermes skills update` 或手动 `git pull` 各 skill 目录 → 验证核心功能 |
| **失败兜底** | - 单个 skill 故障不影响其他 skill<br>- 可临时禁用问题 skill（移出 skills 目录） |
| **特殊注意** | - 部分 skills 依赖 bun/npx，更新后检查运行时可用性<br>- `baoyu-imagine` 等 image gen skills 注意 API 配额和费用 |

---

### 7. Hermes 系统级 skills（自维护）

| 属性 | 内容 |
|------|------|
| **清单** | `hermes-memory-knowledge-operations`、`hermes-model-provider-operations`、`hermes-runtime-operations`、`hermes-skills-maintenance`、`hermes-web-ui-operations`、`darwin-skill` |
| **跟进频率** | **按需** — 自己维护，无 upstream |
| **维护方式** | - 使用过程中发现问题 → 现场 patch<br>- 复杂流程沉淀为 skill 更新<br>- 每季度 review 一次是否过时 |
| **失败兜底** | - 自维护，无外部依赖风险<br>- 保留在 git 中（建议）或定期备份 |

---

### 8. Gateway / 本地基础设施

| 属性 | 内容 |
|------|------|
| **组件** | Gateway (127.0.0.1:8642)、Langfuse (<NAS_HOST_PRIVATE_IP>:3300)、Feishu websocket、NAS ZOS |
| **跟进频率** | **出问题才看**（监控驱动） |
| **触发条件** | - 服务不可达<br>- 日志异常<br>- 性能下降 |
| **跟进方式** | - Gateway: `hermes gateway status` 或查看日志<br>- Langfuse: Web UI 检查 + NAS SSH 检查容器状态<br>- Feishu: 消息送达失败时检查 websocket 连接<br>- NAS: 21:30 只读简报，不主动改动 |
| **失败兜底** | - Gateway: 重启 `hermes gateway start`<br>- Feishu: 检查代理设置 `127.0.0.1:6984`<br>- NAS: SSH 免密登录，按 handoff 文档操作 |
| **特殊注意** | - **NAS 影库/PT 配置**：无明确确认不删媒体、不重建容器/库、不改 PT/下载站配置<br>- Langfuse 凭据仅本地保存，不进记忆/输出 |

---

## 维护日历（建议）

| 频率 | 动作 | 负责系统 |
|------|------|---------|
| **每周** | 检查 Hermes GitHub release notification | #1 Hermes Agent |
| **每月 1 号** | `brew outdated yt-dlp` + 功能测试 | #2 yt-dlp |
| **按需** | 故障排查、安全公告响应、akshare 接口异常时更新 | 全部 |

---

## Cron 提醒配置（已移除）

> 用户决定不需要定期 cron 提醒。维护改为事件驱动：出问题才处理，不主动打扰。
> 
> 如需恢复，可手动创建：
> ```bash
> hermes cron create --name "monthly-system-check" \
>   --schedule "0 10 1 * *" \
>   --prompt "检查 yt-dlp 和 Hermes 是否需要更新"
> ```

---

## 变更日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-06-02 | v1.0 | 初始版本，基于当前系统状态创建 |
| 2026-06-02 | v1.1 | 更新 akshare 状态为已安装(v1.18.64)，补充接口异常说明；移除季度 cron 建议 |
