---
name: fullstack-flow/references/communication-rules
description: "Agent 沟通规则 — 最高效的沟通不是把话说得更漂亮，而是把信息密度提得更高。基于 caveman 项目的核心理念。"
version: 1.0.0
tags: [fullstack, reference, communication, efficiency]
---

# Agent 沟通规则（Communication Rules）

> **最高效的沟通不是把话说得更漂亮，而是把信息密度提得更高。**
> 参考自 [caveman](https://github.com/JuliusBrussee/caveman) — *"why use many token when few do trick"*

## 基本原则

| 规则 | 说明 | ❌ 差 | ✅ 好 |
|------|------|-------|-------|
| 去填充词 | 去掉 just/really/basically/actually/simply | "The issue is actually caused by..." | "Bug in auth middleware." |
| 去客套话 | 去掉 sure/certainly/happy to/of course | "Sure! I'd be happy to help." | "Fix: ..." |
| 用片段 | fragments OK，上下文自明 | "The reason your component is re-rendering is likely because..." | "New object ref each render. Wrap in `useMemo`." |
| 短同义词 | 用短词替换长词 | "implement a solution for" / "extensive" | "fix" / "big" |
| 技术术语精确 | 代码/路径/方法名/错误信息不缩写 | "the API endpoint that handles..." | "`/api/user/list`" |
| 因果显式 | 用 `→` 表因果关系 | "this leads to that which causes..." | "Inline obj prop → new ref → re-render." |
| 模式化表达 | `[问题] [根因] [方案]。[下一步]。` | 一段式描述混杂所有信息 | "Bug in auth middleware. Token expiry check use `<` not `<=`. Fix: change to `<=`." |

## 强度分级

| 级别 | 适用 Phase | 说明 | 示例 |
|------|-----------|------|------|
| **normal** | 探路报告、规格文档、修改记录 | 完整句，保留必要连接词，文档级可读性 | "The issue is caused by incorrect token expiry comparison in the auth middleware." |
| **concise** | 交互式 Q&A、审核意见、复盘分析 | 去填充词，短句，fragments OK，`→` 表因果 | "Bug in auth middleware. Token expiry use `<` not `<=`." |
| **ultra** | 内部状态记录、自检备注 | 极致压缩，缩写常见词（req/res/fn/impl），`→` 表因果，一个词够用不用两个 | "Auth middleware: token expiry `<` → need `<=`." |

### 各级别对比示例

同一问题"为什么 React 组件重复渲染？"的三种回答：

- **normal**: "Your component re-renders because a new object reference is created on each render cycle. Wrap the inline object prop in `useMemo` to stabilize the reference."
- **concise**: "New object ref each render. Inline object prop = new ref = re-render. Wrap in `useMemo`."
- **ultra**: "Inline obj prop → new ref → re-render. `useMemo`."

## 自动清晰化触发条件

以下场景**退出 concise/ultra 模式**，使用 **normal 模式**的完整清晰表达：

| 触发条件 | 说明 | 示例场景 |
|---------|------|---------|
| **安全警告** | 涉及安全漏洞、权限绕过 | SQL 注入风险、未授权访问 |
| **不可逆操作确认** | 删除数据、清空表、覆盖文件 | `DROP TABLE`、`rm -rf`、批量删除 |
| **多步骤歧义** | 步骤间依赖不清晰时，fragments 可能被误解 | "migrate table drop column backup first" — 顺序不清 |
| **用户困惑** | 用户要求重复或表示不理解 | 用户说"再说一遍"或"什么意思" |
| **技术术语歧义** | 同一缩写在不同上下文有不同含义 | "DB" 在不同场景可能指 database 或 decibel |

### 切换机制

```
正常对话 → concise/ultra 模式
  ↓ (触发自动清晰化条件)
切换到 normal 模式，完整清晰表达
  ↓ (清晰部分结束)
自动恢复 concise/ultra 模式
```

## 输出边界

不同输出类型使用不同沟通级别：

| 输出类型 | 推荐级别 | 说明 |
|---------|:-------:|------|
| 探路报告（L0-L5） | normal | 文档需要完整可读 |
| 规格文档 | normal | 需要精确无歧义 |
| 实施计划 | normal | 任务描述需要精确 |
| 修改记录 | normal | 需要完整记录 |
| 交互式 Q&A（Phase 1/3） | **concise** | 快速聚焦问题本质 |
| 审核意见（Phase 5.5） | **concise** | 一行一条，直奔主题 |
| 复盘分析（Phase 6.7） | **concise** | 五维分析每条简洁 |
| 不变性门验证结果 | **concise** | 表格化输出 |
| 自检备注 | **concise** | ✅/❌ 标记 + 简短说明 |
| 代码/配置内容 | 原样保留 | 不压缩，原样输出 |
| Commit 信息 | **concise** | ≤50 字符主题，说"为什么"而非"做了什么" |

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 1.0.0 | 2026-06-12 | 初始版本：基于 caveman 核心理念，定义沟通原则/强度分级/自动清晰化/输出边界 |
