---
name: fullstack-flow/phases/review
description: "文档代码审核：FE/BE 并行子代理（lite+reasoning 不同模型）核对探路报告/规格/代码质量，主流程做 API 文档一致性比对。"
version: 3.0.0
tags: [fullstack, codebuddy-only, review, documentation, sub-agents]
role: codebuddy-reviewer
model: deepseek-v4-flash
tools: [Read, Write, Edit, Grep, Agent]
references:
  - ../references/codegraph-reference.md
---

# Phase 5.5: 文档代码审核（并行子代理）

> 代码改完了不等于做完了。探路报告/规格文档/API 文档可能已与最终代码不一致。
> API 文档生成已在 Phase 5.6 完成，本阶段仅做一致性核对。

## 快速概览

```
Step 0 读取状态 + 模式检测 → Step 1 FE 审核 → Step 2 BE 审核
  → Step 2 API 接口文档核对 → Step 2.2 响应结构校验（新增）
  → Step 2.5 文档完整性扫描（含 I7/I8 不变性门自动验证）
  → Step 3 E2E 验证 → Gate 3 判定 → Phase 出口 → record
```

**核心**: FE/BE 并行审核 + 响应结构校验 + 不变性门自动验证 + 文档完整性扫描 + Playwright E2E 验证

## 职责

**输入**：Phase 5 代码 + 探路报告 + 规格文档 + API 接口文档  
**输出**：即时审核结果（Phase 6 生成修改记录时将审核结论纳入记录）  
**模式**：只读（发现不一致时更新文档，不修改代码）

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="review", next_phase="record"（通过）/ "code"（代码问题）, is_gate_phase=true
- 额外：验证前置制品 `artifacts.code_changes` 非空
- 工作流模式检测：Read `workflow.pattern`：
  - 如果为 `adversarial` → 进入**对抗性审核模式**（见 Step 2.5）
  - 其他模式 → 标准审核流程

### Step 1: 并行子代理审核

一次性发送 2 个 Agent 调用（`run_in_background: true`），无依赖关系，并行执行。

**Agent FE (model: `lite`)** — 快速前端审核，专注语法/组件/枚举/参数一致性。前置加载 `vue-standards` 子技能。

| 审核项 | 方法 | 通过条件 | 文件范围 |
|--------|------|---------|---------|
| 前端文件存在性 | `ls` 确认 | 探路报告引用的 `.vue`/`.ts` 文件存在 | 探路报告记录的前端路径 |
| 前端方法/组件名 | Grep 搜索 | 组件名、API 方法名与代码一致 | 前端文件 |
| 前端行号准确 | Read 确认 | 与实际相差 ≤3 行 | 前端文件 |
| 前端 AC 完成度 | Grep 搜索实现 | AC 有对应逻辑 | 前端文件 |
| 前端 Scope 边界 | Read 确认修改范围 | 未超出声明范围 | 前端文件 |
| API 参数一致性 | 对比 API 文件与后端接口 | 参数名/类型一致 | 前端 API 文件 |
| 枚举/字典引用 | Grep 检查字典 code | 枚举值在枚举定义中存在 | 前端枚举文件 |
| 组件调用链 | Read 确认组件引用 | 组件引用关系一致 | Vue 组件 |
| Composable 合理性 | Read 确认 composable 接口 | API 错误处理完整；输入输出类型正确 | Composable 文件 |
| 模板安全性 | Grep 检查 `v-html` / `v-if`+`v-for` | 无 `v-if`+`v-for` 混用；`v-html` 内容已转义 | Vue 组件 |

不一致 → 更新探路报告中的路径/行号。AC 未完全实现 → 标注"部分实现"。Scope 超出 → 追加 Extra。

代码质量问题 ≥1 个 或 不一致 ≥2 个时，在输出表格"说明"列标注 **⚠️ 需复盘**。

**Agent BE (model: `reasoning`)** — 深度后端审核，专注调用链/事务/权限/异常/测试覆盖

| 审核项 | 方法 | 通过条件 | 文件范围 |
|--------|------|---------|---------|
| 后端文件存在性 | `ls` 确认 | 引用的 `.java` 文件存在 | 探路报告记录的后端路径 |
| 后端方法名 | Grep 搜索 | 方法签名与代码一致 | Java 文件 |
| 后端行号准确 | Read 确认 | ≤3 行偏差 | Java 文件 |
| 调用链有效 | Read 确认 | Controller→Service→Mapper 贯通 | Java 文件 |
| 后端 AC 完成度 | Grep 搜索实现 | AC 有对应逻辑 | Java 文件 |
| 后端 Scope 边界 | Read 确认修改范围 | 未超出声明范围 | Java 文件 |
| 事务边界 | Grep `@Transactional` | 多表操作有事务注解 | Service 类 |
| 权限注解 | Grep `@RequiresPermissions` | 新接口有权限控制 | Controller 类 |
| 异常处理 | Read 确认 | 异常捕获/抛出不遗漏 | Service/Controller |
| 安全审计 | Grep `@RequiresPermissions` + Read SQL 拼接 | 新接口有权限控制；SQL 使用参数化查询而非拼接 | Controller/Mapper |
| 数据完整性 | Read 确认数据写入模式 + 唯一索引 | 数据写入采用 upsert/先查后写，避免 delete→re-insert；涉及分组统计的表有联合唯一索引作为 DB 层保护（参考 FC-001） | Service + Mapper XML |
| 幂等性 | Read 确认同一请求多次执行的结果 | 幂等性有 DB 层约束（唯一索引），不依赖异常控制正常流程（参考 FC-004） | Service |
| 清理逻辑 | Read 确认清理方法的查询条件 | 清理/删除方法无过度过滤条件（避免遗漏 is_deleted 等逻辑删除记录，参考 FC-003） | Mapper/Service |
| 注释一致性 | Read 确认注释声明的行为与实际代码 | 注释声明的预期行为有对应代码实现（参考 FC-002） | 全部 Java 文件 |
| 数据管道 | Read 确认数据转换/赋值处的 null 保护 | 从数据源读取的字段有默认值兜底，不假设字段非空（参考 FC-006） | Service/Entity |
| 性能风险 | Read 确认循环/批量操作 | 无 N+1 查询；批量操作使用 batch API | Service/Mapper |
| 可观察性 | Grep `@AsyncLog` + Read 异常日志 | 新增接口有操作日志；异常有完整上下文信息 | Controller |
| **测试覆盖** | `ls` 检查 `src/test/java/` 下是否存在对应 Controller 的测试类 | 涉及修改的每个 Controller 都有对应测试类；测试覆盖新增接口的正常和异常流程 | Controller + Test |

不一致 → 更新探路报告。AC 未完全实现 → 标注"部分实现"。代码质量问题→标注"⚠️ 建议"但不修改。

代码质量问题 ≥1 个 或 不一致 ≥2 个时，在输出表格"说明"列标注 **⚠️ 需复盘**。

### Step 2: 主流程执行 API 接口文档核对

等待两个子代理完成后，主流程执行全量 API 接口文档比对：

| 核对项 | 方法 | 通过条件 |
|--------|------|---------|
| 接口路径 | 对比 `@RequestMapping` 与文档 | 完全一致 |
| 请求方法 | 对比 `@GetMapping`/`@PostMapping` 等 | 一致 |
| 请求参数 | 对比 `@RequestParam`/`@RequestBody` | 参数名类型一致 |
| 响应格式 | 对比 `Result<T>` 返回 | 字段结构一致 |
| 权限注解 | 对比 `@RequiresPermissions` | 表达式一致 |

不一致 → 更新接口文档。缺失 → 标记"缺失"，本阶段不生成新文档，回退至 Phase 5.6 补充。同时更新 `docs/接口文档/00-全API索引.md`。
涉及新增模块时，同步更新 `docs/接口文档/INDEX.md` 在表顶部插入新行。

### Step 2.2: 响应结构校验（Response Structure Validation）

> 对比修改前后 API 响应的 JSON 结构，确保向后兼容。参考 Headroom 项目的"字节忠实门"理念。
> 需要探路报告中保存的旧接口响应样本（Phase 2 探路时采集）。

#### 校验项

| 校验项 | 方法 | 通过条件 |
|--------|------|---------|
| **字段一致性** | 对比修改前后 JSON 响应字段集合 | 无字段删除（新增字段 ✅） |
| **类型一致性** | 对比同名响应字段的 JSON 类型（string/number/boolean/array/object） | 类型一致 |
| **数据结构** | 对比嵌套结构的层级深度和数组元素类型 | 结构一致 |
| **空值行为** | 对比 null/空数组/空对象的序列化行为 | 行为一致 |

#### 差异判定规则

| 差异类型 | 判定 | 处理方式 |
|---------|:----:|---------|
| 新增字段（旧无新有） | ✅ 向后兼容 | 无需处理 |
| 删除字段（旧有新无） | ❌ 破坏兼容 | 标注需确认，如非有意则为 BUG，回退 Phase 5 |
| 类型变化（同名字段类型不同） | ❌ 破坏兼容 | 标注需确认，回退 Phase 5 |
| 值变化（同参数返回值不同） | ⚠️ 需人工确认 | 需确认是否为预期行为 |
| 排序顺序变化 | ⚠️ 需人工确认 | 需确认是否影响前端展示 |

#### 校验流程

1. **定位旧接口样本**：从探路报告的 `## API 响应样本` 章节读取旧接口的响应样本文件路径
2. **读取样本**：`Read docs/探路报告/samples/{文件名}.json`
3. **调用新接口**：使用 `api-fetcher` 或 `curl` 用相同参数调用新接口
4. **结构对比**：对比新旧响应的 JSON 字段集合、类型、嵌套结构
5. **输出结果**：

```markdown
#### 响应结构校验结果
| 接口 | 字段一致性 | 类型一致性 | 数据结构 | 空值行为 | 整体状态 |
|------|:---------:|:---------:|:--------:|:--------:|:--------:|
| `GET /api/xxx/list` | ✅ 无删除 | ✅ 一致 | ✅ 一致 | ✅ 一致 | ✅ 通过 |
| `GET /api/xxx/detail` | ⚠️ 新增1字段 | ✅ 一致 | ✅ 一致 | ✅ 一致 | ✅ 通过 |
```

6. 校验不通过 → 标注"回退 Phase 5"，说明不兼容的差异项

#### 约束

- 仅对比本次修改涉及的 API 接口
- 如果探路报告中无旧接口样本，跳过此项校验（标注"⚠️ 无旧样本，跳过"）
- 字段删除和类型变化必须回退 Phase 5

### Step 2.5: 不变性门验证 + 全量文档完整性扫描

先执行不变性门自动验证，再执行文档完整性扫描：

#### Step 2.5a: 不变性门自动验证（I7/I8）

Read `scripts/invariant-gates.md` 并执行以下不变性门的 grep 验证：

| 不变性 | 验证内容 | 命令 | 阈值 |
|--------|---------|------|:----:|
| **I7** | 接口文档覆盖所有 Controller 方法 | `grep -rPn '@(Get\|Post\|Put\|Delete\|Patch)Mapping' "{controller_dir}"` 对比 `ls "{api_doc_dir}"/*.md` | 所有新增/修改方法已覆盖 |
| **I8** | 前端 API 调用路径有对应文档 | `grep -rPn 'defHttp\.(get\|post\|put\|delete)\([''"`]\K[^''"`]+' "{frontend_api_dir}"` 对比 `grep -rPn '`/[^`]+`' "{api_doc_dir}"/*.md` | 所有前端路径已覆盖 |

执行 grep 命令验证，输出结果表格追加到审核报告。

#### Step 2.5b: 全量文档完整性扫描

在 I7/I8 不变性门通过后，进一步执行文档完整性扫描，确保代码中存在但文档缺失的部分被补全：

| 扫描项 | 方法 | 通过条件 | 不变性 |
|--------|------|---------|:------:|
| 接口文档覆盖度 | Grep 搜索前端 API 调用路径 (`defHttp.post/get` 中的 url)，与 `docs/接口文档/` 中的接口逐条比对 | 所有前端调用的 API 接口在文档中有记录 | I7, I8 |
| 业务规则文档覆盖度 | 检查修改涉及的业务逻辑是否在 `docs/业务规则/` 中有对应文档 | 涉及的业务规则已记录 | — |
| 全API索引参数准确性 | 比对 `00-全API索引.md` 中的参数描述与 Controller 代码实际参数 | 参数名/说明与实际一致 | I7 |
| Controller 覆盖度 | 检查修改涉及的 Controller 是否在接口文档中有对应章节 | Controller 在文档中有记录 | I7 |

**扫描流程**：
1. 搜索探路报告记录的前端文件路径中与本次修改相关的前端文件，提取所有 `defHttp.post/get` 的 url 路径
2. 搜索后端 Controller 的 `@RequestMapping` 基路径
3. 在 `docs/接口文档/` 中搜索每个 API 路径是否存在
4. 缺失的 API 文档 → 标记缺失，回退至 Phase 5.6 补充
5. 参数描述不准确 → 修正 `00-全API索引.md`
6. 更新 `docs/接口文档/INDEX.md` 的日期行

**输出**：扫描结果追加到 Step 4 的标准表格，新增一行 `文档完整性扫描`。

### Step 3: 端到端场景验证

审核通过后，必须按 Phase 3 规格文档的 AC 逐条执行端到端场景验证：

| 验证层次 | 方法 | 适用场景 | 通过条件 |
|---------|------|---------|---------|
| **后端 API** | `api-fetcher` / `curl` / `powershell` 调用实际接口 | 后端修改 | 返回预期数据 |
| **后端翻页/排序/过滤** | 按 `references/e2e-test-rules.md` 执行 API 层 E2E | 翻页/排序/过滤/搜索修改 | 8步全部通过 + 排序确定性验证 |
| **数据库** | `mysql-{子项目名}.execute_query` 查询 | 数据变更 | 与预期一致 |
| **前端交互（自动）** | `npx playwright test e2e/` | 前端修改 | 全部测试通过 |
| **前端交互（手动）** | 标注 "⚠️ 需人工测试" | 无法自动化 | 用户确认 |

**验证流程**：

1. 读取规格文档 AC 清单
2. 对后端修改：`mvn compile` + `api-fetcher` 调用确认
3. 对涉及翻页/排序/过滤/搜索的后端修改：按 `references/e2e-test-rules.md` 执行 API 层 E2E 测试（含排序确定性验证、数据隔离验证）
4. 对前端修改涉及 UI 交互的 AC：执行 `/e2e` 生成 Playwright 测试脚本
5. 运行 `npx playwright test` 验证 UI 流程
6. 无法自动化的步骤标注 "⚠️ 需人工测试"
7. 验证发现与 AC 不符时标记"回退 Phase 5"

**使用 `/e2e` 命令**：
```
/e2e                                     # 自动从规格文档 AC 生成测试
/e2e 登录后查看文章列表，点击详情页     # 指定测试场景
```

生成后在 `e2e/` 目录生成 `{功能简述}.spec.ts`，运行确认全部通过。
6. 验证发现与 AC 不符 → 标记回退 Phase 5，在输出表格追加 "🚫 回退 Phase 5"

**影响**: 验证结果追加到 Step 4 的标准表格底部。

### Step 4: 合并结果为标准表格

汇总所有审核结果（含不变性门），输出 9 行标准表格：

```markdown
## 文档代码审核（Phase 5.5）
| 审核项 | 状态 | 说明 |
| **I7-接口文档覆盖** | ✅ 全覆盖 / ❌ 有遗漏 | {覆盖比例} |
| **I8-前端API文档覆盖** | ✅ 全覆盖 / ❌ 有遗漏 | {覆盖比例} |
| FE-探路报告 | ✅ 一致 / 🔧 已更新 | {说明} |
| FE-规格文档 | ✅ 一致 / 🔧 已更新 | {说明} |
| FE-代码质量 | ✅ 通过 / ⚠️ 建议 | {说明} |
| BE-探路报告 | ✅ 一致 / 🔧 已更新 | {说明} |
| BE-规格文档 | ✅ 一致 / 🔧 已更新 | {说明} |
| BE-代码质量 | ✅ 通过 / ⚠️ 建议 | {说明} |
| API接口文档 | ✅ 一致 / 🔧 已更新 / 🆕 已新增 | {说明} |
| **文档完整性扫描** | ✅ 全覆盖 / ✅ 全覆盖（X 项已补全） / ❌ 有遗漏未补 | {缺失项数 / 补全项数} |
| **端到端验证** | ✅ 通过 / 🚫 回退 Phase 5 / ⚠️ 需人工测试 | {说明} |
```

### 复盘标记

审核结果中自动标记是否需要进入 Phase 6.7 复盘：

| 标记条件 | 标记为 |
|---------|--------|
| FE-代码质量 / BE-代码质量 为 ⚠️ 建议 | 🔄 需复盘 |
| 探路报告/规格文档/API文档 ≥2 项不一致 | 🔄 需复盘 |
| 全部通过（无 ⚠️ 无不一致） | ➡️ 无需复盘 |

标记规则由主流程在执行 Step 3 合并且输出表格时一并判定，将结果传递给 Phase 6 Step 6。

## 完整性自检

- [ ] 两个子代理都已返回结果
- [ ] FE-探路报告/规格文档/代码质量三项均已审核
- [ ] BE-探路报告/规格文档/代码质量三项均已审核
- [ ] API 接口文档全量比对完成
- [ ] docs/接口文档/INDEX.md 已同步（涉及新增模块时）
- [ ] docs/接口文档/00-全API索引.md 已同步
- [ ] **文档完整性扫描已完成**：前端 API 调用路径与接口文档逐条比对，缺失文档已补全
- [ ] **I7 不变性门（接口文档覆盖 Controller 方法）已验证**
- [ ] **I8 不变性门（前端 API 文档覆盖）已验证**
- [ ] 代码问题已退回 Phase 5（如有）
- [ ] 标注了审核日期
- [ ] 审核意见已遵循 communication-rules 的 concise 级别

## 约束

- 只改文档不改代码（代码问题回 Phase 5）
- Agent FE 只审前端文件（探路报告记录的前端路径），前置加载 `vue-standards` 子技能
- Agent BE 只审后端文件（探路报告记录的后端路径），前置加载 `java-review-standards` 子技能，参考开发规范可辅助加载 `java-dev-standards`
- 标注更新日期
- 审核记录必输出
- API 文档缺失必须补充
- 涉及新增模块时同步更新接口文档目录
- **沟通规则**：审核意见使用 **concise 级别**（一行一条、直奔主题、`L42: 🔴 bug: ...` 格式）。详见 `references/communication-rules.md`。
- **I7/I8 不变性门不可跳过**：接口文档覆盖验证不通过时不可推进至 Phase 出口。

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 `artifacts.review.status`, `artifacts.review.issues`, `artifacts.review.summary`
2. 更新 `artifacts.gate_3.status`, `artifacts.gate_3.failed_items`
3. 参数: current_phase="review", next_phase="record"（通过）/ "code"（代码问题）, is_gate_phase=true
4. 额外：有代码问题时 `metrics_snapshot.gate_retries += 1`
5. `gates_summary` 更新计数
