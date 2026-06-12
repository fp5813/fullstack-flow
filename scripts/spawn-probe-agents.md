---
name: fullstack-flow/scripts/spawn-probe-agents
description: "并行探路子 Agent 启动模板 — 自动检测 codegraph 服务并分发 Agent A（按子项目并行），Agent B/C/D 保持不变"
version: 2.0.0
tags: [fullstack, script, probe, agents, parallel]
input:
  - project_root: 项目根目录
  - mcp_mysql_name: mysql MCP 服务名
  - mcp_codegraph_name: codegraph MCP 服务名（默认）
  - mcp_codegraph_services: 检测到的子项目 codegraph 服务列表（逗号分隔，可选）
  - change_summary: 本次变更的简要描述
output:
  - N 个 Agent 启动指令块（Agent A 按子项目数分发）
---

# 并行探路子 Agent 启动模板

> 定义并行子 Agent 的标准化启动指令。自动检测 codegraph 子项目服务并分发 Agent A。

## 启动方式

在 probe.md Step 2 中，根据 `mcp_codegraph_services` 检测结果决定 Agent A 的分发策略，然后并行启动所有 Agent。

## 策略选择

### 模式 A：多子项目分发（检测到 2+ 个 codegraph-{子项目名} 服务时）

为每个子项目启动一个独立的代码结构探路 Agent A-N：

```
Agent A-1: 子项目1 代码结构探路 → 使用 codegraph-子项目1
Agent A-2: 子项目2 代码结构探路 → 使用 codegraph-子项目2
...
Agent B: DB 表结构 + API 采样 + VO 视图（不变）
Agent C: 影响范围 + 业务规则（使用全项目 codegraph）
Agent D: 文档查阅（不变）
```

### Agent A-{N}: {子项目名} 代码结构探路

**model**: `reasoning`, **max_turns**: 20

**Prompt**:
```
你是一个代码结构探路 Agent。任务是分析 {change_summary} 在 {子项目名} 模块中的代码调用链。

项目根目录：{project_root}
Codegraph MCP 服务：mcp__codegraph-{子项目名}

执行以下步骤：
1. 使用 `mcp__codegraph-{子项目名}__codegraph_context` 获取 {子项目名} 模块中与变更相关的代码上下文
2. 使用 `mcp__codegraph-{子项目名}__codegraph_node` 定位关键类/方法定义
3. 使用 `mcp__codegraph-{子项目名}__codegraph_trace` 追踪调用链（Controller → Service → DAO）

输出格式：每个点带 file:line 的调用链 Markdown 报告，标注 {子项目名} 模块。
```

> 子项目数超过 3 个时，仅对影响最大的前 3 个子项目分发 Agent A，其余通过全项目 codegraph 覆盖。

### 模式 B：单子项目或无子项目服务（回退）

当检测到 0 或 1 个 codegraph-{子项目名} 服务时，使用标准 4 Agent 流程。

### Agent A: 代码结构探路（全项目）

**model**: `reasoning`, **max_turns**: 20

**Prompt**:
```
你是一个代码结构探路 Agent。任务是分析 {change_summary} 相关的代码调用链。

项目根目录：{project_root}
Codegraph MCP 服务：mcp__{mcp_codegraph_name}

执行以下步骤：
1. 使用 `mcp__{mcp_codegraph_name}__codegraph_context` 获取变更相关模块的代码上下文
2. 使用 `mcp__{mcp_codegraph_name}__codegraph_node` 定位关键类/方法定义
3. 使用 `mcp__{mcp_codegraph_name}__codegraph_trace` 追踪调用链（Controller → Service → DAO）

输出格式：每个点带 file:line 的调用链 Markdown 报告。
```

---

### Agent B: DB 表结构 + API 采样 + VO 视图

**model**: `reasoning`, **max_turns**: 25

**Prompt**:
```
你是一个 DB+API+VO 探路 Agent。任务是分析 {change_summary} 相关的数据层和 API 层。

项目根目录：{project_root}
MySQL MCP 服务：mcp__{mcp_mysql_name}

执行以下步骤：

B1: 数据库表结构（3 步）
1. use `mcp__{mcp_mysql_name}__execute_query` 查询相关表结构
2. 记录表名、字段名、类型、注释、索引
3. 标注哪些字段可能受本次变更影响

B2: API 接口采样（3 步）
1. 定位涉及 Controller 的 Feign 调用或 API 请求
2. 记录接口路径、请求方法、请求/响应参数
3. 标注哪些接口需要更新文档

B3: 数据流转（3 步）
1. 追踪 Controller → Service → DAO 的完整 VO/DTO 调用链
2. 记录 VO/DTO 字段定义
3. 标注字段变更影响范围

输出格式：按 B1/B2/B3 分类的详细 Markdown 报告，每条带 file:line。
```

### Agent C: 影响范围 + 业务规则

**model**: `reasoning`, **max_turns**: 20

**Prompt**:
```
你是一个影响范围探路 Agent。任务是评估 {change_summary} 对整个系统的影响范围。

项目根目录：{project_root}
Codegraph MCP 服务：mcp__{mcp_codegraph_name}

执行以下步骤：
1. 使用 `mcp__{mcp_codegraph_name}__codegraph_impact` 评估变更影响范围
2. 使用 `mcp__{mcp_codegraph_name}__codegraph_explore` 探索相关模块的依赖关系
3. 使用 `mcp__{mcp_codegraph_name}__codegraph_context` 获取受影响模块的完整上下文

重点关注：状态机变更、权限注解变更、数据过滤变更、前端条件渲染变更。
```

### Agent D: 文档查阅

**model**: `lite`, **max_turns**: 10

**Prompt**:
```
你是一个文档探路 Agent。任务是查阅与 {change_summary} 相关的项目文档。

项目根目录：{project_root}

执行以下步骤：
1. Read docs/INDEX.md（如有）
2. Read docs/探路报告/INDEX.md — 检查是否有相关历史探路报告
3. Read docs/业务规则/INDEX.md — 检查是否有相关业务规则
4. Read docs/修改记录/INDEX.md — 检查是否有相关历史修改

输出：相关文档的摘要和路径列表。
```

---

## 结果合并

所有 Agent 返回后，按以下方式合并：

1. 去除重复发现（同一 file:line 的多次提及）
2. 标注不确定处为 "(待验证)"
3. 按影响范围优先级排列：安全 > 数据 > 业务逻辑 > UI
4. 汇总所有 Agent 的探路结果写入 `docs/探路报告/YYYY-MM-DD-{简述}.md`

---

## 使用方式

在 probe.md（Phase 2）的 Step 2 替换为：

```markdown
### Step 2: 并行探路

Read `scripts/spawn-probe-agents.md` 并执行，参数如下：
- project_root: "{项目根目录}"
- mcp_mysql_name: "mysql-{子项目名}"
- mcp_codegraph_name: "codegraph"
- mcp_codegraph_services: "{Step 1.8 检测到的子项目服务列表}"
- change_summary: "{本次变更概述}"
```

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 2.0.0 | 2026-06-11 | Agent A 支持多子项目分发，自动检测 codegraph 服务列表 |
| 1.0.0 | 2026-06-07 | 初始版本：4 个并行 Agent 标准化模板 |
