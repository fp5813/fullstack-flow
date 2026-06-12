---
name: fullstack-flow/references/quick-start
description: "快速入门指南 — L1 入口层文档，新用户/Agent 的首个入口点"
version: 1.0.0
tags: [fullstack, reference, entry, guide]
---

# 快速入门指南（Quick Start）

> 这是 fullstack-flow 的 L1 入口文档。如果你是第一次使用，从这里开始。

## 这是什么

fullstack-flow 是一个**质量门控驱动的规范开发流程**，适用于全栈项目（Vue 3 + Java Spring Boot）。核心流程：

```
澄清 → 探路 → 规格 → 计划 → 代码 → 审核 → 修改记录 → 复盘
```

每个步骤产出明确的制品（文档），通过质量门控确保不遗漏。

## 快速上手

### 1. 加载技能

在 CodeBuddy 中输入 `/fullstack-flow` 加载技能。

### 2. 描述你的需求

Phase 1 会引导你描述任务（新功能、BUG 修复、重构等）。清晰的需求描述有助于后续探路阶段自动选择最佳工作流模式。

### 3. 跟随流程

```
Phase 1 描述澄清 → Gate 0 → Phase 2 代码探路
  → Phase 2.5 质量门控 (Gate 1) → Phase 3 规格澄清
  → Phase 4 实施计划 → Phase 4.5 覆盖验证 (Gate 2)
  → Phase 5 编码 → Phase 5.6 API 文档 → Phase 5.5 审核 (Gate 3)
  → Phase 6 修改记录 (Gate 4) → Phase 6.7 复盘
  → Phase 6.5 规则同步 → Phase 6.6 审计 (Gate 5)
```

每个 Phase 有明确的入口/出口协议和校验清单，按步骤执行即可。

### 4. 需要帮助？

| 场景 | 命令/操作 |
|------|---------|
| 查看当前进度 | 系统自动在 Phase 入口时提示 |
| 切换工作流模式 | `/workflow-pattern` |
| 手动压缩上下文 | `/compact` |
| 压缩知识文件 | `/compress-knowledge`（去掉 memory/instincts 中的填充词，节省 ~46% token） |
| 运行流程审计 | `/fullstack-flow/audit-flow` |
| 查看 Skill 健康度 | `/skill-health-check` |

## 三层加载架构

```
L1 入口层（你在这里）
  ├── SKILL.md 摘要
  ├── scripts/phase-entry-exit.md
  └── references/quick-start.md

L2 当前 Phase 层（按需加载）
  ├── 当前 Phase 文档
  ├── scripts/ 中的对应脚本
  └── 按需 Read 的参考文档

L3 深入层（显式查阅）
  ├── 其他 Phase 文档
  ├── 完整 References
  └── docs/INDEX 系列
```

## 核心文件索引

| 文件 | 用途 | 层级 |
|------|------|:----:|
| `SKILL.md` | 主技能定义（本文档） | L1 |
| `scripts/phase-entry-exit.md` | 入口/出口统一协议 | L1 |
| `scripts/update-index.md` | INDEX.md 更新操作 | L2 |
| `scripts/build-fix.md` | Build-Fix 自动循环 | L2 |
| `scripts/env-check.md` | 环境预检 | L2 |
| `scripts/spawn-probe-agents.md` | 并行 Agent 启动模板 | L2 |
| `scripts/create-decision.md` | 决策日志创建 | L2 |
| `references/communication-rules.md` | Agent 沟通规则（信息密度提升） | L1 |
| `references/skills-navigation-index.md` | 技能导航索引（所有组件的依赖关系） | L1 |
| `references/dynamic-workflows-reference.md` | 动态工作流模式详解 | L3 |
| `references/instinct-reference.md` | Instinct 直觉机制 | L3 |
| `docs/失败案例/` | 失败知识库 | L3 |
| `docs/技能审计/` | 健康度与审计报告 | L3 |

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 1.0.0 | 2026-06-07 | 初始版本：L1 入口指南 |
