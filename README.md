# TiUP Playground AI Use Case

This repository documents a reusable AI-assisted workflow:
use AI agents (such as Codex and Claude Code) to analyze `tidb/tikv/PD` code first, then generate and execute a `tiup playground` verification plan to quickly reproduce and validate TiDB behaviors.

## Files

- [tiup-playground-ai-case.md](./tiup-playground-ai-case.md)  
  AI usage case based on the provided template (background, scenarios, process, and benefits)
- [tiup-playground-SKILL.md](./tiup-playground-SKILL.md)  
  `tiup playground` skill document for AI agents
- [ddl_reorg_test_prompt_EN.md](./ddl_reorg_test_prompt_EN.md)  
  AI-generated executable test plan for TiDB DDL reorg phase validation

## Quick Workflow

1. Give `tiup-playground-SKILL.md` to your agent and ask it to set up the environment.
2. Describe what you want to verify (for example, TiDB DDL behavior or SQL Binding effectiveness).
3. Ask the agent to analyze the logic in the code repo first, then generate a `tiup playground` test plan.
4. Ask the agent to execute the plan and provide conclusions with evidence (version info, SQL output, and logs).

## Typical Use Cases

- DDL behavior verification (state transitions, compatibility boundaries)
- SQL Binding hit/miss verification
- Regression reproduction on specific versions (`stable` / `nightly`)
