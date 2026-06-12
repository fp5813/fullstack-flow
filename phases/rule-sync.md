---
name: fullstack-flow/phases/rule-sync
description: "业务规则同步：评估本次修改是否涉及业务规则，涉及则更新 docs/业务规则/ 并同步到 .codebuddy/rules/。"
version: 3.2.0
tags: [fullstack, codebuddy-only, rule-sync, documentation]
role: codebuddy-recorder
model: deepseek-v4-flash
tools: [Read, Grep, Agent]
references: []
---

# Phase 6.5: 业务规则同步

> 老项目需要持续沉淀业务规则，每次修改都是更新规则的机会。文档一致性核对已移至 Phase 5.5。

## 快速概览

```
Step 0 读取状态 → Step 1 评估是否涉及规则 → Step 2 定位并更新规则文件
  → Step 3 同步 .codebuddy/rules/ → Step 3.6 Instinct 捕获 → Phase 出口 → audit
```

**核心**: 业务规则更新 + 领域 Instinct 提取（常量/枚举/状态流转/权限/过滤/渲染）

## 职责

**输入**：Phase 5 代码变更  
**输出**：更新/新增 `docs/业务规则/{模块}/*.md`，同步 `.codebuddy/rules/`  
**模式**：只读 + 写规则文件

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="rule-sync", next_phase="audit"
- 额外：验证前置制品 `artifacts.change_record.path` 存在

### Step 1: 评估是否涉及业务规则

**属于业务规则**：新增状态/条件判断、修改业务逻辑、新增字段约束、新增 API 权限规则、数据库约束变更。

**不属于业务规则**：修复空指针、修改文案、代码风格优化、性能优化。

不涉及 → 在修改记录注明"不涉及业务规则变更"，跳过后续。

### Step 2: 定位并更新规则文件

| 场景 | 操作 |
|------|------|
| 已有规则文件 + 新规则 | 追加章节 |
| 已有规则文件 + 旧规则改动 | 更新描述，标注"更新于 {日期}" |
| **已有规则 30 天未更新** | 标注 `⚠️ 待确认：该规则可能已过期`，要求用户确认 |
| **规则引用的类/方法已不存在** | 标注 `❌ 已废弃：引用源头 {类名} 已删除`，直接清除该规则 |
| 无对应文件 + 涉及规则 | 按模板新建 |

### Step 3: 同步到 .codebuddy/rules/

```yaml
---
globs: ["{匹配路径模式}"]
description: "{规则简要描述}"
alwaysApply: false
---
```
- globs 精确到被影响的最小路径
- `alwaysApply: false`，已存在文件用更新操作

### Step 3.5: 更新 INDEX（Read `scripts/update-index.md`）
- prepend mode: 在 `docs/业务规则/INDEX.md` 表顶部插入新行

### Step 3.6: Instinct 捕获

> 业务规则往往是高价值的领域模式。在更新业务规则的同时，提取可复用的业务模式到 Instinct。

**触发条件**：本次修改涉及业务规则变更时，检查以下场景：

| 业务规则类型 | 领域 | 提取为 Instinct |
|-------------|:----:|----------------|
| 常量/枚举定义 | `数据` | 涉及业务常量枚举时，记录字段含义和取值约束 |
| 状态流转规则 | `领域` | 涉及状态机/状态流转时，记录状态变迁路径 |
| 权限注解约束 | `安全` | 涉及 `@RequiresPermissions` 时，记录权限命名模式和影响 |
| 数据过滤规则 | `数据` | 涉及数据权限/过滤条件时，记录过滤实现模式 |
| 前端条件渲染 | `UX` | 涉及 v-if/v-show 条件时，记录组件显隐的业务条件 |

**执行步骤**：
1. 对每个匹配的规则类型，生成 unique ID
2. 按 `references/instinct-reference.md` 格式写入 instinct 文件
3. 更新 `.codebuddy/instincts/INDEX.md`

### Step 4: 在修改记录中注明

```markdown
## 业务规则同步
- [x] 本次修改涉及业务规则
- [x] 已更新：`docs/业务规则/{模块}/{文件名}.md`
- [x] 变更描述：{简要说明}
```

## 自检

- [ ] 已评估是否涉及业务规则
- [ ] 涉及时已更新/新建规则文件
- [ ] `.codebuddy/rules/` 已同步（globs + description + alwaysApply: false）
- [ ] globs 配置精确，避免过度加载
- [ ] workflow/state.yaml 的 artifacts.rule_sync 已更新

## 约束

- 只写规则文件，不修改源代码。有则可写，不强制。按模块组织，保持简洁。

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 `artifacts.rule_sync.status`, `artifacts.rule_sync.files_updated`（不涉及时设为 not_applicable）
2. 参数: current_phase="rule-sync", next_phase="audit"
