---
name: fullstack-flow/phases/probe
description: "代码探路：并行子代理（codegraph + mysql）探路并生成 L0-L5 结构化探路报告。只读模式。"
version: 2.1.0
tags: [fullstack, codebuddy-only, probe, read-only]
role: codebuddy-prober
model: deepseek-v4-flash
tools: [Read, Grep, Agent]
references:
  - ../references/codegraph-reference.md
  - ../references/api-fetcher-reference.md
  - ../references/mcp-tools-summary.md
mcpServers: [mysql-{子项目名}, api-fetcher-{子项目名}]
---

# Phase 2: 代码探路

> 由 Phase 1 提供原料。使用 **codegraph** MCP 主力探路，并行子代理提升效率。

## 快速概览

```
Step 0 读取状态 → Step 1 提取关键词 → Step 1.5 加载工作流模式
  → Step 2 并行探路（4 Agent）→ Step 3 写入探路报告
  → Step 4 质量门控（Phase 2.5）→ Phase 出口
```

**核心**: 4 并行 Agent（代码/DB+API/VO/文档）→ 结构化探路报告 L0-L5

## ⚡ 每日启动（开始开发前只做一次）

每次开始开发前，确认 token 有效即可直接使用 `api-fetcher` 调用后端 API：

```
▶ api_login          # 首次需手动调用，token 缓存 6 天
▶ api_list ...       # 之后自动注入 token，无需再登录
```

> 当后端未启动或 token 过期时，`api_list` 会提示 401，只需重新调用 `api_login`。
> `api-fetcher` 服务已在 `.mcp.json` 中注册，无需额外配置。

## 职责

**输入**：Phase 1 澄清后的描述（含关键词、目标模块）  
**输出**：`docs/探路报告/YYYY-MM-DD-{功能简述}.md`  
**模式**：只读（绝不 Write/Edit 代码）

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="probe", next_phase="quality-gate"

### Step 1: 提取关键词

从输入中提取探路关键词：BUG → 异常方法名/页面路由/报错信息/表名；需求 → 功能关键词/目标模块/页面。

### Step 1.5: 加载工作流模式

从 `state.yaml` 读取 `workflow.pattern`：
- 不存在 → 默认 `distribute`
- 存在 → 按该模式选择探路策略

| 模式 | 探路策略 | 子 Agent 数 |
|------|---------|:-----------:|
| `classify` | 定向探路（仅相关层） | 2~3 |
| `distribute` | 全覆盖探路（默认） | 4 |
| `adversarial` | 全覆盖 + 额外安全/性能扫描 | 5 |
| `generate` | 多角度探路（收集备选方案） | 3~4 |

> 详细模式定义见 `references/dynamic-workflows-reference.md`。

### Step 1.8: 检测可用 codegraph 服务

探路前先检测项目 `.mcp.json` 中已注册的 codegraph 子项目服务，决定 Agent A 的分发策略：

```bash
# 读取 .mcp.json，提取所有已启用的 codegraph-* 服务名
# 使用 jq 或 Grep 解析
codegraph_services=$(jq -r '.mcpServers | to_entries[] 
  | select(.key | startswith("codegraph-")) 
  | select(.value.disabled == false) 
  | .key' .mcp.json 2>/dev/null || echo "")
```

根据检测结果选择探路策略：

| 检测结果 | 策略 | Agent A 分发 |
|---------|------|-------------|
| 无子项目服务 | 标准探路 | 使用 `codegraph`（全项目扫描） |
| 1 个子项目服务 | 聚焦探路 | 使用 `codegraph-{子项目}` |
| 2+ 个子项目服务 | 并行分发探路 | 每个子项目一个 Agent A-N |

检测结果传入 `spawn-probe-agents.md` 的 `mcp_codegraph_services` 参数。

### Step 2: 并行子代理探路

Read `scripts/spawn-probe-agents.md` 并执行，根据 Step 1.8 检测结果启动探路子 Agent：

- project_root: {项目根目录}
- mcp_mysql_name: "mysql-{子项目名}"
- mcp_codegraph_name: "codegraph"（默认）
- mcp_codegraph_services: "{Step 1.8 检测到的子项目服务列表，逗号分隔}"
- change_summary: "{本次变更概述}"

> 如果检测到多个 `codegraph-{子项目名}` 服务，Agent A 将按子项目并行分发（每个子项目一个子 Agent），大幅提升跨模块探路效率。
> 如果任一 Agent 超限（max_turns 不足），回退方案为手动探路（直接使用 Read/Grep/mysql/codegraph 逐个执行，结果标注"手动探路"）。

### Step 3: 合并结果 → 生成探路报告

输出到 `docs/探路报告/YYYY-MM-DD-{功能简述}.md`：

| 层级 | 内容 | 必填 |
|------|------|------|
| **澄清摘要** | Phase 1 澄清结论（从对话传递） | ✅ |
| **L0** 层状导航 | 页面→API→Service→Mapper→DB 完整调用链 | ✅ |
| **L1** 页面入口 | 路由、Vue 组件、API 调用链路 | ✅ |
| **L2** API 入口 | Controller→Service→Mapper 每步精确行号 | ✅ |
| **L3** 实现类索引 | 所有相关类文件路径和行号 | ✅ |
| **L4** 数据库 | 表/字段/值域/关联（涉及 DB 时） | 按需 |
| **L4.5** VO/DTO 数据视图 | Controller 返回的 VO/DTO 类名+字段结构+赋值来源（涉及 DB/后端时） | ✅ |
| **L5** 前端 API/组件 | API 文件 + 组件依赖清单（涉及前端时） | 按需 |
| **测试数据样例** | 正常流程数据行 + 边界值覆盖 + 字典/枚举 DISTINCT + VO/DTO 字段映射对照（涉及 DB 时） | ✅ |
| **影响范围** | codegraph_impact 分析结果 | 推荐 |
| **API 响应样本** | 各 API 的响应结构摘要和样本文件，供 Phase 5.5 响应结构校验使用 | 推荐 |

**报告不应包含**：修改建议、实施方案（Phase 3/4 的职责）。

**API 响应样本采集说明**：在探路过程中，使用 `api-fetcher` 调用本次涉及的各 API 接口，记录响应结构摘要并保存样本到 `docs/探路报告/samples/` 目录。每个接口一个样本文件，命名格式为 `{接口路径简写}.json`（如 `user-list.json`）。样本用于 Phase 5.5 响应结构校验时对比新旧接口行为。

### Step 4: 更新探路报告索引（Read `scripts/update-index.md`）

执行 prepend 模式：
- index_path: "docs/探路报告/INDEX.md"
- row_content: "| {日期} | [{简述}](./{文件名}) | {一行摘要} |"

### Step 5: 更新工作流状态（Phase 出口，Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 `artifacts.probe_report.path = "docs/探路报告/{最新文件名}"`, `artifacts.probe_report.updated_at = 当前时间`
2. 参数: current_phase="probe", next_phase="quality-gate"
3. 额外：`metrics_snapshot.total_duration_min = 累加值`

### ⚡ 探路前必查：全路径 + 封装层检测

首次探路时必须检查以下两项，避免遗漏：

**1. 全路径覆盖** — 同一功能可能通过多个 UI 入口触发，每个入口的代码路径不同：
- 列表页内联编辑
- 详情弹框（双击行）
- 新增/编辑弹框（按钮点击）
- 自定义弹框内的表单
- 每个路径的 `formSchema`/`componentProps` 可能独立定义

**2. 封装层/中间件检测** — 数据流中可能存在拦截或包装层：
- `onChange` 是否有 wrapper（如 FormModal2 的 setFunction）
- `componentProps` 是否有包裹函数（在渲染时二次封装原始 props）
- `formActionType` 作用域是否指向正确的表单实例
- `treeParams` 是否为快照值而非响应式绑定

## 完整性自检

- [ ] L0-L3 每步有文件:行号，调用链贯通
- [ ] 使用了并行子代理（至少 codegraph + mysql）
- [ ] 已检测 codegraph 服务并选择分发策略
- [ ] 已查阅项目文档
- [ ] 影响范围评估已包含
- [ ] 不确定处标注"(待验证)"
- [ ] 子代理超限时已执行手动探路回退
- [ ] 不包含修改建议
- [ ] 涉及 DB 时已采样真实数据行（正常 + 边界）
- [ ] 涉及 DB 时已标注数据写入链路（谁写入→从哪来→经过哪些转换→最终存到哪）
- [ ] 涉及后端时已记录 VO/DTO 类名和字段结构（反映业务数据视图）
- [ ] VO/DTO 字段赋值来源已标注（直接映射/枚举转换/字典翻译/计算派生/聚合统计）
- [ ] 表字段与 VO 字段映射关系已记录
- [ ] 探路报告已遵循 communication-rules 的 normal 级别
- [ ] 探路过程交互已遵循 communication-rules 的 concise 级别

## 超限回退（子代理超限时使用）

当子代理因 max_turns 超限失败时，按以下步骤手动探路：

1. **Agent D（文档查阅）优先**：文档类探路轻量，通常不会超限，优先获取
2. **手动执行 codegraph**：直接在当前会话中使用 `mcp__codegraph__codegraph_context` 获取调用链，而非委托子代理
3. **手动执行 mysql**：直接使用 `mysql-{子项目名}.describe_table`/`execute_query` 查询表结构和数据
4. **手动 Grep 搜索**：直接搜索关键字定位代码
5. **记录超限原因**：在探路报告的质量检查章节标注 C20 异常

> 手动探路效率低于并行子代理，但能保证探路不中断。超限原因应在 Phase 6.7 复盘中沉淀为改进项。

## 约束

- 绝不写代码或修改建议。精确行号。标注"(待验证)"。
- `execute_query` 仅允许 SELECT 语句，严禁 DML（INSERT/UPDATE/DELETE）。
- 边界条件采样最多 2 个场景，避免过度查询。
- **沟通规则**：探路报告使用 **normal 级别**（完整句、文档级可读性）；探路过程中的交互使用 **concise 级别**。详见 `references/communication-rules.md`。
