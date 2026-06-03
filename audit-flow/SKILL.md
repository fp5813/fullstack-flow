---
name: fullstack-flow/audit-flow
description: "fullstack-flow 流程审计 — 标准化全量审核（20 项检查）所有 Phase/子技能/引用/配置的一致性、完整性和无硬编码，低风险项自动修复，输出审计报告并记录到改进追踪。"
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Bash
user-invocable: true
---

# fullstack-flow 流程审计

> 默认审核 fullstack-flow 下的所有文件。发现问题自动修复或输出"需人工处理"。
> 每次审计结果记录到 `proccess_improvements.yaml`，形成自我进化机制。

## 审计覆盖范围

| 类别 | 扫描范围 | 文件数 |
|------|---------|--------|
| 主流程 | `SKILL.md` | 1 |
| Phase 文档 | `phases/*.md` | 12 |
| 子技能 | `{sub-skill}/*/SKILL.md` | 6 |
| References | `references/*.md` | 9 |
| 工作流 | `.codebuddy/workflow/` | state.yaml + state.schema.yaml |
| MCP 配置 | 项目根 `.mcp.json` | 1 |
| 项目文档 | `docs/` | INDEX 系列 + 模板 |
| 改进追踪 | `.codebuddy/workflow/metrics/` | 门控/耗时/改进记录 |

## 执行流程

### Step 1: 注册完整性检查

#### 1.1 Phase 文件注册检查

对比 `phases/` 目录文件列表与 SKILL.md frontmatter 的 `skills` 列表：

```bash
ls phases/*.md | sed 's|phases/\(.*\)\.md|fullstack-flow/phases/\1|'
# vs SKILL.md 中 skills: 列表下的 phase-* 条目
```

缺失 → 自动追加到 SKILL.md skills 列表。

#### 1.2 子技能注册检查

对比子技能目录（排除 phases, references）与 SKILL.md `skills` 列表：

```bash
ls -d */SKILL.md 2>/dev/null | grep -v phases | grep -v references | sed 's|/SKILL\.md||'
# vs SKILL.md 中 skills: 列表
```

缺失 → 自动追加。

#### 1.3 References 注册检查

对比 `references/*.md` 与 SKILL.md `references` 列表：

```bash
ls references/*.md | sed 's|references/\(.*\)|references/\1|'
# vs SKILL.md 中 references: 列表
```

缺失 → 自动追加。

### Step 2: 引用一致性检查

#### 2.1 Phase 文档引用的参考文件

对所有 Phase 文档执行 Grep，提取 `references:` 中的文件路径，检查目标文件是否存在：

| 引用模式 | 目标路径 | 检查方法 |
|---------|---------|---------|
| `../references/xxx.md` | `references/xxx.md` | `ls references/xxx.md` |

#### 2.2 跨文件链接有效性

对 Phase 文档中的 Markdown 链接 `[text](path)` 做存在性检查（忽略外部 URL）。

### Step 3: 无硬编码检查

对所有 Phase/子技能/Reference 文档执行 Grep：

| 模式 | 含义 | 严重度 |
|------|------|--------|
| `archive` | 旧项目关键词（示例除外） | ⚠️ |
| `D:/JavaProjects/` | 本地绝对路径 | ❌ |
| `D:/LenovoSoftstore/` | 本地软件路径 | ❌ |
| `/home/` | Linux 本地路径 | ⚠️ |
| `C:/Users/` | Windows 本地路径 | ⚠️ |

**自动修复**：
- 绝对路径 → `{project-root}` / `{mcp-servers-path}` 占位符
- `archive` 关键词（非示例语境）→ 通用描述

### Step 4: MCP 一致性检查

#### 4.1 服务名清单比对

对比 `.codebuddy/skills/fullstack-flow/references/mcp-tools-summary.md` 中的 MCP 服务表与 `.mcp.json` 中的 `mcpServers`：

```bash
# .mcp.json 服务名
grep -E '"[a-zA-Z0-9_-]+":\s*\{' .mcp.json | sed 's/"//g; s/:.*//'

# mcp-tools-summary.md 表格中的服务名
grep '| `' references/mcp-tools-summary.md | sed 's/.*| `//; s/` .*//'
```

不一致 → 输出差异报告，标记"需人工处理"。

#### 4.2 Phase 文档 mysql 服务名一致

检查 Phase 文档中的 `mysql-{子项目名}` 模式是否与 mcp-tools-summary.md 中的服务名模式一致。

### Step 5: 状态一致性检查

#### 5.1 state.yaml schema 验证

对照 `state.schema.yaml` 规则逐条验证 `state.yaml`：

| 规则 | 检查 | 通过条件 |
|------|------|---------|
| R1 | phase.current 存在 | 非空 |
| R2 | phase.status 有效 | pending / in_progress / completed |
| R3 | 制品路径存在 | artifacts 中已标记的文件存在 |

#### 5.2 Phase 转换表一致性

对比 `SKILL.md` 中"Phase 转换速查"表与 `state.schema.yaml` 的规则：

- state.schema.yaml 中每个 Phase 的 entry/exit 规则
- 与 SKILL.md 的转换表描述的 next/gate 一致

### Step 6: 文档完整性检查

| 检查项 | 路径 | 方法 |
|--------|------|------|
| 文档中心 INDEX | `docs/INDEX.md` | `ls` 确认 |
| 探路报告 INDEX | `docs/探路报告/INDEX.md` | `ls` 确认 |
| 规格文档 INDEX | `docs/规格文档/INDEX.md` | `ls` 确认 |
| 实施计划 INDEX | `docs/实施计划/INDEX.md` | `ls` 确认 |
| 修改记录 INDEX | `docs/修改记录/INDEX.md` | `ls` 确认 |
| 修改记录 TEMPLATE | `docs/修改记录/TEMPLATE.md` | `ls` 确认 |
| 业务规则 INDEX | `docs/业务规则/INDEX.md` | `ls` 确认 |
| 接口文档 INDEX | `docs/接口文档/INDEX.md` | `ls` 确认 |
| 复盘记录 INDEX | `docs/复盘记录/INDEX.md` | `ls` 确认 |
| 决策日志 INDEX | `.codebuddy/workflow/decisions/INDEX.md` | `ls` 确认 |
| metrics INDEX | `.codebuddy/workflow/metrics/INDEX.md` | `ls` 确认 |

缺失 → 按标准格式创建。

### Step 7: 版本一致性检查

- 读取所有 Phase 文档的 frontmatter `version` 字段
- 检查版本号格式是否一致（semver）
- 检查 CHANGELOG.md 是否存在并包含当前版本记录

### Step 8: 输出审计报告

```markdown
## fullstack-flow 审计报告
**日期**：YYYY-MM-DD
**检查范围**：{文件数} 个文件

| 维度 | 状态 | 问题数 | 说明 |
|------|------|:------:|------|
| 注册完整性 | ✅ | 0 | 全部注册 |
| 引用一致性 | ✅ / ⚠️ / ❌ | N | {发现问题} |
| 无硬编码 | ✅ / ⚠️ / ❌ | N | {发现问题} |
| MCP 一致性 | ✅ / ⚠️ / ❌ | N | {发现问题} |
| 状态一致性 | ✅ / ⚠️ / ❌ | N | {发现问题} |
| 文档完整性 | ✅ / ⚠️ / ❌ | N | {发现问题} |
| 版本一致性 | ✅ / ⚠️ / ❌ | N | {发现问题} |
| **总计** | **✅ {X} 项通过 / ⚠️ {Y} 项警告 / ❌ {Z} 项失败** | **{总计}** | |

### 自动修复记录

| 问题 | 修复操作 | 状态 |
|------|---------|------|
| {发现问题} | {修复方式} | ✅ 已修复 / ⚠️ 需人工处理 |

### 改进追踪已记录

本次审计发现已追加到 `.codebuddy/workflow/metrics/proccess_improvements.yaml`。
```

### Step 9: 记录改进追踪

对每个发现的问题（含自动修复的），追加记录到 `proccess_improvements.yaml`：

```yaml
- id: IMP-{自增序号}
  date: "{YYYY-MM-DD}"
  source: "audit-flow"
  issue: "{问题描述}"
  type: "{hardcode|missing-ref|inconsistency|missing-file}"
  file: "{问题文件路径}"
  fix: "{修复方式}"
  status: "{fixed|manual-required}"
```

## 自检

- [ ] 审计范围覆盖：SKILL.md + 12 Phase + 6 子技能 + 9 References + 配置/文档
- [ ] 注册完整性检查（Phase/子技能/References 三方向）
- [ ] 无硬编码检查已执行（archive/绝对路径/本地路径）
- [ ] MCP 一致性已比对
- [ ] 状态一致性已验证
- [ ] 文档完整性已确认（所有 INDEX + TEMPLATE）
- [ ] 低风险问题已自动修复
- [ ] 高风险问题标记"需人工处理"
- [ ] 审计报告已输出
- [ ] 改进追踪已记录

## 约束

- 只审计不修改代码（除修复低风险配置项外）
- 自动修复仅限：SKILL.md 注册补全、占位符替换
- MCP 服务名不一致标记"需人工处理"（涉及运行态）
- 审计报告追加到 `docs/修改记录/`（视本次审计为一次修改）
