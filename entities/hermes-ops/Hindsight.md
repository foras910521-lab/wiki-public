---
title: Hindsight
created: 2026-04-23
updated: 2026-04-23
type: entity
domain: hermes-ops
tags:
- entity
- hindsight
- memory
- provider
sources:
- SCHEMA.md
confidence: medium
status: active
systems:
- hermes
- hindsight
- memory
---
# Hindsight

## 是什么
Hindsight 是当前 Hermes 使用的 memory provider，负责承接稳定记忆、语义召回与反思型汇总。

## 在本系统中的角色
- 作为 Hermes 文本记忆的外部 provider
- 承接从 `MEMORY.md` / `USER.md` 下游的长期信息
- 与 [[entities/hermes-ops/llm-wiki]] 并行存在：一个管 memory，一个管 wiki
- 在本机架构中通过 [[entities/hermes-ops/Hermes Gateway]] 被调用，并与 [[concepts/hermes-ops/llm-wiki-canonical-maintenance]] / [[concepts/hermes-ops/外置记忆]] 形成边界分工

## 当前关联页面
- [[entities/hermes-ops/llm-wiki]]
- [[concepts/hermes-ops/外置记忆]]
- [[concepts/hermes-ops/llm-wiki-canonical-maintenance]]

## 关键边界
- Hindsight 不是股票研究 wiki
- Hindsight 不承接大段 source page / case page 正文
- Hindsight 主要承接稳定事实、偏好、治理结论
