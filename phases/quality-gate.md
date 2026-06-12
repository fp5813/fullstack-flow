---
name: fullstack-flow/phases/quality-gate
description: "探路报告质量门控：35 项检查覆盖 L0-L5 完整性、文档查阅、工具规范性。通过率 100% 方可进入 Phase 3。"
version: 3.0.0
tags: [fullstack, codebuddy-only, quality-gate, read-only]
role: codebuddy-gatekeeper
model: deepseek-v4-flash
tools: [Read, Grep]
---

# Phase 2.5:  质量门控

> Checklist = "需求编写的单元测试" — 检验探路报告本身的质量。

## 快速概览

```
Step 0 读取状态 → Step 0.5 不变性门自动验证（I1/I2/I9/I10）
  → 执行 35 项完整性检查（L0-L5，标注 I1-I10 引用）
  → Gate 1 判定（100% 通过方可推进）→ Phase 出口
```

**核心**: 不变性门自动验证 + 35 项完整性检查 / 二值判定（通过/回退）/ 无灰色地带

## 职责

**输入**：探路报告（`docs/探路报告/YYYY-MM-DD-{简述}.md`）  
**输出**：检查结果（追加到探路报告末尾的 `## 质量检查` 章节）  
**模式**：只读（不修改源文件）

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="quality-gate", next_phase="spec"（通过）/ "probe"（不通过）, is_gate_phase=true

### Step 0.5: 不变性门自动验证

Read `scripts/invariant-gates.md` 并执行以下不变性门的 grep 验证：

| 不变性 | 验证内容 | 命令 | 阈值 |
|--------|---------|------|:----:|
| **I1** | file:line 引用完整 | `grep -cP '\.(java\|vue\|ts\|js\|xml):\d+' "{probe_report_path}"` | ≥3 |
| **I2** | L0-L3 分层引用完整 | 分层 grep 检查（vue/Controller/Service/Mapper） | 每层 ≥1 |
| **I9** | 无修改建议/实施方案 | `grep -ciP '(修改建议\|实施方案\|修改如下)' "{probe_report_path}"` | =0 |
| **I10** | 涉及 DB 时数据采样完整 | `grep -ciP '(SELECT \|INSERT \|FROM \|table_name\|数据行)'` | ≥3 |

执行 bash/grep 命令验证，输出结果表格追加到探路报告末尾的 `## 质量检查` 章节之前。

不变性门通过后才进入人工 35 项检查。不变性门失败 → 直接回退 Phase 2 补充探路报告。

## 检查项（35 项，标注不变性引用）

### L0 层状导航完整性（→ I1, I2）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C01 | 页面→API 调用链完整 | I1 |
| C02 | API→Service→Mapper 链路贯通 | I1 |
| C03 | 每步标注 file:line | I1 |
| C04 | 涉及 DB 时包含数据库节点 | I2, I10 |

### L1 页面入口覆盖度（→ I2）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C05 | 路由路径明确 | I2 |
| C06 | Vue 组件精确到文件:行号 | I2 |
| C07 | API 调用链路完整（组件→API→HTTP 端点） | I2 |
| C08 |  列出相关子组件及路径 | I2 |

### L2 API 入口精确性（→ I1, I2）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C09 | Controller 方法有 file:line | I1, I2 |
| C10 | Service 方法有 file:line | I1, I2 |
| C11 | Mapper/XML 行号已标注 | I1, I2 |
| C12 | Controller 基路径已标注 | I2 |

### L3 实现类索引完整性（→ I2）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C13 | Controller 类列出 | I2 |
| C14 | Service 接口+实现列出 | I2 |
| C15 | Mapper/XML 列出 | I2 |
| C16 | Entity/DTO 列出 | I2 |

### 探路工具使用
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C17 | 使用至少两种探路工具（codegraph + mysql） | — |
| C18 | 查阅了项目文档 | — |
| C19 | 业务规则发现已记录 | — |
| C20 | 探路深度合理（每代理 ≤3 次调用） | — |
| C21 |  不确定处标注"(待验证)" | — |

### L4 数据库覆盖度（→ I10）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C22 | 涉及 DB 时列出表名 + Entity 映射 | I10 |
| C23 | 关键字段有类型和值域 | I10 |

### L5 前端 API 与组件（→ I2）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C24 | 涉及前端时列出 API 文件 + 方法 | I2 |
| C25 | 涉及前端时列出子组件 | I2 |

### 报告避免项（→ I9）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C26 | 不含修改建议 | I9 |
| C27 | 不含实施方案 | I9 |

### ⚡ 复盘新增：探路覆盖率检查（→ I2）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C28 | 是否覆盖了所有 UI 入口路径（列表页/详情弹框/编辑弹框/自定义弹框） | I2 |
| C29 | 是否检查了封装层/中间件（onChange wrapper、componentProps 包裹、formActionType 作用域） | I2 |
| C30 | 异步函数是否标注了竞态风险（await 期间 state 被外部修改的可能性） | I2 |

### 📊 数据覆盖检查（C31-C35 → I10）
| # | 检查项 | 不变性 |
|---|--------|:------:|
| C31 | 涉及 DB 时已查询真实数据行（至少正常流程 3 行 + 边界场景 DISTINCT）；`LIMIT 3` 返回行覆盖了不同状态值分布（同一状态多行时标注其代表性） | I10 |
| C32 | 枚举/字典/状态字段值域已列出（通过 DISTINCT 查询或 `@Dict` 注解解析） | I10 |
| C33 | 数据写入链路已标注（哪个 Controller→Service→Mapper 负责数据的增/删/改）**且**数据转换逻辑已说明（列举字段 A→转换规则→目标字段的映射关系） | I10 |
| C34 | 涉及后端时已记录 VO/DTO 类名和字段结构（反映业务数据视图而非原始表结构）；VO/DTO 字段赋值来源已标注（直接映射/枚举转换/字典翻译/计算派生/聚合统计） | I10 |
| C35 | 表字段与 VO/DTO 字段映射关系已对照记录（列出不一致或转换之处，如字段名不同、类型转换、值域映射） | I10 |

### 自适应跳检

| Ticket 类型 | 跳过项 |
|-------------|--------|
| 纯前端 | C04, C11, C15, C16, C22, C23, C31, C32, C33, C34, C35 |
| 纯后端 | C05-C08, C24, C25 |
| 数据库修改 | C05-C08, C09-C12 部分, C24, C25, C33（如果只读查询），C34, C35（如果无 VO/DTO 变更） |
| 纯配置 | 仅保留 C17-C21 |

### 出口更新：更新工作流状态

更新 `artifacts.gate_2_5` 的 `status/score/failed_items/summary` 字段。

## 门控判定

通过率 100% → ✅ 进入 Phase 3  
＜100% → ❌ 回到 Phase 2，重新探路

## 输出

不变性门验证结果追加在探路报告末尾、质量检查章节之前：

```markdown
## 不变性门验证结果
| ID | 不变性 | 状态 | 证据 |
|----|--------|:----:|------|
| I1 | file:line 引用完整 | ✅ | 找到 12 处引用 |
| I2 | 分层引用完整 | ✅ | L0:3 L1:4 L2:3 L3:2 |
| I9 | 无修改建议 | ✅ | 0 处违规 |
| I10 | 数据采样完整 | ✅ | 找到 7 处 SQL 引用 |
```

质量检查结果：

```markdown
## 质量检查（Phase 2.5 Gate）
**通过率**: {pass}/{total} = {percent}%
**失败项**:  | # | 检查项 | 原因 | 修复建议 | 不变性 |
**门控状态**: {✅ 通过 / ⚠️ 警告 / ❌ 不通过}
**复盘标记**: {失败项 ≥1 时标注 ⚠️ 需复盘，供 Phase 6.7 入口判定}
```

不变性门失败（I1/I2/I9/I10 任一不通过）→ 直接回退 Phase 2，不进入人工检查。
不通过时停止流程，输出失败项清单。

失败项 ≥1 时，标记 **⚠️ 需复盘**，供 Phase 6 Step 6 复盘四问读取。

## 约束

- 绝不写代码。只追加不覆盖。透明公开（每个失败项说明具体原因）。
- **不变性门不可跳过**：I1/I2/I9/I10 任一不通过 → 直接回退 Phase 2，不进入人工 35 项检查。

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 artifacts.gate_2_5：status/score/failed_items/summary
2. 参数: current_phase="quality-gate", next_phase="spec"（通过）/ "probe"（不通过）, is_gate_phase=true
3. 额外：不通过时 `metrics_snapshot.gate_retries += 1`
4. `gates_summary` 更新计数
