---
name: fullstack-flow/phases/coverage-check
description: "实施计划覆盖验证：检查规格与计划的 AC 覆盖率、文件覆盖率、术语一致性，确保零遗漏后进入 Phase 5。"
version: 2.0.0
tags: [fullstack, codebuddy-only, coverage-check, read-only]
role: codebuddy-analyzer
model: deepseek-v4-flash
tools: [Read, Grep, Agent]
references:
  - ../references/spec-driven-development.md
---

# Phase 4.5: 覆盖验证

> 跨制品一致性分析 — 在写代码前确保规格、计划、任务三者一致。

## 快速概览

```
Step 0 读取状态 → Step 0.5 不变性门自动验证（I3/I4）
  → 验证 AC 覆盖率 → 验证文件覆盖率 → 验证术语一致性
  → 验证数据追溯 → Gate 2 判定 → Phase 出口 → code
```

**核心**: 不变性门自动验证 + 5 条件 AND 门控（100% 通过方可进入 Phase 5）

## 职责

**输入**：规格文档（`docs/规格文档/`）+ 实施计划（`docs/实施计划/`）  
**输出**：覆盖验证报告（追加到实施计划末尾）  
**模式**：只读（不修改代码）

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="coverage-check", next_phase="code"（通过）/ "plan"（不通过）, is_gate_phase=true
- 额外：验证前置制品 `artifacts.plan.path` 存在

### Step 0.5: 不变性门自动验证

Read `scripts/invariant-gates.md` 并执行以下不变性门的 grep 验证：

| 不变性 | 验证内容 | 命令 | 阈值 |
|--------|---------|------|:----:|
| **I3** | AC→探路报告可追踪 | `for ac in $(grep -oP 'AC\d+' "{spec_path}"); do grep -c "$ac" "{probe_report_path}"; done` | 每个 AC ≥1 次引用 |
| **I4** | T###→AC 映射完整 | `grep -oP 'T\d+' "{plan_path}" 对比 AC 引用` | 无孤儿任务（允许标注原因） |

执行 grep 命令验证，输出结果表格追加到实施计划末尾的覆盖验证报告之前。

I3 为 ⚠️ 半自动（辅助检查，最终人工确认映射关系合理性）。
I4 为 ✅ 可自动验证（孤儿任务必须标注原因）。

## 检查维度（标注不变性引用）

### 1. AC 覆盖率（→ I3, I4） | 核心：每个 AC 至少被一个任务覆盖

| AC | 验收标准 | 覆盖任务 | 状态 |
|----|----------|----------|------|
| AC01 | {描述} | T001, T003 | ✅ |

缺少覆盖 → ❌ 回 Phase 4 补充任务。

### 2. 文件覆盖率（→ I4） | 核心：规格中"直接修改文件"在任务清单中有对应

### 3. 孤儿任务检测（→ I4） | 核心：无 AC/文件映射的任务需说明原因

优化类任务可保留但标注原因，无理由孤儿 ❌ 回 Phase 4 删除或补充 AC。

### 4. 术语一致性 | 核心：规格与计划的术语无漂移

### 5. 不在范围检查 | 核心：计划任务未侵入规格声明"不在范围内"的内容

### 6. 依赖关系验证 | 核心：无循环依赖，后端先行，并行任务不操作同一文件

### 7. 数据来源追溯 | 核心：规格文档中的数据来源与探路报告一致（涉及 DB 时）

| 追溯项 | 探路报告 | 规格文档 | 状态 |
|--------|---------|---------|------|
| VO/DTO 类名和字段结构 | L4.5 章节 | API 接口数据章节 | ✅/❌ |
| 表名和字段 | L4 章节 | 表结构章节 | ✅/❌ |
| 表↔VO 字段映射 | 测试数据样例章节 | 表↔VO 字段映射章节 | ✅/❌ |
| 测试数据 SQL | 测试数据样例章节 | 测试数据章节 | ✅/❌ |

缺少追溯 → ❌ 回 Phase 4 修正规格文档。

## 门控判定

| 条件 | 状态 |
|------|------|
| AC 覆盖率 = 100% AND 文件覆盖率 = 100% AND 无不在范围违规 AND 术语一致 AND 数据追溯完整 | ✅ 进入 Phase 5 |
| 以上任一项不满足 | ❌ 回 Phase 4 |

> 孤儿任务有合理理由（prefactor 等技术任务）→ ✅ 标注原因后通过，不视为失败项。

## 自检

- [ ] I3 不变性门（AC→探路报告可追踪）已执行并记录
- [ ] I4 不变性门（T###→AC 映射完整）已执行并记录
- [ ] 不变性门验证结果已追加到实施计划末尾
- [ ] AC 覆盖率 100%
- [ ] 无不在范围违规
- [ ] 术语一致
- [ ] 数据追溯完整（涉及 DB 时）

## 输出

不变性门验证结果追加在实施计划末尾、覆盖验证报告之前：

```markdown
## 不变性门验证结果
| ID | 不变性 | 状态 | 证据 |
|----|--------|:----:|------|
| I3 | AC→探路报告可追踪 | ✅ | AC01:3次 AC02:1次 |
| I4 | T###→AC 映射完整 | ✅ | 0 个孤儿任务 |
```

覆盖验证报告：

```markdown
## 覆盖验证（Phase 4.5 Gate）
**不变性门**: I3 ✅ / I4 ✅
**AC 覆盖率**: {covered}/{total} = {percent}%
**文件覆盖率**: {covered}/{total} = {percent}%
**术语一致性**: {consistent}/{total} = {percent}%
**数据来源追溯**: {consistent}/{total} = {percent}%（涉及 DB 时）
**不在范围违规**: {count} 项
**门控状态**: {✅ 通过 / ❌ 不通过}
**处理建议**: {根据门控状态的下一步}
```

## 约束

- 绝不写代码。发现不一致只标注不自动修改。用具体指标说话，不过度分析（最多 20 条发现）。
- **不变性门不可跳过**：I3/I4 任一不通过 → 回退 Phase 4。

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 `artifacts.gate_4_5.status`, `artifacts.gate_4_5.score`, `artifacts.gate_4_5.failed_items`
2. 参数: current_phase="coverage-check", next_phase="code"（通过）/ "plan"（不通过）, is_gate_phase=true
3. 额外：不通过时 `metrics_snapshot.gate_retries += 1`
4. `gates_summary` 更新计数
