---
name: fullstack-flow/phases/audit
description: "业务规则审计：6 并行 Agent 扫描全项目 6 类业务规则模式（状态判断/枚举/权限/数据过滤/前端条件/@Dict），全覆盖识别未归档规则并输出审计报告。"
version: 2.0.0
tags: [fullstack, codebuddy-only, audit, business-rule, read-only, parallel-agents]
role: codebuddy-auditor
model: deepseek-v4-flash
tools: [Read, Grep, Agent]
references:
  - ../references/codegraph-reference.md
---

# Phase 6.6: 业务规则审计

> Phase 6.5 只记录被修改触及的规则。稳定代码中的业务规则无人发现。本阶段主动扫描补齐缺口。

## 快速概览

```
Step 0 读取状态 → Step 1 确定审计范围 → Step 2 并行 Agent 扫描（6 类模式）
  → Step 2a 合并去重 → Step 3 比对已归档规则
  → Step 4 输出审计报告 → Gate 5 → Phase 出口 → 结束
```

**核心**: 6 并行 Agent 全覆盖扫描（状态/枚举/权限/数据过滤/前端条件/@Dict） + 自动合并去重

## 职责

**输入**：Phase 6.5 输出的业务规则状态 + Phase 5 代码变更范围（不依赖 BUG/需求）  
**输出**：`docs/业务规则/审计/YYYY-MM-DD-审计报告.md`  
**模式**：只读（不修改源代码）

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="audit", next_phase=null（工作流结束）

### Step 1: 确定审计范围

- 全量审计（初次推荐）→ 扫描全部模块
- 模块审计 → 指定模块
- 差异审计 → 扫描已覆盖模块之外的缺口

### Step 2: 并行 Agent 扫描 6 类业务规则模式

一次性发送 6 个 Agent 调用（`run_in_background: true`），每类模式一个专用 Agent，无依赖关系：

| Agent | 扫描模式 | 模型 | 查询方式 |
|-------|---------|:----:|---------|
| Agent-1 | 状态/字典值判断 | lite | `codegraph_context(task="找出 equals('archiveStatus') 或字典值比较的 if/switch")` |
| Agent-2 | 枚举类 | lite | `codegraph_search(query="Enum", kind="class")` + `codegraph_node` |
| Agent-3 | 权限注解 | lite | `codegraph_context(task="找出 @RequiresPermissions")` |
| Agent-4 | 数据权限过滤 | lite | `codegraph_context(task="找出 inSql/joinSql/archive_data_permission")` |
| Agent-5 | 前端条件渲染 | lite | `codegraph_context(task="找出 v-if archiveStatus")` |
| Agent-6 | @Dict 注解 | lite | `codegraph_context(task="@Dict 在实体类中的使用")` |

**Agent 约束**：
- 每个 Agent 只负责自己的一种模式，不交叉扫描
- 返回 ≤300 tokens 的摘要（只输出位置 file:line 和简要描述）
- 不修改任何文件

### Step 2a: 主流程等待并合并结果

等待 6 个 Agent 全部完成后，执行合并：

1. **按文件:行号排序** — 统一排序便于对比
2. **去重** — 同一位置被多个 Agent 发现时只保留一条，标注所有匹配的模式类型
3. **标记** — 每个发现标注模式类型（状态/枚举/权限/数据过滤/前端条件/@Dict）

### Step 3: 对比已有规则

与 `docs/业务规则/` 对比：状态值是否已覆盖、权限标识是否已记录、枚举类是否已归档、字典编码是否已说明。

### Step 4: 输出审计报告

```markdown
# 业务规则审计报告
## 基本信息（日期/范围/已归档 N 条/未归档 N 条）
## 未归档规则清单
| # | 位置 | 模式 | 描述 | 操作建议 |
|---|------|------|------|---------|
| 1 | `file:line` | 状态判断 | `if("X".equals(status))` | 新增规则文件 |
## 按模块汇总
| 模块 | 已有规则 | 未归档数 | 优先级 |
```
## 自检

- [ ] 已确定审计范围
- [ ] 6 个并行 Agent 已全部返回结果
- [ ] 已完成 6 类业务模式全覆盖扫描（强制）
- [ ] 合并去重已完成
- [ ] 已与已有规则对比
- [ ] 报告已输出

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 `artifacts.audit_report.path`, `artifacts.gate_5.status`, `artifacts.gate_5.failed_items`
2. 参数: current_phase="audit", next_phase=null（工作流结束）
3. 额外: `phase.current = null`, `phase.status = "completed"`（工作流结束）
4. `gates_summary` 更新计数

## 复盘衔接

审计报告作为 Phase 6.7 复盘回顾的输入素材之一：

| 条件 | 动作 |
|------|------|
| 未归档规则 ≥5 条 | 强制触发**阶段复盘**（Phase 6.7 入口判定为"是"） |
| 未归档规则 <5 条 | 作为复盘可选素材，供五维分析的"技能可用性"维度使用 |

## 约束

- 每个 Agent 只负责一类模式，不交叉扫描
- Agent 返回 ≤300 tokens 摘要（仅输出 file:line 位置和简要描述）
- 绝不修改源代码
