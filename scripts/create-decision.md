---
name: fullstack-flow/scripts/create-decision
description: "决策日志创建脚本 — 在 decisions/ 目录创建设计决策文件并更新 state.yaml 引用"
version: 2.0.0
tags: [fullstack, script, decision, log]
input:
  - decision_title: 决策标题（用于文件名和文档标题）
  - decision_content: 决策内容（Markdown 正文）
  - phase_id: 触发此决策的 Phase ID（如 spec, plan）
output:
  - decisions/{YYYY-MM-DD}-{decision_title}.md
  - state.yaml decisions[] 更新
---

# 决策日志创建脚本

> 当用户确认设计决策时，创建标准化的决策日志文件并更新 state.yaml 引用。

## 执行流程

### Step 1: 创建决策日志文件

创建文件 `decisions/{YYYY-MM-DD}-{简述}.md`，内容如下：

```markdown
# 设计决策：{decision_title}

**日期**: {YYYY-MM-DD}
**来源**: Phase {phase_id}
**决策者**: 用户确认

## 背景
{说明此决策的背景情况}

## 决策
{决策的具体内容}

## 替代方案
{被拒绝的方案及原因}

## 已排除方案
| 方案 | 排除原因 | 排除者 |
|------|---------|--------|
| {方案 A} | {原因说明} | 用户/CodeBuddy |
| {方案 B} | {原因说明} | 用户/CodeBuddy |

> **已排除方案**记录在此次决策中被明确否决的选项及其原因，避免在后续 Phase 或未来工作中被再次提出。

## 影响范围
{此决策影响的模块/文件}
```

### Step 2: 更新 state.yaml

将决策文件路径追加到 state.yaml 的 `decisions[]` 数组：

```
decisions.append("decisions/{YYYY-MM-DD}-{简述}.md")
```

---

## 使用方式

在 spec.md 或 plan.md 的出口部分替换为：

```markdown
如有设计决策被用户确认，Read `scripts/create-decision.md` 并执行：
- decision_title: "{简要描述}"
- decision_content: "{完整决策正文}"
- phase_id: "spec" / "plan"
```

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 2.0.0 | 2026-06-12 | 决策日志模板新增"已排除方案"章节，记录被明确否决的选项及原因 |
| 1.0.0 | 2026-06-07 | 初始版本：决策日志创建 + state.yaml 引用更新 |
