---
title: Hermes Gateway
created: 2026-04-23
updated: 2026-04-23
type: entity
domain: hermes-ops
tags:
- entity
- hermes
- runtime
- configuration
sources:
- SCHEMA.md
confidence: medium
status: active
systems:
- hermes
---
# Hermes Gateway

## 是什么
Hermes Gateway 是本机 Hermes 体系的运行入口与服务承接点。

## 在本系统中的角色
- 承接 Hermes CLI / agent 运行面的服务角色
- 与 memory provider、skills、wiki 工作流一起构成 Hermes 的执行环境
- 在本机架构中与 [[entities/hermes-ops/Hindsight]]、[[entities/hermes-ops/llm-wiki]]、[[entities/hermes-ops/Roundtable Runtime]] 形成协作关系

## 当前关联页面
- [[entities/hermes-ops/Hindsight]]
- [[entities/hermes-ops/llm-wiki]]
- [[entities/hermes-ops/Roundtable Runtime]]

## 备注
更细的 live 运行细节可后续继续沉淀到 query / diagnosis 页。
