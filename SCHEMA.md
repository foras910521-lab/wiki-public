# LLM Wiki Schema

## Domain
这是 foras 的统一 LLM Wiki / Obsidian Vault。它合并原三个 wiki，让知识在一个图里维护、检索、互证：

- `stock`：股票研究、交易方法、个股/主题/案例。
- `hermes-ops`：Hermes、agent、wiki 运维、模型、工具、记忆、运行经验。
- `self-media`：自媒体、视觉、Lovart、内容生产。
- `cross-domain`：跨领域原则、方法论、查询归档。

## Official Alignment
本库采用 Karpathy LLM Wiki / Hermes llm-wiki 的单一 vault 心智：

```text
LLM Wiki/
├── SCHEMA.md
├── index.md
├── log.md
├── raw/
├── entities/
├── concepts/
├── comparisons/
├── queries/
├── cases/
├── sources/
└── _meta/
```

领域隔离不再靠多个物理 Wiki，而靠路径命名空间、frontmatter `domain`、tags 和 wikilinks。

## Conventions
- Obsidian 是 IDE；LLM 是维护者；Wiki 是代码库。
- 每次新增/更新正式页面，必须同步更新 `index.md` 与 `log.md`。
- 优先使用 `[[wikilinks]]` 连接相关页面，避免跨库 plain text 引用。
- 关键事实、数字、日期、百分比、具体结论必须回到 `raw/` 或 `sources/`，保留精确出处。
- `raw/` 是证据层，默认不可覆盖原文；只允许新增、补 frontmatter、补索引说明。
- “学习有用的部分”优先落为 `queries/<domain>/...` 或更新已有 concept；不要散成很多薄页面。
- 如果一个原则会影响多个领域，放入 `concepts/cross-domain/` 或 `queries/cross-domain/`，并链接到相关领域页面。

## Frontmatter
```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: raw | source | entity | concept | comparison | query | case | tool | system | project | meta
domain: stock | hermes-ops | self-media | cross-domain
tags: []
sources: []
confidence: high | medium | low
---
```

## Tag Taxonomy
- Domain: `stock`, `hermes-ops`, `self-media`, `cross-domain`, `relationship-psychology`
- Knowledge Ops: `llm-wiki`, `obsidian`, `provenance`, `maintenance`, `lint`, `drift`, `memory`, `skill`, `routing`
- Stock: `stock-research`, `trading-system`, `market-wizards`, `minervini`, `oneil`, `case-study`, `screening`
- Self Media: `content-operations`, `visual-system`, `lovart`, `image-generation`, `xhs`, `xiaohongshu`, `douyin`, `scriptwriting`, `public-account`, `radar`, `hyperframes`, `video-pipeline`, `batch-production`
- Agent/Ops: `hermes`, `agent-runtime`, `provider`, `tooling`, `gateway`, `cron`, `evaluation`, `observability`
- Tools/Workflow: `crawler`, `mediacrawler`, `workflow`, `template`, `content-intelligence`
- Extended observed tags: `2018-outlook`, `a-share`, `account-analysis`, `adk`, `adoption-decision`, `adult-consent`, `agent`, `agent-lightning`, `agent-radar`, `agent-skills`, `agent-swarm`, `agentic-rag`, `ai-efficiency`, `akshare`, `animation`, `anti-ai`, `appendix`, `architecture`, `archive`, `archive-rule`, `archive-rules`, `asset-management`, `asset-production`, `attachment-theory`, `audit`, `authority-citation`, `azure-ai-search`, `b-class`, `backfill`, `backlog`, `batch-03`, `batch-04`, `bdsm-subtext`, `benchmark`, `benchmarking`, `book-distillation`, `brand-kit`, `browser-automation`, `bundles`, `candidate-universe`, `canslim`, `capabilities`, `case`, `case-extractor`, `cdp`, `checklist`, `citation-migration`, `citation-strategy`, `closure`, `closeout`, `codex`, `codex-image2`, `coding-agent`, `color-system`, `column-matrix`, `comment-analysis`, `comment-corpus`, `comparators`, `competitor-analysis`, `component-system`, `comfyui`, `conardli`, `concept`, `configuration`, `content-analysis`, `content-health`, `continuation`, `copy-polish`, `copywriting`, `cover-design`, `cpo`, `credential`, `credit-safety`, `css-3d`, `css-blur`, `css-cover`, `css-destruction`, `css-dissolve`, `css-distortion`, `css-grid`, `css-light`, `css-mechanical`, `css-other`, `css-patterns`, `css-push`, `css-radial`, `css-scale`, `cycle-stocks`, `dashboard`, `dark-psychology-silhouette`, `data-analysis`, `data-loop`, `data-pipeline`, `data-request`, `datafication`, `david-ryan`, `deployment`, `dependency`, `desktop`, `deterministic-scan`, `diagnosis`, `disagreement`, `distillation`, `ds-boundary`, `ds-system`, `earnings`, `eastmoney`, `efficiency`, `emotional-tension`, `entity`, `escalation`, `evidence`, `evidence-closure`, `evidence-review`, `exact-id`, `execution`, `experiment`, `experiments`, `external-memory`, `external-support`, `feishu`, `ffmpeg`, `fiber`, `field-mapping`, `finance-agent`, `first-month-plan`, `framework`, `frontmatter`, `generator`, `git`, `github`, `github-pr`, `governance`, `governance-lite`, `gpt-image`, `gpt-image-2`, `graph-database`, `graph-rag`, `green-v3`, `harness`, `hash`, `hermes-skills`, `high-value`, `hindsight`, `holdings`, `hook-bank`, `hotspot`, `image-models`, `image2`, `implementation-plan`, `implicit-screening`, `imported`, `index`, `infographic`, `installation-decision`, `intent-calibration`, `interface`, `inversion`, `investing`, `investment-research`, `john-bowlby`, `keyword-seeding`, `kickoff`, `kimi`, `knowledge-base`, `knowledge-creator`, `knowledge-explainer`, `knowledge-graph`, `knowledge-ops`, `langfuse`, `launch`, `layout-pack`, `leader-stock`, `learning-map`, `learning-plan`, `library`, `linzhong-lichtung`, `livingston`, `llm-agent`, `llm-wiki`, `long-page`, `long-pages`, `lovart-brief`, `low-risk-entry`, `male-persona`, `mark-minervini`, `market-card`, `market-first`, `market-leader`, `market-map`, `market-regime`, `market-theme`, `marketplace`, `master-template`, `mcp`, `mem0`, `method-index`, `midjourney`, `model-provider`, `model-routing`, `model-test`, `multi-persona`, `multimedia-learning`, `multimodal`, `mx`, `n8n`, `nano-banana`, `nas`, `navigation`, `network`, `night`, `night-reading`, `notebooklm`, `nousresearch`, `nuwa`, `object-theatre`, `official-alignment`, `official-followup`, `openclaw`, `opencode`, `operating-baseline`, `operating-plan`, `operating-system`, `operation-plan`, `operations`, `optical`, `orchestration`, `outline`, `over-independent`, `paper-diagram`, `paper-today`, `papertoday`, `partner-responsiveness`, `person`, `persona`, `personality`, `phase-gate`, `phase4`, `philosophy`, `pipeline`, `plan`, `platform-learning`, `platform-mechanism`, `platform-safety`, `plugin`, `plugin-hub`, `policy`, `portfolio`, `positioning`, `post-close`, `postmortem`, `pre-open`, `production`, `production-pipeline`, `production-workflow`, `prompt-engineering`, `prompt-governance`, `prompt-size`, `protocol`, `pruning`, `psd`, `psychology`, `public-account-followup`, `public-account-writing`, `quality-control`, `quant`, `query`, `query-archive`, `rag`, `raw`, `readiness`, `reference-audit`, `reference-analysis`, `reference-teardown`, `reinforcement-learning`, `relative-strength`, `reopen`, `research`, `research-bundle`, `research-package`, `research-session`, `retrospective`, `review`, `reviewer`, `risk-control`, `rl`, `roadmap`, `root-cause`, `roundtable`, `ryan`, `runtime`, `safety`, `sampling`, `scene-design`, `schwager`, `script`, `script-skeleton`, `script-v1-1`, `script-v1-2`, `script-v1-3`, `script-v1-4`, `scribe`, `secretary`, `sector-map`, `secure-base`, `security`, `seedance-2`, `seedream`, `self-check`, `self-evolution`, `self-improvement`, `sepa`, `sha256`, `short-video`, `shortlist`, `simplemem`, `simplified-plan`, `skill-design`, `skill-evolution`, `skill-governance`, `skill-graph`, `skill-learning`, `skill-workflow`, `skillclaw`, `skills`, `slow-scan`, `snapshot`, `soul`, `source`, `source-backed`, `source-evidence`, `source-governance`, `source-pack`, `source-page`, `source-whitelist`, `sources`, `split`, `stability-detector`, `standalone`, `standardized-draft`, `stock-analysis`, `stock-research`, `stock-roundtable`, `stock-wiki`, `stocks`, `strict-screen`, `style-study`, `subtitles`, `systemic-thinking`, `taxonomy`, `template`, `terminal-agent`, `theme`, `tianfu`, `token-budget`, `tool-wrapper`, `toolset`, `toolset-slimming`, `trend-template`, `triage`, `tts`, `tutorial`, `typography`, `update`, `upgrade`, `validation`, `vault`, `vault-ops`, `vcp`, `vector-animation`, `verification`, `video`, `video-pipeline`, `video-teardown`, `video-workflow`, `visual-brief`, `visual-design`, `visual-directions`, `visual-production`, `visual-quality`, `visual-style`, `voiceover`, `voiceover-script`, `warm-professional`, `watchdog`, `watchlist`, `wechat`, `wechat-learning`, `wechat-official-account`, `wechat-radar`, `week2`, `whisper`, `whitelist`, `wiki`, `work001`, `workflow`, `synthesis`

## Maintenance
- Weekly lint: broken wikilinks, orphan pages, missing index entries, stale pages, duplicate concepts。
- Monthly drift audit: sample ~5 summary/query pages and compare claim-by-claim against raw/source evidence。
- Scope finding is deterministic: use search/tags/metadata first, then let LLM reason inside the narrowed set。

## Migration Note
本库最初于 2026-05-04 从旧机器 `<LEGACY_HOME>/Documents/LLM Wiki` 合并形成统一 vault。

2026-06-06 新机器恢复时，从迁移包 `restore-safe/projects/LLM Wiki/` 恢复到当前路径：

```text
<HERMES_PROJECTS>/wiki
```

旧路径 `<LEGACY_HOME>/...` 在部分 `raw/`、`log.md`、`_archive/`、`_imports/`、历史 source metadata 中作为 provenance 保留，不代表当前可执行路径。工具/运行手册中的可执行路径必须单独验证并翻译到当前机器。


## Official LLM Wiki Operating Rules

对齐 Hermes 官方 `llm-wiki` skill：本库是可由 Obsidian/VS Code 直接打开的 Markdown Wiki，不依赖数据库；`raw/` 为不可变证据层，`entities/`、`concepts/`、`comparisons/`、`queries/` 为 Agent 维护的综合层，`SCHEMA.md` 是约束层。

### Page Thresholds
- **Create a page** when an entity/concept appears in 2+ sources OR is central to one source.
- **Add to existing page** when a source mentions something already covered.
- **Do not create a page** for passing mentions, minor details, or material outside the active domains.
- **Split/watch long pages** when a page exceeds ~200 lines; do not rewrite large pages automatically without a focused task.
- **Archive** only when content is fully superseded and links/index/log can be updated safely.

### Raw Source Frontmatter
Raw sources should include normal vault frontmatter plus source tracking when practical:

```yaml
source_url: https://example.com/source
ingested: YYYY-MM-DD
sha256: <hex digest of body after frontmatter>
```

`sha256` is a drift signal, not a reason to overwrite raw evidence automatically. Existing historical raw files may lack hashes; new important raw captures should include them.

Canonical local-body `sha256` is defined as `SHA256(UTF-8 bytes of body after YAML frontmatter, with leading newlines stripped)`. Keep separate contracts separate: use `sha256_original_text` when the intended hash covers only an extracted original-text section. Do not backfill or overwrite hashes for upstream skill/repository captures unless the wrapper-vs-source boundary is explicit.

### Provenance and Confidence
- Multi-source synthesis pages should link or cite raw/source evidence close to important claims.
- Use `confidence: low` for weak fragments/search-only evidence, `medium` for single-source or fast-moving claims, and `high` only for source-backed stable conclusions.
- Do not let low-confidence snippets harden into accepted facts without later corroboration.

### Update Policy
When new information conflicts with existing content:
1. Check dates and source authority.
2. Preserve both claims when the conflict is real.
3. Add `contested: true` / `contradictions:` when appropriate.
4. Surface the issue in a lint/governance report instead of silently overwriting.

### Lint Severity Order
1. Broken wikilinks and missing canonical navigation.
2. Missing/invalid frontmatter on active pages.
3. Source drift or missing raw hashes for important raw files.
4. Orphan pages and pages missing from index/topic map.
5. Long pages over ~200 lines.
6. Tag sprawl and style issues.

### Navigation Scaling
- `index.md` is the global backbone and should stay human-readable.
- When the index exceeds ~200 entries, maintain `_meta/topic-map.md` as a higher-level route map grouped by domain/theme.
- Do not try to list every raw/import/archive file in the main index.
