---
name: fullstack-flow
description: "全栈开发流程 — 质量门控驱动的规范开发（探路→规格→计划→代码→记录→沉淀→本能→动态工作流）。核心哲学：渐进式披露 + 动态匹配 + 单一职责"
version: 26.0.0
tags: [fullstack, vue3, java, spring-boot, codebuddy-only, spec-driven]
related_skills: [systematic-debugging, codebase-knowledge-graph]
role: codebuddy-coder
skills:
  - fullstack-flow/phases/clarify
  - fullstack-flow/phases/probe
  - fullstack-flow/phases/quality-gate
  - fullstack-flow/phases/spec
  - fullstack-flow/phases/plan
  - fullstack-flow/phases/coverage-check
  - fullstack-flow/phases/code
  - fullstack-flow/phases/api-doc
  - fullstack-flow/phases/review
  - fullstack-flow/phases/record
  - fullstack-flow/phases/rule-sync
  - fullstack-flow/phases/audit
  - fullstack-flow/phases/retrospect
  - fullstack-flow/audit-flow
  - fullstack-flow/onboard-project
  - fullstack-flow/change-record
  - fullstack-flow/code-explore
  - fullstack-flow/vue-standards
  - fullstack-flow/java-dev-standards
  - fullstack-flow/java-review-standards
  - fullstack-flow/java-test-standards
references:
  - references/spec-driven-development.md
  - references/codegraph-reference.md
  - references/mcp-tools-summary.md
  - references/api-fetcher-reference.md
  - references/change-record-detailed.md
  - references/vue-reactivity.md
  - references/vue-component-data-flow.md
  - references/vue-composables.md
  - references/vue-testing.md
  - references/instinct-reference.md
  - references/dynamic-workflows-reference.md
  - references/quick-start.md
  - references/skills-navigation-index.md
  - references/scripts-ref.md
  - references/e2e-test-rules.md
  - references/out-of-scope.md
  - references/communication-rules.md
---

# 全栈开发流程（CodeBuddy 专用）

> **核心理念**：写代码前先过门控。每一步有制品、有检查、有记录，避免 AI 直接生成导致的返工和遗漏。

## 核心设计哲学：渐进式披露 + 动态匹配 + 单一职责

### 渐进式披露（Progressive Disclosure）

> 不一次性加载所有信息，而是"随用随取"——按三层结构逐层披露。

```
L1 入口层（始终加载）
├── SKILL.md 摘要（本文档）
├── 快速参考表
├── scripts/phase-entry-exit.md（入口/出口协议）
└── references/quick-start.md（快速入门指南）

L2 当前 Phase 层（Phase 入口时按需加载）
├── 当前 Phase 文档（通过 role 机制递增加载）
├── 当前 Phase 所需脚本（Read scripts/xxx.md）
└── 当前 Phase 所需参考（按需 Read references/xxx.md）

L3 深入层（显式请求或 Phase 转换时加载）
├── 非当前 Phase 文档
├── 非必需 References
└── 完整功能文档（子技能 SKILL.md）
```

**分层规则**：
- L1 在技能加载时自动注入，提供全局导航和入口协议
- L2 在 Phase.current 切换时由入口协议自动加载
- L3 通过 INDEX 导航 / 搜索 / 命令显式访问

### 动态匹配与加载（Dynamic Match & Load）

> 通过"索引建立 → 意图匹配 → 动态补充 → 重新匹配"四阶段循环，确保每个时刻上下文只包含当前所需。

```
索引建立 → 意图匹配 → 动态补充 → {匹配度足够？}
    ↑                              ↓ 是
    └──────── 重新匹配 ←────────── 执行 → Phase 转换?
                                               ↓ 是 → 回到意图匹配
                                               ↓ 否 → 继续执行
```

**四阶段**：
1. **索引建立**：`references/skills-navigation-index.md` 定义每个技能组件的前置依赖、适用场景和标签
2. **意图匹配**：Phase 入口时，按 `phase.current` + `task.type` + 问题类型自动匹配所需组件列表
3. **动态补充**：按匹配结果，通过 `Read` 按需加载当前 Phase 所需脚本和参考
4. **重新匹配**：Phase 出口时重新评估上下文，预加载下一 Phase 可能需要的组件

**现有实现**：
- 动态工作流模式（`/workflow-pattern`）按 `task.type` 匹配 4 种执行模式
- 上下文优化策略（延迟加载/子 Agent 摘要/压缩）管理上下文预算
- INDEX 导航体系（7 类 INDEX.md）支持按需查阅历史制品

### 单一职责（Single Responsibility）

> 每个技能组件（Phase/子技能/脚本/Agent）只解决一类特定的任务，避免构建试图包揽一切的"万能 Agent"，从而保证模型输出质量和可维护性。

**设计原则**：

```
                    ┌─────────────────────────┐
                    │    用户需求（大而全）     │
                    └────────┬────────────────┘
                             │ 分解
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ Phase 1  │  │ Phase 2  │  │ Phase 3  │
        │ 描述澄清  │→ │ 代码探路  │→ │ 规格澄清  │
        │ (只澄清)  │  │ (只探路)  │  │ (只定规格) │
        └──────────┘  └──────────┘  └──────────┘
              │              │              │
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ Agent A  │  │ Agent B  │  │ Agent C  │
        │ 代码结构  │  │ DB+VO   │  │ 影响范围  │
        │ (只追踪)  │  │ (只采样)  │  │ (只分析)  │
        └──────────┘  └──────────┘  └──────────┘
```

**三条规则**：

1. **Phase 单一职责**：每个 Phase 只完成一个明确的制品输出。Phase 1 只做"描述澄清"，Phase 2 只做"代码探路"，Phase 5 只做"编码实现"。不跨Phase越界。
   
2. **Agent 单一职责**：Phase 2 的 4 个并行子 Agent 各自只负责一个探路维度（代码结构/DB+VO/影响范围/文档查阅），Agent prompt 中不包含其他维度的指令。Phase 5 的 FE/BE Agent 只负责各自的技术栈。
   
3. **脚本单一职责**：`scripts/` 下的每个脚本只解决一个操作。`phase-entry-exit.md` 只管入口/出口协议，`update-index.md` 只管 INDEX 更新，`build-fix.md` 只管编译修复循环。不把多个职责塞进一个脚本。

**保证模型输出维度**：

> 单一职责的直接目的是**约束模型输出范围**。当 Agent 的职责范围明确时，模型不需要在"全局上下文"中判断该做什么，只需在"特定领域的狭窄范围"内输出最优解。

| 场景 | 宽职责（应避免） | 窄职责（推荐） |
|------|----------------|---------------|
| 探路 Agent | "分析这个功能的代码、数据库、API 和影响范围" | "只追踪从 Controller 到 Mapper 的调用链，输出 file:line" |
| 审核 Agent | "审核前端、后端、API 文档、测试覆盖率" | "只审核 Vue 组件是否符合 Composition API 规范" |
| 脚本 | "这个脚本同时处理入口/出口/INDEX/决策日志" | "这个脚本只处理入口/出口协议，INDEX 更新由另一个脚本负责" |

**现有实现**：
- 14 个 Phase 各自独立，职责边界明确
- Phase 2 的 4 并行 Agent 各自只负责一个维度
- Phase 5 的 FE/BE Agent 分离后端/前端职责
- `scripts/` 下 6 个脚本各司其职，无交叉职责
- 入口/出口协议不允许修改代码（职责隔离）

## 快速参考

| 阶段 | 核心动作 | 门控 | 产出 |
|:----:|---------|:----:|------|
| **Phase 1** 描述澄清 | 环境预检 + 多轮 Q&A + 用户确认摘要 | Gate 0 | 澄清描述 + 关键词 |
| **Phase 2** 代码探路 | 4 并行 Agent 探路 | — | 探路报告 |
| **Phase 2.5** 质量门控 | **不变性门(I1/I2/I9/I10)** + 35 项完整性检查 | **Gate 1** (100%) | 门控报告 |
| **Phase 3** 规格澄清 | 交互式 Q&A + 用户确认 Scope/AC | — | 规格文档 |
| **Phase 4** 实施计划 | 任务分解 + 风险预评估 + 用户确认计划 | — | 实施计划 |
| **Phase 4.5** 覆盖验证 | **不变性门(I3/I4)** + 5 条件 AND 检查 | **Gate 2** (100%) | 覆盖报告 |
| **Phase 5** 最小修改 | TDD + Build-Fix + **不变性门(I5/I6)** | **Gate 2.5** | 修改后的代码 |
| **Phase 5.6** API 文档 | 按接口生成文档 | — | 接口文档 |
| **Phase 5.5** 代码审核 | **不变性门(I7/I8)** + 一致性核对 + E2E 验证 | **Gate 3** | 审核结果 |
| **Phase 6** 修改记录 | 代码对比 + 回滚方案 | **Gate 4** | 修改记录 |
| **Phase 6.5** 规则同步 | 业务规则更新 + Instinct | — | 规则文件 |
| **Phase 6.6** 规则审计 | 6 类模式扫描 | **Gate 5** | 审计报告 |
| **Phase 6.7** 复盘回顾 | 五维分析 + Instinct 提取 + 失败案例 | — | 复盘记录 + 失败案例 |

## 职责分工

## 职责分工

| 角色 | 职责 |
|------|------|
| **用户/测试** | 提交 BUG 描述 / 新增功能描述（不限格式，CodeBuddy 负责澄清） |
| **CodeBuddy** | Phase 1 → 2 → 2.5 → 3 → 4 → 4.5 → 5 → 5.6 → 5.5 → 6 → 6.7 → 6.5（→ 6.6 阶段性执行） |

CodeBuddy 收到任务后先执行 Phase 1 描述澄清，不依赖任何外部预生成文档。

## 流程速览

| Phase | 制品 | 核心动作 | 门控 |
|-------|------|----------|------|
| **Phase 1** 描述澄清 | 澄清后描述（对话确认） | 环境预检 → 多轮交互（≤5 问题/轮），提取 ≥3 个探路关键词，**输出摘要后用户确认** | Gate 0: 自检关键词充足 + 用户已确认 |
| **Phase 2** 代码探路 | `docs/探路报告/*.md` | 4 并行 Agent（代码/DB+API采样+VO视图/影响/文档），三层数据采样（API→VO→DB），调用链标注 file:line | — |
| **Phase 2.5** 质量门控 | 追加到探路报告 | 不变性门(I1/I2/I9/I10) + 35 项完整性检查（含 L4.5 VO/DTO 视图 + 表↔VO 映射） | Gate 1: 通过率 100% |
| **Phase 3** 规格澄清 | `docs/规格文档/*.md` | 交互式 Q&A，输出 What/Goal/Scope/AC，**用户确认 Scope/AC** | — |
| **Phase 4** 实施计划 | `docs/实施计划/*.md` | 分解 T### [P] 任务，标注 AC，**用户确认计划** | — |
| **Phase 4.5** 覆盖验证 | 追加到实施计划 | 不变性门(I3/I4) + AC 覆盖率 / 文件覆盖率 / 术语一致性 / 数据追溯完整性 | Gate 2: 通过率 100% |
| **Phase 5** 最小修改 | 修改后的代码 | FE/BE 并行子代理，TDD 先行（`/java-test` / `vue-standards`），Build-Fix 自动修复，不变性门(I5/I6)编译测试检查 | Gate 2.5: 测试全部通过 |
| **Phase 5.6** API 文档生成 | `docs/接口文档/*.md` | 根据实际代码主动生成/更新接口文档，一个接口一个文档文件 | — |
| **Phase 5.5** 文档代码审核 | 即时审核结果（Phase 6 纳入记录） | 不变性门(I7/I8)文档覆盖验证 + FE/BE 并行审核 + Playwright E2E 验证 + 文档完整性全覆盖扫描 | Gate 3: 一致性自检 + 文档全覆盖 |
| **Phase 6** 修改记录 | `docs/修改记录/*.md` | 前后代码对比 + 回滚方案（不使用 git） | Gate 4: 完整性自检 |
| **Phase 6.5** 业务规则同步 | `docs/业务规则/*.md` | 评估并更新本次修改涉及的业务规则 | — |
| **Phase 6.6** 业务规则审计 | `docs/业务规则/审计/*.md` | 扫描 6 类业务规则模式全覆盖（状态/枚举/权限/数据过滤/前端条件/@Dict），完整审计阶段执行 | Gate 5: 6 类全覆盖 |
| **Phase 6.7** 复盘回顾 | `docs/复盘记录/*.md` + `docs/失败案例/*.md` | 复盘四问 → 五维分析（流程/质量/惯例/技能/根因）→ 失败案例 → 沉淀改进项 → 汇总检查 | — |

> 各 Phase 详细指南见 `phases/phase-*.md`。

## 强制执行规则

1. **Phase 1 必须执行环境预检 + 描述澄清 + 用户确认摘要**：Phase 1 开始后先检查项目工具链和 MCP 服务就绪状态（Step 0.5），然后提取 ≥3 个关键词，输出澄清摘要后必须获得用户确认方可进入 Phase 2
2. **Phase 2** 必须使用 MCP 工具链自行探路，生成 `docs/探路报告/`，不得跳过
3. **Phase 2 探路涉及 DB 时必做数据采样**：使用 `codegraph` 定位 Controller 层 API 接口，获取 VO/DTO 字段结构反映业务数据视图；使用 `mysql-{子项目名}.execute_query` 查询真实数据行（正常流程+边界DISTINCT）；对照表字段与 VO 字段记录映射关系。追踪数据写入链路，输出到探路报告 L4、L4.5 和测试数据样例章节
4. **Phase 2.5** 质量门控必须执行，通过率 100% 方可进入 Phase 3
5. **Phase 3 必须输出 What/Goal/Scope/AC 并由用户确认**：读取探路报告，输出规格文档，Scope 和 AC 必须获得用户确认方可进入 Phase 4
6. **Phase 4 必须输出任务清单并由用户确认**：输出 T### [P] 标准化任务清单，每任务标注 AC，实施计划和风险评估必须获得用户确认方可进入 Phase 5
7. **Phase 4.5** 覆盖验证必须执行，AC 覆盖率 100% 方可进入 Phase 5
8. **Phase 5 必须执行 TDD 流程，执行测试前须经用户确认**：涉及代码修改时，后端 Controller 先写 ApiTest（Red）再实现代码（Green），前端组件/composable 先写 spec（Red）再实现（Green）。测试全部通过（Gate 2.5）方可进入 Phase 5.6。详细规范见 `vue-standards` 第 4 章和 `code.md` 第 5.3.5 节。
9. **Phase 5.6** 必须为本次修改涉及的所有新增/变更的接口生成 API 文档，一个接口一个文档文件，包含完整的请求/响应说明、业务规则和调用示例。完成后同步更新 `docs/接口文档/INDEX.md` 和 `docs/接口文档/00-全API索引.md`
10. **Phase 5.5** 必须审核探路报告/规格/API 接口文档与代码一致性，不一致必须更新
11. **Phase 6** 必须按模板输出修改记录，包含回滚方案
12. **Phase 6.5** 必须评估本次修改是否涉及业务规则，涉及则更新
13. **Phase 6.6** 必须扫描 6 类业务规则模式全覆盖，完整审计建议阶段性执行
14. 服务由 IDE 管理 — 禁止在终端中用 `mvn spring-boot:run` / `java -jar` 启动
15. 修改记录输出到 `docs/修改记录/`，按模板格式
16. **每个 Phase 完成后必须更新对应 INDEX.md**（探路报告/规格文档/实施计划/修改记录/业务规则/接口文档）
17. **Phase 5.6 生成 API 文档后**：新增接口模块时同步更新 `docs/接口文档/INDEX.md`
18. **Phase 6.5 处理过期规则**：引用源头已删除的规则标注"已废弃"并直接删除
19. **docs/只维护有效文档**：fullstack-flow 流程不读取的目录/文件直接删除，不保留一次性或人工文档
20. **Phase 2 探路必须覆盖全路径**：同一功能的所有 UI 入口（列表页/详情弹框/编辑弹框）和数据流封装层（onChange wrapper、componentProps 包裹、formActionType 作用域）
21. **Phase 5 修改 async 函数必须做竞态分析**：API 返回后检查值是否已变，避免旧响应覆盖新状态
22. **Phase 6 完成后必须执行"复盘触发检查"四问**：质量门控失败/审核发现缺陷/反复修改/暴露技能盲区，任一"是"则进入 Phase 6.7
23. **Phase 6.7** 复盘回顾必须在以下情况执行：质量门控失败、审核发现缺陷、反复修改同一问题、暴露技能流程盲区
24. **复盘发现必须沉淀为可验证的改进**：每条发现至少对应一项 SKILL.md 规则更新 / Phase 检查项新增 / Memory 记录 / CHANGELOG 条目变更
25. **每个 Phase 入口必须读取 `.codebuddy/workflow/state.yaml`**，确认当前阶段状态正确，如检测到未完成的工作流则进入恢复模式
26. **每个 Phase 出口必须更新 `.codebuddy/workflow/state.yaml`**，写入制品路径和门控结果，设置下一阶段为 pending
27. **Phase 3/4 有设计决策时必须记录到 `decisions/`**：用户明确选择 / Agent 推荐替代方案 / 新增设计模式时强制记录
28. **Phase 6.7 复盘时同步 Memory**：复盘五维分析中涉及项目惯例/流程改进/技术沉淀的，由 CodeBuddy 自动写入项目对应的 Memory 目录并更新 MEMORY.md
29. **state.yaml 禁止手动编辑**，只能通过 Phase 入口/出口协议自动更新
30. **Phase 1/3 交互式 Q&A 必须验证用户回答**：用户提供的模块名、页面路径、方法名、表名等具象信息，CodeBuddy 必须使用 Read/Grep/codegraph 等工具在 1 轮内验证其真实性。验证失败时标注"(待验证)"并继续追问核实，不得直接采信无法验证的用户陈述。
31. **验证发现问题必须回退对应 Phase**：用户/测试人员在任意阶段（含 Phase 6 完成后）发现问题，按类型回退到对应阶段重新执行并更新制品，完成后重新走后续所有 Phase，不得在流程外做 ad-hoc 修复。判定规则：
    - 代码逻辑错误 → 回退 Phase 5（代码修改），重走 Phase 5.6 → Phase 5.5 → Phase 6
    - 方案设计缺陷 → 回退 Phase 4（实施计划），重走 Phase 4.5 → Phase 5 → Phase 5.6 → Phase 5.5 → Phase 6
    - 需求/规格遗漏 → 回退 Phase 3（规格澄清），重走 Phase 3 → Phase 4 → Phase 4.5 → Phase 5 → Phase 5.6 → Phase 5.5 → Phase 6
    - 探路/根因误判 → 回退 Phase 2（代码探路），重走 Phase 2.5 → Phase 3 → Phase 4 → Phase 4.5 → Phase 5 → Phase 5.6 → Phase 5.5 → Phase 6
32. **Phase 5.5 必须执行端到端场景验证**：Phase 5.5 审核通过后，必须按 Phase 3 规格的 AC 逐条模拟用户操作路径执行端到端验证。无法依赖交互的步骤标注"需人工测试"。验证通过后方可进入 Phase 6。验证发现与 AC 不符时标记"回退 Phase 5"。
33. **验证回退时必须更新 state.yaml**：回退时设置 `phase.current = "{回退 Phase ID}"`, `phase.status = "in_progress"`, `progress.phases_blocked` 追加回退原因。回退重新完成后更新所有相关制品，按正常出口协议推进。
34. **修改记录末尾追加验证变更日志**：Phase 6 生成修改记录时，如存在 Phase 6 之后的验证→回退→修复循环，在修改记录末尾追加"验证变更日志"章节，按时间顺序记录每次验证发现和修复摘要，格式：
    ```
    ## 验证变更日志
    | 轮次 | 发现 | 修复 | 回退阶段 |
    |------|------|------|---------|
    | 1 | {描述} | {修复摘要} | Phase 5 |
    ```
35. **Phase 5.5 必须执行文档完整性扫描**：Phase 5.5 审核时，必须扫描 `docs/接口文档/` 和 `docs/业务规则/` 是否覆盖了本次修改涉及的所有 Controller/API/业务规则。使用 `Grep` 搜索前端 API 调用路径和后端 Controller `@RequestMapping`，与文档逐条比对。缺失文档在当前 Phase 5.5 内补充，不得留到后续 Phase。
36. **Phase 5 涉及后端 Controller 时必按 TDD 流程创建测试类**：先创建 `{ControllerName}ApiTest.java`（Red）运行确认失败，再实现 Controller/Service 代码（Green）使测试通过。测试数据通过 API 链调用造数，禁止直接 INSERT 数据库。
37. **Phase 6.7 复盘时必须执行 Instinct 提取**：完成五维分析后，必须从本次实施中识别可复用模式并生成 Instinct 文件。详见 `references/instinct-reference.md`。
38. **Phase 6.5 涉及业务规则变更时，同步提取领域 Instinct**：常量/枚举、状态流转、权限约束、数据过滤、前端条件渲染等业务规则类型，必须在规则更新后生成对应的 Instinct 文件。
39. **每个 Phase 出口必须审计 token 用量和 MCP 工具使用**：在 Phase 出口协议中追加 `metrics_snapshot.tools_used.{phase-id}` 记录（包含每个 MCP 工具的调用次数和 token 消耗）和 `metrics_snapshot.token_audit.{phase-id}` 记录（包含 input_tokens、output_tokens、mcp_tool_calls 和 key_files_loaded）。累计值记入 `metrics_snapshot.token_accumulated`。
40. **子 Agent 返回必须限制摘要大小**：Phase 2/5 派发的子 Agent 返回主会话时，输出 ≤1K tokens 的"任务完成清单 + 关键决策点"摘要，不得返回完整对话日志。
41. **Instinct 存放规则**：项目级 `.codebuddy/instincts/personal/`，全局级 `~/.CODEBUDDY/instincts/personal/`。scope 字段区分 project/global。
42. **置信度管理**：首次提取置信度 0.60，每次复现 +0.05~0.10，用户手动确认 +0.15。最大 1.00。
43. **Instinct 提升规则**：置信度 ≥ 0.80 的 project scope 建议提升为 global。建议用户执行 `/instinct-promote`。
44. **建立定期 Skill 健康度检查制度**：每 30 天（或每 10 次开发工作流执行后）必须运行一次 `/skill-health-check`，检查技能文件一致性、过时规则、规则冲突和失败案例趋势。健康度报告输出到 `docs/技能审计/健康度报告-YYYY-MM.md`。
45. **复盘时必须评估是否产出结构化失败案例**：BUG 修复复盘和同类问题再次出现时必须创建失败案例文件到 `docs/失败案例/`，使用标准化模板。其他复盘类型建议创建。失败案例必须关联对应复盘记录。
46. **每次复盘后执行"汇总检查"**：Phase 6.7 Step 5 必须读取最近 3 次复盘记录，检查是否存在重复出现的失败模式。相同模式出现 2 次以上时自动触发增强沉淀（不等待定期健康度检查）。
47. **失败案例归档规则**：复盘产出的失败案例文件存入 `docs/失败案例/YYYY/` 按月归档。创建后同步更新 `FAILURE-INDEX.md`。失败案例不可删除，只能标注"已解决"。
48. **流程脚本化原则**：所有固定的、重复的步骤应引用 `scripts/` 目录下的结构化指令文档，而非在每个 Phase 文档中重复编写。Phase 文档在入口/出口/INDEX 更新/环境预检/Build-Fix/并行 Agent 启动等场景应优先 Read 对应脚本并传参执行。
49. **脚本库维护规则**：修改 `scripts/` 下的脚本后，必须在 CHANGELOG 中记录版本变更，并同步更新所有引用了该脚本的 Phase 文档。新增 Phase 时如需入口/出口协议，直接 Read `scripts/phase-entry-exit.md`。
50. **不变性门必须在对应 Phase 执行**：Phase 2.5 入口后必须先执行 I1/I2/I9/I10 不变性门自动验证（grep/bash 命令），通过后方可进入人工 35 项检查。Phase 4.5 入口后必须先执行 I3/I4 不变性门验证。Phase 5.5 入口后必须先执行 I7/I8 不变性门验证。不变性门失败直接回退上一 Phase，不进入后续检查。详见 `scripts/invariant-gates.md`。
51. **Agent 必须遵守决定不做清单**：Agent 在任何 Phase 不得执行 `references/out-of-scope.md` 中声明的禁止操作（如直接修改数据库 schema、部署到生产环境、修改非 Scope 文件等）。违反处理：立即停止当前操作，记录越界行为到修改记录，回退到对应 Phase 重新执行。
52. **Agent 输出必须遵循沟通规则**：Agent 的输出应遵循 `references/communication-rules.md` 中的沟通规范。基本原则：去填充词、去客套话、用片段、技术术语精确。交互式 Q&A 和审核意见使用 concise 级别；探路报告、规格文档、修改记录使用 normal 级别。安全警告和不可逆操作使用完整清晰表达。详见 `references/communication-rules.md`。

## 为什么需要 Phase 3（规格澄清）和 Phase 4（实施计划）？

没有规范驱动 → 靠感觉判断修改范围、无明确验收标准、遗漏场景、方向错了才返工。
有规范驱动 → 先说清楚边界、规格中定义 AC、计划阶段覆盖全部分支、规格澄清阶段纠正方向。

**质量门控（Phase 2.5 + Phase 4.5）不是可选项。**

## 原则解决层级

当设计原则之间发生冲突时，按以下优先级判定：

| 优先级 | 原则类别 | 说明 |
|--------|---------|------|
| **P0** | 安全与数据完整性 | 始终优先于其他所有原则 |
| **P1** | 用户意图与规格 AC | 用户确认的验收标准高于内部优化 |
| **P2** | 流程纪律 | 规范驱动 > 直接编码，门控不可跳过 |
| **P3** | 最小修改 | 改动范围不超出规格 Scope |
| **P4** | 代码质量 | 顺手修复/重构/代码规范 |

**常见冲突裁决**：
- **最小修改 vs 顺手修复**：除非涉及安全或导致 Phase 5.5 退回，否则不改无关代码
- **数据采样精度 vs 效率**：简单文案/配置修改可跳过多层采样，仅做文件定位；涉及业务逻辑变更时严格执行完整采样
- **门控阈值统一**：Phase 2.5（100%）和 Phase 4.5（100%）均要求完美覆盖，探路报告必须完整

## 缺失维度（已纳入流程）

以下非功能维度在开发流程中被系统性地覆盖：

| 维度 | 覆盖阶段 | 检查内容 |
|------|---------|---------|
| **安全** | Phase 5.5 BE 审核 | `@RequiresPermissions` 权限注解完整性、SQL 注入风险、认证绕过 |
| **性能** | Phase 2 探路建议 | N+1 查询检测、分页实现方式、批量操作效率 |
| **可测试性** | Phase 5 代码 + Phase 5.5 审核 | Phase 5 对涉及 Controller 创建/更新测试类，验证正常/异常/边界流程；Phase 5.5 BE 审核检查测试覆盖完整性；编译+测试通过验证 |
| **向后兼容** | Phase 3 Scope | 明确声明 API/数据格式的兼容性要求 |
| **可观察性** | Phase 5.5 BE 审核 | 新增接口的日志记录、异常信息完整性 |
| **API 文档** | **Phase 5.6** | **每个新增/变更接口生成独立文档，含请求/响应/规则/调用示例** |
| **知识沉淀** | **Phase 6.7 / 6.5** | **每次实施后提取可复用模式生成 Instinct（直觉），形成自学习闭环** |
| **失败案例库** | **Phase 6.7** | **结构化记录失败案例（BUG/设计/流程），支持检索和趋势分析** |
| **Skill 健康度** | **月度/10次执行** | **定期检查技能文件一致性、过时规则、规则冲突和失败案例趋势** |
| **数据库迁移** | Phase 2 L3 链路 | 数据写入链路标注，确认不影响现有数据路径 |
| **依赖管理** | Phase 5 约束 | 不引入计划外新依赖 |
| **国际化** | Phase 2 探路标记 | 涉及前端文案修改时标注 i18n 键名 |
| **Token 审计** | **Phase 出口协议** | **每次 Phase 退出记录 input/output token 用量 + MCP 工具调用次数，逐阶段累积，辅助上下文压缩决策** |
| **工具使用审计** | **Phase 出口协议** | **每次 Phase 退出记录 MCP 工具名称/调用次数/token 消耗，支持效率分析** |
| **流程脚本化** | **scripts/ 目录** | **将固定/重复步骤提取为结构化指令脚本，集中引用而非复制粘贴** |
| **单一职责** | **Phase/Agent/脚本层级** | **每个组件只解决一类任务，避免万能 Agent，保证模型输出维度** |

> 安全/性能/可测试性为 P0 维度，在 Phase 2.5 质量门控和 Phase 5.5 审核中作为必查项。其他维度在对应阶段按需检查。

## Agent 记忆与状态管理

fullstack-flow 工作流状态由 `.codebuddy/workflow/` 集中管理，取代隐式的制品存在性推断。

### 目录结构

```
.codebuddy/workflow/
├── state.yaml                   # 当前工作流状态（单一事实源）
├── state.schema.yaml            # 验证规则（Phase 入口/出口自检）
├── decisions/
│   ├── INDEX.md                 # 决策日志索引
│   └── YYYY-MM-DD-{简述}.md     # 单个决策记录
├── metrics/
│   ├── gates.yaml               # 门控通过/失败历史
│   └── phases.yaml              # Phase 耗时/重试统计
└── sessions/                    # 会话记录（自动生成）
```

### Phase 转换协议

每个 Phase 按以下协议操作 state.yaml。入口/出口协议已标准化，各 Phase 文档中仅引用本表，不再重复书写。

### 入口协议（Phase 文档 Step 0）

所有 Phase 入口均执行以下步骤（仅 Phase 1 增加初始化/恢复检测）。各 Phase 文档中改为引用 `scripts/phase-entry-exit.md` 并传参，避免重复编写。

```
1. Read `.codebuddy/workflow/state.yaml`
2. 验证 phase.current == "{当前 Phase ID}"
3. 验证前置制品存在（如有）
4. 设置 phase.status = "in_progress", phase.started_at = 当前时间
5. 更新 session.last_activity = 当前时间
```

### 出口协议（Phase 文档末尾）

所有 Phase 出口均执行以下步骤：

```
1. 更新 artifacts.{对应制品}：更新制品路径和时间戳
2. 更新门控结果（Gate 阶段）：status/score/failed_items
3. 设置 phase.status = "completed", phase.completed_at = 当前时间
4. progress.phases_completed.append("{当前 Phase ID}")
5. 设置 phase.current = "{下一 Phase ID}", phase.status = "pending"
   - Gate 失败时：phase.current = "{回退 Phase ID}", phases_blocked 记录原因
   - 工作流结束（Phase 6.6）：phase.current = null, phase.status = "completed"
6. metrics_snapshot.phase_durations[{当前 Phase ID}] = 耗时分钟数
7. gates_summary 自动更新（Gate 阶段）
8. 更新 session.last_activity = 当前时间
```

### Phase 转换速查

| Phase | 入口验证 | 出口制品 | 下一阶段 | Gate 回退 |
|-------|---------|---------|---------|----------|
| clarify | 初始化/恢复 | task.keywords | probe | — |
| probe | current==probe | probe_report.path | quality-gate | — |
| quality-gate | current==quality-gate | gate_2_5 | spec | <70% → probe |
| spec | +验证探路报告存在 | spec.path + decisions | plan | — |
| plan | +验证 spec 存在 | plan.path + decisions | coverage-check | — |
| coverage-check | +验证 plan 存在 | gate_4_5 | code | 失败 → plan |
| code | +验证 gate_4_5 通过 | code_changes[] | api-doc | — |
| api-doc | +验证 code_changes 非空 | api_docs.paths[] | review | — |
| review | +验证 api_docs 非空 | review + gate_3 | record | 有问题 → code |
| record | +验证 gate_3 通过 | change_record.path | retrospect 或 rule-sync | 复盘四问分支 |
| retrospect | current==retrospect | retrospect.path | rule-sync 或结束 | — |
| rule-sync | +验证 change_record 存在 | rule_sync | audit | — |
| audit | current==audit | audit_report + gate_5 | 结束 | — |

### 决策日志触发条件

| 场景 | 必须记录 | 推荐记录 |
|------|---------|---------|
| 用户从多个选项中明确选择 | ✅ | - |
| Agent 推荐 + 用户确认的替代方案 | ✅ | - |
| 新增设计模式或组件约束 | ✅ | - |
| 标准方案（最常见） | - | ✅ |
| 纯技术实现细节 | - | ❌ |

### 复盘触发条件汇总

复盘入口由 3 个独立条件和 1 个强制规则驱动，统一判定表：

| 来源 | 触发条件 | 标记位置 | 目标 |
|------|---------|---------|------|
| Phase 2.5 质量门控 | 失败项 ≥ 1 个 | 门控输出追加 `⚠️ 需复盘` | Phase 6 Step 6 四问读取 |
| Phase 5.5 代码审核 | 代码质量 ≥ 1 个 ⚠️ 或 不一致 ≥ 2 个 | 审核输出标注 `⚠️ 需复盘` | Phase 6 Step 6 四问读取 |
| Phase 6 复盘四问 | Q1-Q4 任一"是" | 修改记录追加复盘触发章节 | Phase 6.7 入口 |
| Phase 6.6 审计 | 未归档规则 ≥ 5 条 | 审计报告标记 | 强制触发 Phase 6.7 |
| **强制执行** | BUG 修复 / 大规模重构 / 重复 BUG / 用户要求 | SKILL.md 规则 22 | 直接进入 Phase 6.7 |

## 上下文与 Token 管理

fullstack-flow 流程多阶段连续执行，上下文窗口会随探路报告、规格文档、代码修改等内容增长。通过自动压缩 + Token 审计双重机制管理上下文。

### 自动压缩

| 项 | 说明 |
|------|------|
| 触发阈值 | 上下文达到 130K tokens（约 13%，模型总容量 1000K） |
| 配置方式 | `.codebuddy/.env` → `CODEBUDDY_AUTOCOMPACT_PCT_OVERRIDE=13` |
| 保留内容 | 探路报告摘要、AC 清单、修改记录 |
| 丢弃内容 | 冗余对话历史 |
| 手动压缩 | `/compact 仅保留当前 Phase 制品` |
| 建议时机 | Phase 5 完成后 / Phase 3 开始前 |

### Token 审计

每次 Phase 出口记录到 `metrics_snapshot.tools_used` 和 `metrics_snapshot.token_audit`：

```yaml
metrics_snapshot:
  tools_used:
    clarify:
      mcp_calls:
        - { tool: "codegraph_context", count: 2, input_tokens: 1200, output_tokens: 800 }
      key_files_loaded: ["state.yaml"]
  token_audit:
    clarify: { input_tokens: 3200, output_tokens: 1500, mcp_tool_calls: 2, key_files: [state.yaml] }
  token_accumulated:
    total_tokens: 125600
    phases_count: 6
```

完整工作流结束后汇总到 `.codebuddy/workflow/metrics/token-audit.yaml`。

### 上下文优化策略

| 策略 | 触发时机 | 操作 |
|------|---------|------|
| **制品文件化** | 每个 Phase 出口 | 关键信息写文件，后续 Phase 用 `Read` 读取 |
| **子 Agent 摘要** | Phase 2/5 | 返回 ≤1K tokens 摘要，禁止完整日志 |
| **延迟加载** | Phase 入口 | 按需读取，不一次性加载所有制品 |
| **压缩建议** | Phase 出口 | >200K 时提示 `/compact` |
| **重复检测** | Phase 入口 | 避免重复加载已存在于上下文中的内容 |

## Instinct（直觉）机制

fullstack-flow 集成了持续学习机制：每次走完一个工作流后，系统从执行过程中识别可复用的模式，生成 Instinct（直觉）文件，并逐步进化成技能/命令。

### 存储路径

```
.codebuddy/instincts/              # 项目级
├── INDEX.md
├── personal/                      # 自动学习
│   └── YYYY-MM-{id}.yaml
└── inherited/                    # 导入

~/.CODEBUDDY/instincts/           # 全局级
├── INDEX.md
├── personal/
└── inherited/
```

### 生命周期

```
执行 → Phase 6.7 复盘 → 模式提取 → Instinct (YAML) → `/instinct-status`
                                                          ↓
                                                   置信度 ≥ 0.80
                                                          ↓
                                             `/instinct-promote` → 全局级
                                                          ↓
                                             `/instinct-evolve` → 技能/命令
```

### 新增命令

| 命令 | 功能 |
|------|------|
| `/instinct-status` | 显示当前项目和全局的所有 Instinct（置信度排序） |
| `/instinct-evolve` | 聚类相关 Instinct，生成技能/命令/Agent |
| `/instinct-promote` | 将项目级 Instinct 提升为全局级 |
| `/aside` | Phase 执行中临时回答侧问题，不丢失当前上下文 |
| `/context-budget` | 分析 Token 消耗占比，辅助压缩决策 |
| `/e2e` | 生成 Playwright 端到端测试脚本验证 UI 交互 |
| `/test-coverage` | 运行测试并报告代码覆盖率 |
| `/quality-gate` | 独立运行质量门控检查（不依赖 Phase 流程） |
| `/checkpoint` | 在当前 Phase 标记里程碑，保存进度快照 |
| `/java-test` | Java 后端 TDD 流程（Red→Green→Refactor） |
| `/save-session` | 保存当前会话快照，用于长工作流中断后恢复 |
| `/resume-session` | 从会话快照恢复工作流，继续中断的 Phase |
| `/workflow-pattern` | 查看/切换动态工作流模式 |
| `/skill-health-check` | 运行 Skill 健康度检查（一致性/过时规则/失败案例趋势），输出到 `docs/技能审计/` |
| `/failure-cases [filter]` | 按领域/严重度/日期检索失败案例库，支持全文搜索 |
| `/compress-knowledge [file\|--all\|--stats]` | 压缩知识文件（memory/instincts），去掉填充词保留技术信息，可节省 ~46% 输入 token |

> 详细定义和格式见 `references/instinct-reference.md`。

## 动态工作流（Dynamic Workflows）

fullstack-flow 支持根据任务类型即时编排子 Agent 并行工作模式，无需配置即可在 Phase 1 完成后自动选择最优模式。

### 内置模式

| 模式 | 英文 | 适用场景 | 自动选择条件 |
|------|------|---------|:-----------:|
| **分类并执行** | classify | BUG 修复、范围明确的任务 | `task.type == "bug"` |
| **分发并汇总** | distribute | 新功能、复杂多模块（默认） | `task.type == "feature"` 或 `complex` |
| **对抗性验证** | adversarial | 优化、重构、安全敏感修改 | `task.type == "optimization"` |
| **生成并筛选** | generate | 技术选型、复杂算法、重构 | `task.type == "refactor"` |

### 覆盖的 Phase

| Phase | classify | distribute | adversarial | generate |
|-------|:--------:|:----------:|:-----------:|:--------:|
| Phase 2 探路 | 定向探路 | 全覆盖 4 Agent | 全覆盖 + 安全扫描 | 多角度收集 |
| Phase 5 代码 | 单 Agent | FE/BE 并行 | 实现+审核循环 | 多方案对比 |
| Phase 5.5 审核 | 标准 | 标准 | 独立审核 Agent | 方案对比审核 |

### 模式切换

```
# 查看当前模式
/workflow-pattern

# 手动切换
/workflow-pattern distribute
/workflow-pattern adversarial

# Java TDD 快捷方式
/java-test OrderController         # 直接进入 TDD Red→Green 流程
/java-test 用户管理列表查询接口     # 自动搜索对应 Controller
```

> 详细定义见 `references/dynamic-workflows-reference.md`。

## 交付物路径

```
docs/INDEX.md                          ← 文档中心入口（仅 active 制品）
docs/探路报告/INDEX.md
docs/规格文档/INDEX.md
docs/实施计划/INDEX.md
docs/修改记录/INDEX.md
docs/业务规则/INDEX.md
docs/接口文档/INDEX.md
docs/复盘记录/INDEX.md
docs/失败案例/FAILURE-INDEX.md           ← 失败案例索引
docs/技能审计/INDEX.md                   ← 技能审计报告索引
docs/接口文档/00-全API索引.md
references/quick-start.md                 ← L1 快速入门指南
references/skills-navigation-index.md     ← 技能导航索引（组件依赖 + 动态匹配规则）
references/mcp-project-templates/        ← 项目级 MCP 配置模板
scripts/phase-entry-exit.md            ← Phase 入口/出口协议脚本
scripts/update-index.md                ← INDEX.md 统一更新脚本
scripts/build-fix.md                   ← Build-Fix 自动循环脚本
scripts/env-check.md                   ← 环境预检脚本
scripts/spawn-probe-agents.md          ← 并行探路子 Agent 启动模板
scripts/create-decision.md             ← 决策日志创建脚本
.codebuddy/instincts/INDEX.md          ← Instinct 索引（直觉文件）
.codebuddy/workflow/state.yaml         ← 工作流状态（含 phase.current + token_audit + tools_used）
.codebuddy/workflow/metrics/token-audit.yaml  ← Token 审计明细（含 MCP 工具使用记录）
```

## 如何贡献改进

fullstack-flow 是一个自学习流程，欢迎持续改进。

### 发现流程缺陷

通过以下方式报告缺陷：

1. **运行审计**：执行 `/fullstack-flow/audit-flow` 自动扫描 20 项检查维度
2. **审计自动修复**：低风险问题（注册缺失、绝对路径）自动修复
3. **审计人工处理**：高风险问题（MCP 不一致）输出"需人工处理"，用户确认后执行

### 改进生命周期

```
发现缺陷（审计/Phase 6.7 复盘）
  → 记录到 proccess_improvements.yaml
  → 自动修复 / 人工修复
  → 更新 Phase 文档 / SKILL.md / References
  → CHANGELOG 记录版本变更
  → 下次审计验证改进闭环
```

### 改进追踪文件

每次审核和复盘发现的流程缺陷记录在 `.codebuddy/workflow/metrics/proccess_improvements.yaml`：

```yaml
improvements:
  - id: IMP-001
    date: "2026-06-03"
    source: "audit-flow"          # 来源：audit-flow / retrospect / 人工
    issue: "Phase 5.5 包含 archive 硬编码路径"
    type: "hardcode"              # 类型：hardcode / missing-ref / inconsistency / missing-file
    file: "phases/review.md"
    fix: "替换为探路报告占位符"
    status: "fixed"               # 状态：fixed / manual-required / open
```

统计改进历史可以看到：
- 哪些类型的问题最常出现
- 哪些 Phase 文档最容易出问题
- 修复周期
