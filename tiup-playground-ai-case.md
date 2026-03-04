---
title: "AI Agent + TiUP Playground: TiDB Verification Workflow"
category: "Database Testing"
ai_usage:
  - "Code Analysis"
  - "Test Plan Generation"
  - "Automated Validation"
date: "2026-03"
sort: 202603.0
ai_tools:
  - "Codex (GPT-5)"
  - "Claude Code"
description: "Use AI agents to analyze logic in tidb/tikv/PD first, then generate and execute tiup playground test plans to quickly verify DDL, SQL Binding, and regression behaviors."
links: []
thumbnail: ""
---

# AI Agent + TiUP Playground: TiDB Verification Workflow

## Overview

This use case targets TiDB engineering and testing workflows: first use AI agents (such as Codex and Claude Code) to analyze behavior in the `tidb/tikv/PD` repositories, then let the AI generate and execute a `tiup playground` verification plan based on that analysis.

Compared with manually writing test steps from scratch, this workflow is better for cases where understanding code logic first is critical, such as DDL execution paths, SQL Binding effectiveness, and version-specific regressions.

## Applicable Scenarios

- Verify TiDB DDL behaviors (schema change effects, metadata transitions, compatibility boundaries)
- Verify whether SQL Binding is effective as expected
- Reproduce whether a known issue exists on a specific version (`stable` / `nightly`)
- Run fast hypothesis-driven experiments based on code logic

## Simple Workflow

1. Provide [tiup-playground-SKILL.md](./tiup-playground-SKILL.md) to the agent and ask it to set up the environment.
2. Describe what you want to verify, such as a TiDB DDL case or whether a SQL Binding is effective.
3. Ask the agent to analyze code logic in `tidb/tikv/PD` first and define expected behavior and observation points.
4. Ask the agent to generate a `tiup playground` test plan from that analysis (version, topology, SQL steps, assertions).
5. Ask the agent to execute the plan, collect outputs (`tidb_version()`, SQL results, logs), and provide a final conclusion.

## Benefits

- Shorten the cycle from "read code -> design experiment -> conclude"
- Reduce misjudgment caused by missing observation points
- Make verification reusable, traceable, and easy to document

## Created

- March 2026

## AI Tools Used

- Codex (GPT-5)
- Claude Code

## Links
