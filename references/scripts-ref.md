---
name: fullstack-flow/references/scripts-ref
description: "脚本库总览 — 列出所有可引用的 scripts/ 脚本及其用途、参数和引用阶段"
tags: [fullstack, reference, scripts, automation]
version: 2.0.0
---

# 脚本库总览（scripts/）

> 所有固定的、重复的步骤集中存放在 `scripts/` 目录下，Phase 文档通过 `Read` 加载对应脚本并传参执行。
> 脚本化的目的：让 AI 精力集中在逻辑判断与组合上，而不是重建轮子。

## 脚本清单

| 脚本 | 版本 | 用途 | 输入参数 | 引用 Phase |
|------|:----:|------|---------|-----------|
| [phase-entry-exit.md](../scripts/phase-entry-exit.md) | 1.0.0 | Phase 入口/出口统一协议 | current_phase, next_phase, artifacts_key, prereq_artifacts, is_gate_phase | **所有 13 个 Phase** |
| [update-index.md](../scripts/update-index.md) | 1.0.0 | INDEX.md 统一更新（prepend/append） | index_path, row_content, mode | probe, spec, plan, api-doc, record, retrospect 等 |
| [build-fix.md](../scripts/build-fix.md) | 1.0.0 | Build-Fix 自动循环（6 种错误分类 + 3 轮修复） | project_type, max_rounds | code |
| [env-check.md](../scripts/env-check.md) | 2.0.0 | 环境预检（工具链 + MCP 就绪 + CI 预检） | project_type, mcp_mysql_name, mcp_codegraph_name | clarify |
| [spawn-probe-agents.md](../scripts/spawn-probe-agents.md) | 1.0.0 | 并行探路子 Agent 标准化启动模板 | project_root, mcp_mysql_name, mcp_codegraph_name, change_summary | probe |
| [create-decision.md](../scripts/create-decision.md) | 2.0.0 | 决策日志创建 + state.yaml 引用更新（含已排除方案） | decision_title, decision_content, phase_id | spec, plan |
| [invariant-gates.md](../scripts/invariant-gates.md) | 1.1.0 | 不变性门验证（10 条核心不变性 + 自动化 grep/bash 验证） | phase, probe_report_path, spec_path, plan_path, api_doc_dir, controller_dir, frontend_api_dir | quality-gate, coverage-check, code, review |

## 脚本化原则

1. **引用优先于复制**：所有重复步骤优先引用 `scripts/` 下的脚本，而非在每个 Phase 文档中重新编写
2. **参数化输入输出**：每个脚本定义明确的 `input` 参数和 `output` 制品，引用时按参数替换占位符
3. **版本可追溯**：每个脚本有独立的 version 和变更记录，修改后更新 CHANGELOG 和所有引用 Phase
4. **可组合性**：脚本可被多个 Phase 引用（如 phase-entry-exit 被所有 Phase 引用），也可被其他脚本嵌套调用

## 维护指南

- 新增脚本：在 `scripts/` 下创建 `.md` 文件，包含 frontmatter 的 input/output 定义，并更新此总览
- 修改脚本：更新脚本自身版本号，检查所有引用该脚本的 Phase 文档是否需要同步调整
- 废弃脚本：在脚本 frontmatter 添加 `deprecated: true` 字段，并标记替代脚本路径

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 2.0.0 | 2026-06-12 | 新增 invariant-gates 脚本；更新 env-check.md v2.0、create-decision.md v2.0 版本号 |
| 1.0.0 | 2026-06-07 | 初始版本：列出 6 个脚本 |
