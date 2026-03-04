# TiUP Playground AI Use Case

这个仓库用于沉淀一个可复用的 AI 使用案例：  
先让 AI Agent（如 Codex、Claude Code）在 `tidb/tikv/PD` 仓库做代码分析，再基于分析结果用 `tiup playground` 生成并执行验证计划，用于快速复现与确认 TiDB 相关问题。

## 文件说明

- [tiup-playground-ai-case.md](./tiup-playground-ai-case.md)  
  AI 使用 case（按模板整理，包含背景、场景、流程和收益）
- [tiup-playground-SKILL.md](./tiup-playground-SKILL.md)  
  给 Agent 使用的 `tiup playground` Skill 文档

## 简单使用流程

1. 将 `tiup-playground-SKILL.md` 提供给 Agent，并让它配置到当前环境。  
2. 告诉 Agent 你要验证的目标（例如 TiDB DDL 行为、SQL Binding 是否生效）。  
3. 先在代码仓库中让 Agent 分析逻辑与预期，再让它生成 `tiup playground` 测试计划。  
4. 让 Agent 执行测试并输出结论与证据（版本信息、SQL 结果、日志）。  

## 适用示例

- DDL 行为验证（状态转换、兼容性边界）
- SQL Binding 命中验证
- 特定版本（stable/nightly）回归复现

