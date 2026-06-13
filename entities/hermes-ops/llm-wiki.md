---
title: llm-wiki
created: 2026-04-23
updated: 2026-04-23
type: entity
domain: hermes-ops
tags:
- entity
- wiki
- hermes
- governance
sources:
- SCHEMA.md
confidence: medium
status: active
systems:
- hermes
- llm-wiki
---
# llm-wiki

## 是什么
`llm-wiki` 是 Hermes 当前采用的持久化 markdown wiki 工作模式：以 `SCHEMA.md + index.md + log.md + interlinked pages` 为骨架，承接持续增量维护，而不是每次重新做一次性检索。

## 在本系统中的角色
- 作为 Hermes / Agent Ops Wiki 与 LLM Wiki（本地文件夹：LLM Wiki） 的共同底座模式
- 负责把运行知识、研究知识分别沉淀到独立 wiki
- 通过 `index.md` 与 `log.md` 维持导航与演化记录
- 在运行层与 [[entities/hermes-ops/Hermes Gateway]] 协同，由 [[entities/hermes-ops/Hindsight]] 承接非 wiki 型长期记忆

## 当前关联页面
- [[concepts/hermes-ops/外置记忆]]
- [[concepts/hermes-ops/llm-wiki-canonical-maintenance]]
- [[concepts/hermes-ops/Stock Roundtable Runtime Architecture]]
- [[entities/hermes-ops/Hindsight]]

## 关键边界
- `llm-wiki` 不是 memory provider
- `llm-wiki` 不替代 [[entities/hermes-ops/Hindsight]]
- `llm-wiki` 承接文档化知识，memory 承接稳定偏好与事实
