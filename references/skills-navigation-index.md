---
name: fullstack-flow/references/skills-navigation-index
description: "技能导航索引 — 定义所有技能组件的前置依赖、适用场景和加载层级，支持动态匹配"
version: 2.0.0
tags: [fullstack, reference, navigation, index]
---

# 技能导航索引（Skills Navigation Index）

> 所有技能组件的依赖关系、适用场景和加载层级定义。
> 用于"索引建立→意图匹配→动态补充→重新匹配"四阶段动态加载循环。

---

## 一、组件依赖图谱

### Phase 组件

| ID | 名称 | 前置依赖 | 适用场景 | 层级 | 关键制品 |
|----|------|---------|---------|:----:|---------|
| clarify | 描述澄清 | —（入口 Phase） | 所有任务 | L1 | 澄清摘要 + keywords |
| probe | 代码探路 | clarify | 所有非 trivial 任务 | L2 | 探路报告 |
| quality-gate | 质量门控 | probe | 探路后必检 | L2 | 门控报告 |
| spec | 规格澄清 | quality-gate passed | 所有任务 | L2 | 规格文档 |
| plan | 实施计划 | spec | 所有任务 | L2 | 实施计划 |
| coverage-check | 覆盖验证 | plan | 计划后必检 | L2 | 覆盖报告 |
| code | 编码实现 | coverage-check passed | 所有任务 | L2 | 代码修改 |
| api-doc | API 文档 | code | 涉及 API 修改 | L2 | 接口文档 |
| review | 代码审核 | api-doc | 所有任务 | L2 | 审核报告 |
| record | 修改记录 | review passed | 所有任务 | L2 | 修改记录 |
| retrospect | 复盘回顾 | record | BUG/重复问题 | L2 | 复盘记录 + 失败案例 |
| rule-sync | 规则同步 | retrospect 或 record | 涉及业务逻辑 | L2 | 规则文件 |
| audit | 流程审计 | rule-sync | 流程结束 | L2 | 审计报告 |

### 脚本组件

| ID | 名称 | 引用 Phase | 适用场景 | 层级 |
|----|------|-----------|---------|:----:|
| phase-entry-exit | 入口/出口协议 | 所有 13 Phase | 必须 | L1 |
| update-index | INDEX 更新 | api-doc, plan, probe, record, retrospect, rule-sync, spec | 需要更新 INDEX | L2 |
| build-fix | Build-Fix 循环 | code | 编译/测试失败 | L2 |
| env-check | 环境预检 | clarify | 首次运行 | L2 |
| spawn-probe-agents | 并行探路 Agent | probe | 探路阶段 | L2 |
| create-decision | 决策日志 | spec, plan | 有设计决策 | L2 |
| **invariant-gates** | **不变性门验证** | **quality-gate, coverage-check, review** | **门控阶段自动验证** | **L2** |

### 子技能组件

| ID | 名称 | 触发场景 | 层级 |
|----|------|---------|:----:|
| audit-flow | 流程审计 | 显式命令 `/audit-flow` 或 Phase 6.6 | L3 |
| onboard-project | 项目接入 | 新子项目 | L3 |
| change-record | 修改记录生成 | Phase 6 | L3 |
| code-explore | 代码探路 | Phase 2 | L3 |
| vue-standards | Vue 规范 | Phase 5 前端 | L3 |
| java-dev-standards | Java 开发规范 | Phase 5 后端 | L3 |
| java-review-standards | Java 审查清单 | Phase 5.5 后端审核 | L3 |
| java-test-standards | Java 测试规范 | Phase 5 TDD 流程 | L3 |

### 参考文档

| ID | 名称 | 关联场景 | 层级 |
|----|------|---------|:----:|
| quick-start | 快速入门 | 新用户首次使用 | L1 |
| skills-navigation-index | 导航索引 | 动态匹配 | L1 |
| spec-driven-development | 规格驱动开发 | Phase 3/4 | L2 |
| codegraph-reference | Codegraph 使用 | Phase 2/审计 | L2 |
| mcp-tools-summary | MCP 工具清单 | 所有 Phase | L2 |
| api-fetcher-reference | API Fetcher | Phase 2 探路 | L2 |
| change-record-detailed | 修改记录详解 | Phase 6 | L2 |
| instinct-reference | Instinct 机制 | Phase 6.7 | L3 |
| e2e-test-rules | E2E 测试规则 | Phase 5.5 后端 API 修改验证 | L3 |
| dynamic-workflows-reference | 动态工作流 | Phase 1 模式选择 | L3 |
| vue-* (4 文件) | Vue 技术参考 | Phase 5 前端 | L3 |
| scripts-ref | 脚本库总览 | 维护者查阅 | L3 |

---

## 二、场景标签矩阵

每个组件标注适用场景标签：

| 场景标签 | 说明 | 包含组件 |
|---------|------|---------|
| `entry` | 入口/初始化 | clarify, phase-entry-exit, env-check, quick-start |
| `explore` | 探路/调查 | probe, spawn-probe-agents, code-explore, codegraph-reference, mcp-tools-summary |
| `spec` | 规格/计划 | spec, plan, spec-driven-development |
| `gate` | 门控验证 | quality-gate, coverage-check, review, java-review-standards |
| `code` | 编码实现 | code, build-fix, vue-standards, java-dev-standards, java-test-standards, vue-* references |
| `document` | 文档生成 | api-doc, record, change-record, change-record-detailed |
| `retrospect` | 复盘改进 | retrospect, rule-sync, audit, instinct-reference, create-decision |
| `maintenance` | 技能维护 | audit-flow, skill-health-check, scripts-ref, update-index |

---

## 三、动态匹配规则

### 匹配优先级

```
phase.current（当前阶段） > task.type（任务类型） > 问题类型（具体情境）
```

### Phase 转换时的预加载清单

| 当前 Phase | 预加载内容 |
|-----------|-----------|
| clarify → probe | scripts/spawn-probe-agents.md, references/codegraph-reference.md |
| probe → quality-gate | scripts/invariant-gates.md |
| quality-gate → spec | references/spec-driven-development.md |
| spec → plan | 同上 |
| plan → coverage-check | scripts/invariant-gates.md |
| coverage-check → code | scripts/build-fix.md |
| code → api-doc | （无） |
| api-doc → review | scripts/invariant-gates.md |
| review → record | scripts/update-index.md |
| record → retrospect | scripts/update-index.md |
| retrospect → rule-sync | （无） |
| rule-sync → audit | （无） |

---

## 四、维护指南

- 新增 Phase：在此索引中新增一行，标注前置依赖和场景标签
- 新增脚本：在脚本组新增行，标注引用 Phase
- 变更层级：L1 → L2 或 L2 → L3 需要同步更新 `SKILL.md` 的渐进式披露章节和 `references/quick-start.md` 的核心文件索引

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 2.0.0 | 2026-06-12 | 新增 invariant-gates 脚本组件；quality-gate/coverage-check/review 预加载清单追加 invariant-gates |
| 1.0.0 | 2026-06-07 | 初始版本：完整组件依赖图谱 + 场景标签矩阵 + 动态匹配规则 |
