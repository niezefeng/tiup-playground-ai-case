---
title: "AI Agent + TiUP Playground：TiDB 问题验证工作流"
category: "数据库测试"
ai_usage:
  - "代码分析"
  - "测试计划生成"
  - "自动化验证"
date: "2026-03"
sort: 202603.0
ai_tools:
  - "Codex (GPT-5)"
  - "Claude Code"
description: "在 tidb/tikv/PD 代码仓库先让 AI Agent 做代码分析，再生成并执行 tiup playground 测试计划，快速验证 DDL、SQL Binding 等问题。"
links: []
thumbnail: ""
---

# AI Agent + TiUP Playground：TiDB 问题验证工作流

## 概要

这个使用 case 聚焦在 TiDB 研发与测试场景：先让 AI Agent（如 Codex、Claude Code）在 `tidb/tikv/PD` 代码仓库中做问题分析，再由 AI 基于分析结论生成 `tiup playground` 验证计划并执行，从而更快完成问题复现和结论确认。

相比直接人工手写测试步骤，这种流程更适合“先理解代码逻辑，再验证行为”的问题，例如 DDL 执行路径、SQL Binding 是否命中、特定版本回归等。

## 适用场景

- 验证 TiDB DDL 行为（如变更生效、元数据状态转换、兼容性边界）
- 验证 SQL Binding 是否按预期生效
- 复现某个 issue 在特定版本（stable/nightly）是否存在
- 基于代码逻辑对猜想做快速实验

## 简单使用流程

1. 把 [tiup-playground-SKILL.md](/Users/anthonynie/AI_project/codex/tiup-playground-SKILL.md) 提供给 Agent，并让它配置到当前环境。
2. 告诉 Agent 你要验证的内容，例如“做一个 TiDB DDL 测试”或“验证某条 SQL Binding 是否生效”。
3. 先在 `tidb/tikv/PD` 仓库里让 Agent 分析代码逻辑，明确预期行为与关键观测点。
4. 让 Agent 基于分析结果生成 `tiup playground` 测试计划（版本、拓扑、SQL 步骤、断言）。
5. 让 Agent 执行计划，收集输出（`tidb_version()`、SQL 结果、日志），并给出最终结论。

## 实际收益

- 缩短“读代码 -> 写实验 -> 得结论”的链路
- 减少遗漏观测点导致的误判
- 让验证过程可复用、可追溯、可沉淀

## 作成日

- 2026年3月

## 利用したAI

- Codex (GPT-5)
- Claude Code

## link
