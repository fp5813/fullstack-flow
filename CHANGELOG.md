# fullstack-flow 优化记录

## [26.0.0] - 2026-06-12

### 基于 Headroom 项目的 8 条优化建议全部实施

受 Headroom（chopratejas/headroom）项目启发，对 fullstack-flow 流程实施 8 条优化建议：

| # | 优化建议 | 说明 |
|:-:|---------|------|
| 1 | **不变性门（Invariant Gates）** | 10 条核心不变性（I1-I10），每条对应可执行的 grep/bash 验证命令 |
| 2 | **决定不做清单（Out-of-Scope Registry）** | 5 类禁止操作声明，Agent 越界时记录并回退 |
| 3 | **影子流量验证（Shadow Traffic）** | API 兼容性对比，新旧接口响应结构差异自动检测 |
| 4 | **本地预检=CI 预检** | env-check 增加编译/TypeScript/Lint/测试预检 |
| 5 | **决策日志已排除方案** | 决策日志模板新增 excluded_options 字段 |
| 6 | **属性测试模板（Property Testing）** | java-test-standards 新增排序确定性/分页一致性属性测试 |
| 7 | **多 Agent 并行审计** | audit 改为 6 并行 Agent 扫描 6 类业务规则模式 |
| 8 | **并行审核** | review.md 已有 FE/BE 并行子代理，已验证无需修改 |

#### 新增文件

- **scripts/invariant-gates.md** (v1.1.0): 10 条核心不变性 + 自动化 grep/bash 验证
- **references/out-of-scope.md** (v1.0.0): 决定不做清单

#### 修改文件

- **SKILL.md** (v25.5.0 → v26.0.0): 快速参考表标注不变性门；流程速览更新；强制执行规则新增第 50 条（不变性门）和第 51 条（决定不做清单）
- **phases/quality-gate.md** (v2.1.0 → v3.0.0): 追加 Step 0.5 不变性门(I1/I2/I9/I10)；35 项检查标注不变性引用
- **phases/coverage-check.md** (v1.2.0 → v2.0.0): 追加 Step 0.5 不变性门(I3/I4)；5 条件检查标注不变性引用
- **phases/review.md** (v2.3.0 → v3.0.0): Step 2.5 拆分为不变性门(I7/I8) + 文档完整性扫描
- **phases/code.md** (v4.0.0): 追加 Step 4.5 不变性门(I5/I6)
- **phases/spec.md** (v2.1.0 → v2.2.0): 引用 out-of-scope.md；自检追加"不在范围项已明确记录"
- **phases/audit.md** (v1.2.0 → v2.0.0): Step 2 改为 6 并行 Agent 扫描 + Step 2a 合并去重
- **scripts/env-check.md** (v1.0.0 → v2.0.0): 新增 Step 0.5.4 CI 预检
- **scripts/create-decision.md** (v1.0.0 → v2.0.0): 决策日志模板新增"已排除方案"章节
- **references/e2e-test-rules.md** (v1.0.0 → v2.0.0): 新增第 6 章影子流量验证
- **references/skills-navigation-index.md** (v1.0.0 → v2.0.0): 新增 invariant-gates 脚本组件
- **references/scripts-ref.md** (v1.0.0 → v2.0.0): 新增 invariant-gates 引用，更新版本号
- **java-test-standards/SKILL.md**: 新增第 6.5 章属性测试模板；第 7 章新增属性测试行；第 8 章审查清单追加

### 工具和 MCP 使用记录 — Phase 出口自动审计

在 Phase 出口协议中追加工具使用和 MCP 调用记录，支持效率分析和上下文压缩决策。

| 新增字段 | 格式 | 记录内容 |
|---------|------|---------|
| `tools_used.{phase}` | `{ mcp_calls: [{tool, count, input_tokens, output_tokens}], key_files_loaded: [] }` | 每个 MCP 工具的调用次数和 token 消耗 |
| `token_audit.{phase}.mcp_tool_calls` | `N` | MCP 工具调用总次数 |

#### 修改文件

- **scripts/phase-entry-exit.md**: 出口流程新增 Step 7（记录工具和 MCP 使用情况）和 Step 8（记录 token 审计）
- **SKILL.md**: Token 审计示例新增 `tools_used` 字段，强制执行规则 39 更新，缺失维度表新增"工具使用审计"行，交付物路径更新

## [25.4.0] - 2026-06-11

### codegraph 自动平衡机制 — 探路时自动检测子项目 MCP 服务并分发

探路前自动检测项目 `.mcp.json` 中注册的 `codegraph-{子项目名}` 服务，按子项目维度分发探路子 Agent。

| 检测结果 | 策略 | Agent A 分发 |
|---------|------|-------------|
| 无子项目服务 | 标准探路 | 使用 `codegraph`（全项目扫描） |
| 1 个子项目服务 | 聚焦探路 | 使用 `codegraph-{子项目}` |
| 2+ 个子项目服务 | 并行分发探路 | 每个子项目一个 Agent A-N |

#### 修改文件

- **phases/probe.md**: 新增 Step 1.8 检测可用 codegraph 服务，Step 2 参数增加 `mcp_codegraph_services`
- **scripts/spawn-probe-agents.md**: v1.0.0 → v2.0.0，Agent A 改为多子项目分发模式
- **code-explore/SKILL.md**: 并行探路流程中 Agent A 改为自动分发

#### 项目配置

- **moma/.mcp.json**: 新建，注册 `codegraph` 服务
- **moma/mcp-servers/codegraph-wrapper.js**: 新建，MCP 服务端启动脚本

## [25.3.0] - 2026-06-11

### 测试方式变更 — Java main() 改为 Python 脚本

将 `java-test-standards` 的测试脚本从 Java `main()` 方法改为 Python 脚本。

| 方面 | 改造前 | 改造后 |
|------|--------|--------|
| 脚本语言 | Java (`HttpClient`) | Python (`requests`) |
| 运行方式 | `mvn exec:java -pl ...` | `python tests/api/test_{entity}.py` |
| 脚本位置 | `src/test/java/com/{project}/api/` | `tests/api/` |
| 环境依赖 | Maven + JDK | Python 3.8+ + `requests` |

#### 修改文件

- **java-test-standards/SKILL.md**: 全面重写 — 测试原则、目录结构、模板代码、TDD 流程、审查清单全部改为 Python
- **phases/code.md**: 6 处 `mvn exec:java` 替换为 `python`，Java 模板替换为 Python 模板

## [25.2.0] - 2026-06-11

### 用户确认机制优化 — 所有修改代码前先与用户确认

在 3 个关键决策节点 + 测试执行节点追加用户确认环节：

| Phase | 新增确认环节 | 确认内容 |
|-------|-------------|---------|
| Phase 1 描述澄清 | Step 4.5 用户确认摘要 | 任务类型/目标模块/影响范围/业务修改点/关键词 |
| Phase 3 规格澄清 | Step 4 用户确认 Scope/AC | 影响范围/验收标准/业务流程修改点 |
| Phase 4 实施计划 | Step 4 用户确认计划 | 任务清单/风险评估/业务流程修改确认 |
| Phase 5 最小修改 | 5.3.5.0 用户确认测试流程 | 测试范围/测试方式/数据准备/验证方式 |

#### 修改文件

- **phases/clarify.md**: 新增 Step 4.5 用户确认澄清摘要，更新自检和约束
- **phases/spec.md**: 新增 Step 4 用户确认 Scope/AC，后续步骤重新编号
- **phases/plan.md**: 新增 Step 4 用户确认实施计划，后续步骤重新编号
- **phases/code.md**: 新增 5.3.5.0 用户确认测试流程，更新自检和约束
- **SKILL.md**: 流程速览表更新，强制执行规则 1/5/6/8 更新

## [25.1.0] - 2026-06-11

### 子技能拆分 — java-dev-standards 拆分出 java-review-standards

从 `java-dev-standards` 中拆分出独立的 `java-review-standards` 子技能，将代码审查清单单独管理。

| 新子技能 | 来源 | 定位 |
|---------|------|------|
| `java-review-standards/` | 原 java-dev-standards 第 2 章 | 审查清单，Agent BE 在 Phase 5.5 加载 |

#### 新增文件

| 文件 | 用途 |
|------|------|
| `java-review-standards/SKILL.md` | 代码审查清单（6 类：后端/安全/API/性能/数据完整性/文档） |
| `java-review-standards/INDEX.md` | 导航索引 |

#### 修改文件

- **java-dev-standards/SKILL.md**: 删除第 2 章（代码审查清单），更新 description、快速参考表和末尾使用说明
- **java-dev-standards/INDEX.md**: 更新用途和快速参考表，移除 Phase 5.5 引用
- **SKILL.md**: skills 列表新增 `java-review-standards`
- **phases/review.md**: Agent BE 前置加载改为 `java-review-standards`
- **references/skills-navigation-index.md**: 组件表新增 java-review-standards 行，门控场景标签新增引用
- **README.md**: 目录结构新增 java-review-standards

## [25.0.0] - 2026-06-11

### 子技能拆分 — java-standards 拆分为 java-dev-standards + java-test-standards

根据用户需求，将原 `change-record/java-standards` 子技能拆分为两个独立的子技能：

| 新子技能 | 来源 | 定位 |
|---------|------|------|
| `java-dev-standards/` | 原第 1 章（Spring Boot 开发）+ 第 3 章（审查清单）+ 第 4 章（自检） | 开发规范，Agent BE 在 Phase 5/5.5 加载 |
| `java-test-standards/` | 原第 2 章（JUnit 测试）**全面重写** | 测试规范，Agent BE-Test 在 Phase 5 TDD 时加载 |

### java-test-standards 关键变更

| 方面 | 旧规范 | 新规范 |
|------|--------|--------|
| 测试方式 | JUnit 5 + Mockito/Testcontainers/MockMvc | `main()` 方法 + `mvn exec:java` |
| 数据准备 | 必须通过 API 接口调用造数 | **优先查询数据库，不足时 API 补充** |
| 验证方式 | JUnit Assertions | 响应内容 + 日志输出 + 数据库查询 |
| 是否改项目代码 | 测试类是项目代码 | 不修改任何项目代码 |

#### 新增文件

| 文件 | 用途 |
|------|------|
| `java-dev-standards/SKILL.md` | Spring Boot 开发规范 + 代码审查清单 |
| `java-dev-standards/INDEX.md` | 导航索引 |
| `java-test-standards/SKILL.md` | main() 测试脚本规范 + 审查清单 |
| `java-test-standards/INDEX.md` | 导航索引 |

#### 删除文件

| 文件 | 说明 |
|------|------|
| `change-record/java-standards/SKILL.md` | 被两个新子技能替代 |
| `change-record/java-standards/` | 空目录 |

#### 修改文件

- **SKILL.md**: v24.3.0 → v25.0.0
  - skills 列表：`change-record/java-standards` → `java-dev-standards` + `java-test-standards`
- **phases/code.md**: 5.3.5 节精简，增加子技能引用，修改数据造数规则表述
- **phases/review.md**: Agent BE 引用改为 `java-dev-standards`
- **references/skills-navigation-index.md**: 组件表 + 场景标签矩阵更新
- **README.md**: 目录结构更新

## [24.3.0] - 2026-06-07

### MCP 分层分离 — 用户级通用指引与项目级配置解耦

根据审核反馈，将 `references/mcp-tools-summary.md` 中混合的用户级通用指引和项目级配置进行分层分离。

#### 新增文件

| 文件 | 用途 |
|------|------|
| `references/mcp-project-templates/TEMPLATE.md` | 新项目 MCP 初始化标准化模板 |
| `references/mcp-project-templates/computility.md` | computility 项目 MCP 配置（移自原 mcp-tools-summary.md） |
| `references/mcp-project-templates/archive.md` | archive 项目 MCP 配置（移自原 mcp-tools-summary.md） |

#### 修改文件

- **references/mcp-tools-summary.md**: v1.0.0 → v2.0.0
  - 删除项目级配置（computility 的 IP:端口、archive 的禁用状态等）
  - 仅保留通用指引：命名模式、使用原则、使用优先级
  - 新增指向 `mcp-project-templates/` 的项目索引表

- **SKILL.md**: v24.2.0 → v24.3.0
  - 交付物路径新增 `references/mcp-project-templates/` 引用

## [24.2.0] - 2026-06-07

### 新增核心设计原则：单一职责

根据审核反馈，新增"单一职责"核心设计原则，完善核心设计哲学体系。

#### 修改文件

- **SKILL.md**: v24.1.0 → v24.2.0
  - 核心设计哲学标题新增"单一职责"
  - 新增"单一职责"专节：3 条规则（Phase/Agent/脚本各自单一职责）
  - 新增"保证模型输出维度"场景对比表（宽职责 vs 窄职责）
  - description 字段更新
  - 缺失维度表新增"单一职责"维度

## [24.1.0] - 2026-06-07

### 核心设计哲学：渐进式披露 + 动态匹配

根据功能审核结果，新增渐进式披露与动态匹配的核心设计哲学文档化。

#### 新增文件

- **references/quick-start.md** (v1.0.0): L1 快速入门指南，新用户的入口点
- **references/skills-navigation-index.md** (v1.0.0): 技能导航索引，定义所有组件的依赖关系、场景标签和动态匹配规则

#### 修改文件

- **SKILL.md**: v24.0.0 → v24.1.0
  - 新增"核心设计哲学：渐进式披露 + 动态匹配"章节
  - 三层加载架构（L1/L2/L3）定义
  - 四阶段动态循环（索引→匹配→补充→重新匹配）文档化
  - frontmatter references 新增 quick-start.md 和 skills-navigation-index.md
  - 交付物路径新增 references 引用

- **proccess_improvements.yaml**: 新增 IMP-007 审计记录

## [24.0.0] - 2026-06-07

### 流程脚本化 — 将固定/重复步骤转化为结构化指令脚本

根据功能审核结果，将 7 类固定重复步骤提取为 6 个标准化脚本，减少 ~385 行重复代码。

#### 新增目录

- **scripts/**: 脚本库目录（6 个结构化指令文档）

#### 新增脚本

| 脚本 | 优先级 | 替代重复 | 替代行数 |
|------|:------:|---------|:--------:|
| `scripts/phase-entry-exit.md` (v1.0.0) | P0 | 13 个 Phase 的入口/出口协议 | ~142 行 |
| `scripts/update-index.md` (v1.0.0) | P0 | 9 个 Phase 的 INDEX 更新逻辑 | ~30 行 |
| `scripts/build-fix.md` (v1.0.0) | P1 | code.md 的 Build-Fix 循环 | ~77 行 |
| `scripts/env-check.md` (v1.0.0) | P1 | clarify.md 的环境预检 | ~46 行 |
| `scripts/spawn-probe-agents.md` (v1.0.0) | P1 | probe.md 的 4 并行 Agent 启动 | ~60 行 |
| `scripts/create-decision.md` (v1.0.0) | P2 | spec/plan 的决策日志创建 | ~4 行 |

#### 修改文件

- **SKILL.md**: v23.0.0 → v24.0.0
  - 规则 48-49: 流程脚本化原则 + 脚本库维护规则
  - 入口/出口协议注解：标记为"可引用 scripts/phase-entry-exit.md"
  - 交付物路径：新增 scripts/ 目录引用

- **references/scripts-ref.md**（新建）: v1.0.0 — 脚本库总览文档

### 设计决策

- 脚本采用 **Markdown 结构化指令**（而非 .sh/.py 可执行文件），因为 CodeBuddy AI 在对话上下文中运行，直接加载脚本指令比执行外部脚本更可靠
- 每个脚本通过 `Read` 工具加载到 AI 上下文，按 `input` 参数替换占位符后执行
- P0 脚本（entry-exit + index）无依赖可立即使用；P1 脚本（build-fix/env/probe）依赖 P0 的入口/出口替换完成后使用

## [23.0.0] - 2026-06-07

### 复盘与优化体系增强 — 定期健康度检查 + 失败案例库 + 汇总复盘

根据功能完整性审核结果，补齐复盘系统的三个缺失维度。

#### 新增文件

- **docs/技能审计/复盘优化功能审核报告-2026-06-07.md**: 功能完整性审核报告
- **docs/失败案例/TEMPLATE.md**: 失败案例标准化模板（YAML frontmatter + 5 Why 根因 + 预防措施）
- **docs/失败案例/FAILURE-INDEX.md**: 失败案例索引，支持按领域/严重度分类检索

#### 修改文件

- **SKILL.md**: v22.8.0 → v23.0.0
  - 规则 44-47: 定期健康度检查/失败案例必建/汇总检查/失败案例归档
  - 命令表新增 `/skill-health-check`、`/failure-cases`
  - 快速参考表 Phase 6.7 产出追加"失败案例"
  - 交付物路径新增 `docs/失败案例/FAILURE-INDEX.md`
  - 缺失维度表新增"失败案例库"和"Skill 健康度"

- **phases/retrospect.md**: v1.1.0 → v2.0.0
  - 快速概览流程追加 Step 2.3 和 Step 5
  - Step 2.3 失败案例结构化：创建判定条件、文件生成步骤、索引更新
  - Step 3 沉淀类型表新增类型 F（失败案例）
  - Step 4.7 失败案例归档
  - Step 5 汇总检查：跨复盘重复模式检测、汇总洞察、改进追踪
  - 输出模板追加"失败案例"和"汇总洞察"章节
  - 完整性自检从 9 项扩展为 13 项
  - 约束新增失败案例不可删除和汇总检查记录规则

- **proccess_improvements.yaml**: 新增 IMP-004、新增 `recurring` 类型

### 设计决策

- 失败案例存放为独立文件（非内嵌在复盘记录中），支持跨复盘检索
- 汇总检查使用"轻量版"设计（每次复盘内置），避免额外启动成本
- 定期健康度检查为独立命令（按需/定时调用），不阻塞正常开发流程

## [22.8.0] - 2026-06-04

### 新增 — Checkpoint 标记 + Java TDD 快捷命令

实现 ECC 对比项全部剩余 P2 项：里程碑标记和语言级 TDD。

#### 新增文件

- **commands/checkpoint.md**: `/checkpoint` — 在当前 Phase 标记里程碑，保存进度快照到 `checkpoints/`
- **commands/java-test.md**: `/java-test` — Java 后端 TDD 快捷命令（Red→Green→Refactor）

#### 修改文件

- **SKILL.md**: v22.7.0 → v22.8.0
  - 命令表追加 `/checkpoint`、`/java-test`
  - Phase 5 核心动作追加 `/java-test` 引用
  - 动态工作流追加 Java TDD 快捷方式示例

### 命令功能

| 命令 | 功能 |
|------|------|
| `/checkpoint` | 标记里程碑，保存 Phase/步骤/制品快照到 `metrics/checkpoints/` |
| `/checkpoint ApiTest 已通过，准备实现 Service` | 带描述标记 |
| `/java-test OrderController` | 按 Controller 名走 TDD 流程 |
| `/java-test 用户管理列表查询` | 按功能描述自动搜索对应 Controller |

## [22.7.0] - 2026-06-04

### 新增 — 测试覆盖率报告 + 质量门控独立命令

新增 `/test-coverage` 和 `/quality-gate` 命令，补全 ECC 对比 P1 全部 2 项。

#### 新增文件

- **commands/test-coverage.md**: `/test-coverage` — 运行测试并报告代码覆盖率（支持 JaCoCo / Vitest / Jest）
- **commands/quality-gate.md**: `/quality-gate` — 独立运行质量门控检查（不依赖 Phase 流程）

#### 修改文件

- **SKILL.md**: v22.6.0 → v22.7.0，命令表追加 2 个

### `/test-coverage` 命令

| 参数 | 功能 |
|------|------|
| 无参数 | 检测技术栈 + 运行覆盖率工具 |
| `--report` | 仅展示最近一次报告 |
| `--html` | 生成 HTML 报告 |
| `--threshold 80` | 指定最低阈值（默认 80%） |

| 技术栈 | 工具 | 阈值 |
|--------|------|:----:|
| Maven/Java | JaCoCo | ≥ 80% |
| Vue/TS (Vitest) | c8 | ≥ 80% |
| React/TS (Jest) | istanbul | ≥ 80% |

### `/quality-gate` 命令

| 参数 | 功能 |
|------|------|
| 无参数 | 运行全部可用门控 |
| `--phase 2.5` | 仅运行指定 Phase 门控 |
| `--list` | 列出所有可用门控 |
| `--fix` | 自动修复低风险文档缺失 |

| 门控 | 检查内容 | 自动修复 |
|:----:|---------|:--------:|
| Gate 1 | 探路报告 L0-L5 完整性 | INDEX.md |
| Gate 2 | AC/文件/术语覆盖 | INDEX.md |
| Gate 2.5 | 测试全部通过 | ❌ |
| Gate 3-5 | 文档/规则一致性 | INDEX.md |

## [22.6.0] - 2026-06-04

### 新增 — 端到端测试（E2E）+ Playwright 集成

新增 `/e2e` 命令和 Phase 5.5 E2E 验证流程，支持从规格文档 AC 自动生成 Playwright 测试脚本。

#### 新增文件

- **commands/e2e.md**: `/e2e` — 从规格 AC 生成 Playwright E2E 测试脚本并运行

#### 修改文件

- **phases/review.md**: v2.1.0 → v2.2.0
  - Step 3 全面升级：新增 Playwright E2E 验证层
  - 新增验证层次表（后端 API / 数据库 / 前端自动 / 人工标注）
  - 新增 `/e2e` 命令引用
  - 快速概览更新为 Playwright E2E
- **SKILL.md**: v22.5.0 → v22.6.0
  - 流程速览 Phase 5.5 核心动作追加 Playwright E2E
  - 命令表追加 `/e2e`

### E2E 验证层次

| 层次 | 方法 | 适用场景 |
|------|------|---------|
| 后端 API | `api-fetcher` / `curl` | 后端接口 |
| 数据库 | `mysql-*.execute_query` | 数据变更 |
| 前端交互（自动） | `npx playwright test` | UI 流程 |
| 前端交互（手动） | 标注 ⚠️ 需人工测试 | 无法自动化的步骤 |

## [22.5.0] - 2026-06-04

### 新增 — 风险预评估 + 会话保存/恢复

Phase 4 实施计划升级风险预评估（5 类风险逐项检查）；新增会话快照机制支持长工作流中断恢复。

#### 修改文件

- **phases/plan.md**: v2.1.0 → v3.0.0
  - Step 3 扩展为"排序 + 风险预评估"
  - 新增 6 类风险检查表（技术/兼容性/数据/依赖/回退/范围蔓延）
  - 高风险判定：追加"风险应对"子章节
  - 输出模板升级风险表格式
  - 自检清单追加 2 项风险检查
- **commands/save-session.md**: 新增 — 保存会话快照
- **commands/resume-session.md**: 新增 — 从快照恢复
- **SKILL.md**: v22.4.0 → v22.5.0，命令表追加 `/save-session` `/resume-session`

### 风险预评估表

| 风险类别 | 检查项 | 应对策略 |
|---------|--------|---------|
| 技术风险 | 不熟悉的技术栈/库 | 追加 Spike 任务 |
| 兼容性风险 | 影响现有 API/表结构 | 版本路由/字段兼容 |
| 数据风险 | 表结构变更/数据迁移 | 迁移脚本 + 回滚脚本 |
| 依赖风险 | 外部接口/未合并 PR 未就绪 | Mock/标记阻塞 |
| 回退复杂度 | 涉及文件数/API 变更数 | 步骤化回退方案 |
| 范围蔓延 | 超出 Scope | 单独开新任务 |

### 会话快照

```
/save-session     → 保存快照（含 phase/artifacts/progress/用户输入）
/resume-session   → 列出快照 / 恢复指定快照
/resume-session --latest → 恢复最近一次
```

## [22.4.0] - 2026-06-04

### 新增 — `/context-budget` Token 消耗分析命令

查看各 Phase Token 消耗占比、当前上下文预算状态、已加载文件清单，辅助压缩决策。

#### 新增文件

- **commands/context-budget.md**: `/context-budget` 命令

#### 修改文件

- **SKILL.md**: v22.3.0 → v22.4.0，命令表追加 `/context-budget`

### 命令参数

| 参数 | 功能 |
|------|------|
| 无参数 | 显示 Token 预算报告 + 压缩建议 |
| `--compact` | 按建议执行压缩 |
| `--history` | 显示历史 Phase 消耗趋势 |
| `--detail` | 显示已加载文件清单 |

### 输出示例

```
 Phase          输入      输出      小计      占比
 ───────────── ─────── ─────── ──────── ──────
 clarify        3,200    1,500    4,700    3.7%
 probe         12,800    6,200   19,000   15.1%
 code          35,000   18,000   53,000   42.2%  ← 最高消耗
 ───────────── ─────── ─────── ──────── ──────
 总计          79,700   46,700  125,600    100%

 状态: 📋 考虑准备压缩 (30%~50%)
```

## [22.3.0] - 2026-06-04

### 新增 — Phase 5 Build-Fix 自动修复

编译/测试失败时自动检测错误类型并定向修复，最多 3 轮循环。

#### 修改文件

- **phases/code.md**: v3.1.0 → v4.0.0
  - 新增 5.4.5 Build-Fix 自动修复
  - 5.4.5.1 错误分类检测（6 种类型）
  - 5.4.5.2 自动修复策略（逐类型匹配）
  - 5.4.5.3 修复循环控制（最多 3 轮）
  - 5.4.5.4 修复验证命令
  - 5.4.5.5 修复记录格式
  - 自检清单追加 build-fix 检查项

### Build-Fix 流程

```
编译/测试失败 → 错误分类
  ├─ 语法错误 → 定位行号修正
  ├─ 类型不匹配 → 修正类型声明
  ├─ 缺失导入 → 追加 import
  ├─ 方法签名 → 检查 Interface vs Impl
  ├─ 测试断言 → 判断断言/实现谁错
  └─ TS 错误 → 修正类型/守卫
  ↓
重新编译/测试
  ├─ 通过 → 继续
  └─ 失败 → 最多 3 轮 → 仍失败则标记"需人工处理"

## [22.2.0] - 2026-06-04

### 新增 — `/aside` 侧问命令

Phase 执行过程中可临时回答侧问题，不丢失当前 Phase 上下文。

#### 新增文件

- **commands/aside.md**: `/aside` — 保存 Phase 状态 → 回答侧问题 → 恢复 Phase

#### 修改文件

- **SKILL.md**: v22.1.0 → v22.2.0
  - 新增命令表追加 `/aside` 和 `/workflow-pattern`

### 工作流程

```
Phase 1/3 Q&A 中 → 用户输入 /aside 侧问题
  ↓
Step 1: 读取 state.yaml，保存 Phase 上下文到内存
Step 2: 确认挂起，输出 /aside 确认信息
Step 3: 回答侧问题（只读，≤3 轮）
Step 4: 恢复上下文，继续原 Phase
```

### 约束

- 不修改 state.yaml 和任何项目文件
- 不超过 3 轮交互
- 不在 Phase 5 代码修改中使用
- 侧问题内容不保留到 Memory/Instinct

## [22.1.0] - 2026-06-04

### 新增 — Token 审计与上下文优化

每次 Phase 出口自动审计 input/output token 用量，记录到 `metrics_snapshot.token_audit`，减少上下文重复加载。

#### 修改文件

- **SKILL.md**: v22.0.0 → v22.1.0
  - 新增"Token 审计与上下文优化"章节
  - 缺失维度表新增"Token 审计"行
  - 新增强制规则 39（Phase 出口 token 审计）
  - 新增强制规则 40（子 Agent 摘要 ≤1K）
  - 交付物路径追加 token-audit.yaml

### Token 审计字段

```yaml
metrics_snapshot:
  token_audit:
    clarify: { input_tokens: 3200, output_tokens: 1500 }
    probe:   { input_tokens: 8500, output_tokens: 4200 }
  token_accumulated:
    total_tokens: 125600
    phases_count: 6
```

## [22.0.0] - 2026-06-04

### 重大变更 — Phase 5 强制 TDD 流程

TDD（Red→Green→Refactor）从可选升级为强制执行规则，覆盖后端 Java 和前端 Vue。

#### 修改文件

- **SKILL.md**: v21.2.0 → v22.0.0
  - 流程速览 Phase 5 核心动作追加"TDD 先行"
  - 规则 8 升级为"必须执行 TDD 流程"，新增 Gate 2.5（测试全部通过）
  - 规则 36 更新为 TDD 模式（Red→Green，API 造数）
- **code.md**: v3.1.0
  - 5.3.5 TDD 流程已是强制规则（之前已实现）

### TDD 强制执行链条

```
Phase 4.5 覆盖验证通过
    ↓
Phase 5 入口: TDD 强制
    ├── 后端: 先写 ApiTest (Red) → 运行确认失败 → 实现代码 (Green)
    ├── 前端: 先写 spec (Red) → 运行确认失败 → 实现组件 (Green)
    └── Gate 2.5: 测试全部通过
    ↓
Phase 5.6: API 文档生成

## [21.2.0] - 2026-06-04

### 重大变更 — Phase 5 全面采用 TDD 原则（Red → Green → Refactor）

测试优先于实现代码，API 测试定义契约在先，实现使测试通过在后。

#### 修改文件

- **phases/code.md**: v3.0.0 → v3.1.0
  - 5.3.5 全面重写为 **TDD 三阶段流程**
  - 5.3.5.1 识别影响范围（前置）
  - 5.3.5.2 Red: 先写 API 测试并运行确认失败
  - 5.3.5.3 Green: 实现代码使测试通过
  - 5.3.5.4 Refactor: 重构优化（可选）
  - 5.3.5.5 并行执行与 TDD 集成
  - 5.3.6 并行执行（BE Agent 在 TDD 测试就绪基础上实现）
  - 5.3.7 TDD 测试回归（Green 最终验证）

## [21.1.0] - 2026-06-04

### 改进 — Phase 5 测试类创建规则升级为 API 集成测试规范

明确 Java 项目统一走 API 接口测试，不创建单元测试类。测试数据全部通过 API 调用造数，禁止直接数据库 INSERT。

#### 修改文件

- **phases/code.md**: v2.1.0 → v3.0.0
  - 5.3.5 全面重写为"创建 API 级集成测试"
  - 新增 5.3.5.1 识别影响范围（codegraph/Grep 定位 Controller）
  - 新增 5.3.5.2 创建测试类规范（命名/运行方式/启动命令）
  - 新增 5.3.5.3 测试数据造数规则（API 链调用造数，禁止 SQL INSERT）
  - 新增 5.3.5.4 测试范围（正常/异常/业务规则/边界值）
  - 新增 5.3.5.5 已存在测试类处理
  - 新增 5.3.5.6 约束（统一 API 测试，无单元测试）

## [21.0.0] - 2026-06-04

### 重大新增 — 动态工作流（Dynamic Workflows）

根据任务类型即时编排定制化子 Agent 执行框架，支持 4 种并行协作模式。

#### 新增文件

- **references/dynamic-workflows-reference.md**: v1.0.0
  - 4 种工作流模式定义
  - 模式选择逻辑（按 task.type 自动匹配）
  - 子 Agent 编排协议（输入/输出规范 + 汇总合并规则）
- **commands/workflow-pattern.md**: `/workflow-pattern` — 查看和切换工作流模式

#### 修改文件

- **SKILL.md**: v20.0.0 → v21.0.0
  - description 追加"→动态工作流"
  - 新增"动态工作流（Dynamic Workflows）"章节
  - references 列表追加 dynamic-workflows-reference.md
- **phases/clarify.md**: v2.0.0 → v2.1.0
  - 新增 Step 3: 选择工作流模式（按 task.type 自动推荐）
  - 澄清摘要模板追加"任务类型"和"推荐工作流模式"
  - Step 5 写入 workflow.pattern
- **phases/probe.md**: v2.1.0 → v2.2.0
  - 新增 Step 1.5: 加载工作流模式（按模式选择探路策略）
- **phases/code.md**: v2.1.0 → v2.2.0
  - 新增 5.2: 工作流模式选择（执行策略区别）
- **phases/review.md**: v2.1.0 → v2.2.0
  - Step 0 读取 workflow.pattern → adversarial 时进入对抗性审核模式

### 4 种工作流模式

| 模式 | 探路策略 | 代码策略 | 审核策略 |
|------|---------|---------|---------|
| classify | 定向探路（2~3 Agent） | 单 Agent 顺序 | 标准 |
| distribute | 全覆盖探路（4 Agent） | FE/BE 并行 | 标准 |
| adversarial | 全覆盖+安全扫描（5 Agent） | 实现+审核循环 | 独立 Agent 审查 |
| generate | 多角度收集（3~4 Agent） | 多方案对比 | 方案对比审核 |

## [20.0.0] - 2026-06-04

### 改进 — Phase 1 新增环境预检（Step 0.5）

在描述澄清之前增加轻量级环境检查，确保项目工具链和 MCP 服务就绪，避免探路阶段因环境问题中断。

#### 修改文件

- **phases/clarify.md**: v1.1.0 → v2.0.0
  - 新增 Step 0.5: 环境预检
    - 0.5.1 项目工具链检查（node/npm/mvn/git）
    - 0.5.2 MCP 服务就绪检查（codegraph/mysql-{子项目}/api-fetcher）
    - 0.5.3 输出环境摘要表
    - 0.5.4 检查流程（不阻塞，仅标注 ⚠️）
  - 自检清单新增“环境预检已执行”项
- **SKILL.md**: v19.0.0 → v20.0.0
  - 流程速览 Phase 1 核心动作追加"环境预检 →"
  - 强制执行规则第 1 条更新为"环境预检 + 描述澄清"

### 检查内容

| 检查项 | 方法 | 通过条件 | 阻塞 |
|--------|------|---------|:----:|
| node | `node --version` | 正常输出版本号 | ❌ |
| npm | `npm --version` | 正常输出版本号 | ❌ |
| mvn | `mvn --version` | 正常输出版本号 | ❌ |
| git | `git --version` | 正常输出版本号 | ❌ |
| codegraph | `~/.CODEBUDDY/mcp.json` 配置 | 已定义 | ❌ |
| mysql-{子项目} | `settings.local.json` 启用列表 | 已启用 | ❌ |
| api-fetcher | `~/.CODEBUDDY/mcp.json` 配置 | 已定义 | ❌ |

所有检查不阻塞流程，缺失项标注 ⚠️ 警告。

## [19.0.0] - 2026-06-04

### 重大新增 — Instinct（直觉）机制：自学习闭环

新增持续学习系统：每次 fullstack-flow 工作流执行完毕后，自动从实施过程中提取可复用模式，生成 Instinct 文件，并支持进化（Evolve）为用户级技能/命令。

#### 新增文件

- **references/instinct-reference.md**: v1.0.0 — Instinct 参考文档
  - 定义、格式、存储结构
  - 生命周期：模式提取 → Instinct YAML → 提升全局 → 进化技能
  - 领域分类（代码/数据/测试/安全/UX/流程/领域）
  - 置信度规则、promotion 规则
- **commands/instinct-status.md**: `/instinct-status` — 查看所有 Instinct（项目+全局）
- **commands/instinct-evolve.md**: `/instinct-evolve` — 聚类 instincts 生成技能/命令
- **commands/instinct-promote.md**: `/instinct-promote` — 项目级→全局级提升

#### 修改文件

- **SKILL.md**: v18.1.0 → v19.0.0
  - description 追加"→本能"
  - 新增强制执行规则 37-41（Instinct 提取/存放/置信度/提升）
  - 缺失维度表新增"知识沉淀"行
  - 新增 "Instinct（直觉）机制" 章节
  - 交付物路径追加 `.codebuddy/instincts/INDEX.md`
  - references 列表追加 `instinct-reference.md`
- **phases/retrospect.md**: v1.0.0 → v1.1.0
  - 新增 Step 2.5: Instinct 提取（模式识别 → 写入文件 → 更新索引 → 全局提升检查）
  - 自检清单新增 instinct 检查项
- **phases/rule-sync.md**: v3.1.0 → v3.2.0
  - 新增 Step 3.6: Instinct 捕获（业务规则类型 → instinct 映射表）

#### 新增目录

- `~/.CODEBUDDY/instincts/personal/` — 全局 instinct 存储
- `~/.CODEBUDDY/instincts/inherited/` — 导入 instinct
- `~/.CODEBUDDY/instincts/INDEX.md` — 索引

## [18.1.0] - 2026-06-04

### 新增 — Phase 5 强制创建/更新后端接口测试类

新增流程规则：每次开发任务涉及后端 Controller 修改时，Phase 5 必须创建或更新对应的 API 级集成测试类。Phase 5.5 审核时检查测试覆盖完整性。

#### 修改文件

- **SKILL.md**: v18.0.0 → v18.1.0
  - 新增强制执行规则第 36 条（Phase 5 涉及后端 Controller 时必建对应测试类）
  - 可测试性维度覆盖更新（Phase 5 代码 + Phase 5.5 审核）
- **phases/code.md**: v2.1.0 → v2.2.0
  - 新增 Step 5.3.5：创建/更新对应测试类（涉及后端 Controller 时必做）
  - 包含测试内容覆盖（正常/异常/特殊链路/边界值）
  - 已存在测试类则追加而非覆盖
- **phases/review.md**: v2.1.0 → v2.2.0
  - BE Agent 审核表新增"测试覆盖"行：检查测试类存在性及覆盖完整性

### 重大变更 — 新增 Phase 5.6 API 接口文档生成

代码修改完成后主动生成 API 接口文档，一个接口对应一个文档文件。变更后的完整流程：Phase 5 → **Phase 5.6** → Phase 5.5。

#### 新增文件

- **phases/api-doc.md**: v1.0.0 — API 接口文档生成阶段
  - Step 1-2: 确定待生成接口范围 + 收集接口信息（路径/参数/返回类型/DTO 字段/表结构/业务规则）
  - Step 3: 按标准模板生成接口文档，一个接口一个 `.md` 文件
  - Step 4: 自检文档准确性（路径/参数/字段/规则/示例）
  - Step 5: 同步 `docs/接口文档/INDEX.md` 和 `00-全API索引.md`

#### 修改文件

- **SKILL.md**: v17.0.0 → v18.0.0
  - skills 列表新增 `fullstack-flow/phases/api-doc`
  - 主流程序列更新为 `→ 5 → 5.6 → 5.5 →`
  - 流程速览表新增 Phase 5.6 行
  - 新增强制执行规则第 9 条（Phase 5.6 必须生成 API 文档）
  - 后续规则编号递增（原 9→34 变为 10→35）
  - Phase 转换速查表插入 api-doc 转换行
  - 缺失维度表新增 "API 文档" 维度行
- **phases/review.md**: 描述更新标注"文档生成已由 Phase 5.6 承担"；Step 2 和 Step 2.5 的缺失文档处理改为"回退 Phase 5.6"
- **phases/code.md**: Phase 出口指向 `api-doc` 而非 `review`

#### 新增文档

- `docs/接口文档/01-内部用户-批量查询用户.md`: 批量查询用户接口详情
- `docs/接口文档/02-内部系统-发送站内信.md`: 发送站内信接口详情

## [17.0.0] - 2026-06-03

### 重大变更 — 技能泛化：archive-dev → fullstack-flow

技能从档案系统专用进化为个人通用全栈开发技能。名称、描述、标签统一重构，领域示例泛化。

#### 修改文件

- **SKILL.md**: v16.12.0 → v17.0.0
  - `name`: archive-dev → fullstack-flow
  - `description`: 去除"档案系统"，改为通用"全栈开发流程"
  - `tags`: archive → fullstack
  - 标题、skills 列表、正文引用同步更新
- **phases/*.md** (12 个): frontmatter 的 name 前缀和 tags 更新
- **references/**: spec-driven-development.md 和 codegraph-reference.md 中的领域示例泛化
- **关联技能**: archive-change-record / archive-java-standards / archive-code-explore 描述去除"档案"前缀
- **workflow/state.yaml**: 注释更新
- **docs/INDEX.md**: 维护原则引用更新
- **memory/data_source_principles.md**: 引用更新

## [16.12.0] - 2026-06-03

### 改进 — 全流程门控收紧为二值判定（100% 通过率）

所有质量门控移除"警告后继续"灰色地带，改为严格的二值判定。Phase 2.5/4.5/5.5/6.6 四个 Gate 同步收紧，确保每个阶段交付物零缺陷方可推进。

#### 门控变更

| Gate | Phase | 变更前 | 变更后 |
|------|-------|--------|--------|
| Gate 1 | P2.5 | ≥90% 通过率（70-89% 警告后继续） | 100% 通过率（<100% 回退 Phase 2） |
| Gate 2 | P4.5 | AC 100% + 文件 100%（孤儿/术语/数据追溯可警告后继续） | 5 条件 AND 全部满足（任一项不满足回退 Phase 4） |
| Gate 3 | P5.5 | 文档完整性扫描 `⚠️ 部分缺失已补全` 为警告 | `✅ 全覆盖（X 项已补全）` 视为成功 |
| Gate 5 | P6.6 | ≥3 类业务规则模式 | 6 类全覆盖 |

#### 修改文件

- **SKILL.md**: v16.11.0 → v16.12.0
  - Gate 1 描述: `≥90%` → `100%`
  - Gate 2 描述: `AC 100%` → `通过率 100%`，核心动作增加"数据追溯完整性"
  - Gate 3 描述: `一致性自检` → `一致性自检 + 文档全覆盖`
  - Gate 5 描述: `≥3 类模式` → `6 类全覆盖`
  - Rule 4: `≥90%` → `100%`
  - Rule 12: `≥3 类` → `6 类全覆盖`
  - 门控阈值统一说明更新
- **quality-gate.md**: 门控判定三档→二档，描述 `≥90%`→`100%`，出口协议 `<70%`→`<100%`
- **coverage-check.md**: v1.1.0→v1.2.0，门控判定去除 ⚠️警告后继续，改为 5 条件 AND；孤儿/追溯改为 ❌回退；输出模板增加术语一致性和不在范围违规统计行
- **review.md**: v2.0.0→v2.1.0，文档完整性扫描 `⚠️ 部分缺失已补全`→`✅ 全覆盖（X项已补全）`
- **audit.md**: v1.1.0→v1.2.0，描述和自检 `≥3 类`→`6 类全覆盖`
- **spec.md**: 前置验证去除 `warning` 状态
- **code.md**: 前置验证去除 `warning` 状态
- **record.md**: 前置验证去除 `warning` 状态，Q3 复盘四问 `降级通过？`→`门控失败重试？`
- **retrospect.md**: 素材收集和复盘类型表去除"降级通过"引用
- **state.schema.yaml**: v1→v2，R6 门控状态去除 `warning`，新增 R9/R10/R11 门控 100% 验证规则

## [16.9.0] - 2026-05-29

### 改进 — 数据源获取原则升级：API 接口采样 + VO/DTO 数据结构视图 + 强化门控

数据采样方式从直接 SQL 升级为通过 Controller API 接口获取，通过 VO/DTO 赋值对象体现业务数据结构。新增数据覆盖检查项和追溯验证。

#### 修改文件

- **probe.md**: Agent B2 拆为 B2-1（API 接口数据获取，通过 VO/DTO 体现业务结构）和 B2-2（DB 真实数据采样，对照 VO 验证映射）；B3 链路标注扩展包含 VO/DTO 赋值步；探路报告新增 L4.5 VO/DTO 数据视图；显式声明 execute_query 仅 SELECT；边界条件上限 2 个场景
- **quality-gate.md**: C31 增强 LIMIT 3 覆盖有效性检查；C33 扩展包含数据转换逻辑；新增 C34（VO/DTO 结构检查）、C35（表↔VO 字段映射检查）；总项数 33→35
- **spec.md**: 数据来源章节重构为"API 接口数据(VO/DTO)"+"表结构"+"表↔VO 字段映射关系"三部分；自检增加数据章节完整性检查
- **coverage-check.md**: 新增第 7 维度"数据来源追溯"，验证规格文档与探路报告之间的数据一致性
- **SKILL.md**: 更新强制执行规则第 3 条，明确 API 接口采样 + VO/DTO 对照

## [16.8.0] - 2026-05-29

### 新增 — 数据来源分析 + 测试数据采样集成到探路流程

在 Phase 2 探路中增加 B2 真实数据采样和 B3 数据流转追踪，Phase 2.5 新增 C31-C33 数据覆盖门控，Phase 3 规格模板增加数据来源章节。

#### 修改文件

- **probe.md**: 扩展 Agent B 为三层（表结构 + 真实数据采样 + 流转追踪），测试数据样例从推荐改为必填
- **quality-gate.md**: 新增 C31（数据行采样）、C32（枚举值域）、C33（写入链路）三项门控
- **spec.md**: 规格模板新增"数据来源"章节（输入数据表/转换逻辑/测试数据）
- **SKILL.md**: 新增强制规则第 3 条（涉及 DB 时必做数据采样），更新规则编号

## [16.7.0] - 2026-05-29

### 新增 — "实践-复盘-沉淀"闭环系统

设计"实践-复盘-沉淀"三层架构，集成到 fullstack-flow 流程中，每次开发完成后分析根因并驱动技能改进。

#### 架构

```
实践层（Phase 1→6.6 现有）→ 复盘层（Phase 6.7 + 衔接标记）→ 沉淀层（规则/检查项/Memory/CHANGELOG）
```

#### 新增文件

- **retrospect.md**: v1.0.0 — 复盘回顾技能文件
  - 混合模式：轻量四问（每次 Phase 6 判断是否需复盘）+ 完整复盘（发现问题时执行）
  - 6 步流程：入口判定 → 收集素材 → 五维分析（流程/代码/惯例/技能/根因）→ 生成沉淀建议 → 执行沉淀 → 输出文档
  - BUG 复盘专用 5 Why 分析法
  - 强制触发条件：BUG 复盘/门控连环失败/审核重大缺陷/未归档规则≥5/用户要求
- **docs/复盘记录/TEMPLATE.md**: 复盘文档模板
- **docs/复盘记录/INDEX.md**: 复盘记录索引

#### 修改文件

- **SKILL.md**: v16.2.0 → v16.7.0
  - 主流程新增 Phase 6.7
  - 新增强制执行规则 20-22（复盘四问/Phase 6.7 执行条件/复盘发现必须沉淀）
  - 交付物路径增加 `docs/复盘记录/`
- **record.md**: v1.1.0 → v1.2.0
  - 新增 Step 6: 复盘触发检查（复盘四问 + 强制触发判定）
  - 自检清单增加复盘检查项
- **review.md**: 审核结果标记"⚠️ 需复盘"，供 Phase 6.7 入口判定
- **quality-gate.md**: 门控输出增加复盘标记字段
- **audit.md**: 审计报告纳入复盘输入，未归档≥5条强制触发复盘
- **docs/INDEX.md**: 追加复盘记录目录行

#### 背景

- 已有复盘成分分散在 CHANGELOG/质量门控/审计中，但无根因记录、无闭环、无沉淀管线
- 每次修改后只记录"改了什么"，不分析"为什么出错"和"如何避免"
- 质量门控失败只退回不记根因，审计报告与技能改进脱节
- 新系统确保每次实践都能驱动技能进化，形成持续改进的飞轮

### 验证 — 全流程走通确认

- 基于"编研归档新增组卷"修改记录，完整走通闭环流程：
  - Phase 6 Step 6 复盘四问 → Q2 触发 → **进入 Phase 6.7**
  - Phase 6.7 收集素材 → 五维分析（确认流程无缺陷，发现项目惯例和技能可用性两个改进维度）
  - 生成 3 项沉淀建议 → 执行沉淀（Memory/feedback + 复盘文档 + INDEX.md）
  - S01 已实施（项目惯例已记录到 Memory）；S02/S03 标记为"建议"待后续优化
- **闭环验证结果**: ✅ 全流程贯通，复盘四问判定正确，五维分析有效，沉淀管线完整

---

> 历史版本（v16.6.0 ~ v1.0.0）已归档，详见 `git log` 或历史对话记录。
> 归档包含：v16.6.0 Phase 5.5 并行审核、v16.5.0 IDE 编译验证、v16.4.0 复盘驱动优化、v16.3.0 流程闭环审计修复、v16.2.0 大规模精简、v16.1.0 API 接口文档审核、v10.1.0 业务规则同步、v10.0.0 Speckit 理念融合、v9.1.0-cb Phase 3/4 技能实现、v6.1.0-cb CodeBuddy 专用版、v4.0.0 TDD 工作流、v3.x 子技能拆分、v2.0.0 初始规范、v1.0.0 初始版本。
