---
name: fullstack-flow/phases/retrospect
description: "复盘回顾：混合触发式复盘（轻量检查+五维分析+根因分析+失败案例结构化），输出复盘文档和失败案例并执行技能改进沉淀与汇总检查。"
version: 2.0.0
tags: [fullstack, codebuddy-only, review, retrospective, improvement, failure-case, health-check]
role: codebuddy-recorder
model: deepseek-v4-flash
tools: [Read, Write, Edit, Grep, Agent]
references:
  - ../references/codegraph-reference.md
  - ../references/spec-driven-development.md
---

# Phase 6.7: 复盘回顾

> 代码改完了，复盘是最后一环。发现问题不追溯，下次还会犯同样错误。
> 混合触发：每次开发后轻量检查"复盘四问"，发现问题才输出完整复盘报告。

## 快速概览

```
Step 0 读取状态 → Step 1 收集素材 → Step 2 五维分析
  → Step 2.3 失败案例结构化 → Step 2.5 Instinct 提取
  → Step 3 生成沉淀建议 → Step 4 执行沉淀（技能/Memory/复盘文档/失败案例）
  → Step 5 汇总检查 → Phase 出口 → rule-sync
```

**核心**: 五维分析（流程/代码/惯例/技能/根因）+ Instinct 提取 + 技能改进沉淀

## 输入

- Phase 6 修改记录（`docs/修改记录/`）
- Phase 2.5 质量门控输出（如有）
- Phase 5.5 审核输出（如有）
- Phase 6.6 审计报告（如有）
- 最近复盘记录（`docs/复盘记录/`）
- Memory/feedback 历史记录

## 输出

- `docs/复盘记录/YYYY-MM-DD-{简述}.md` — 复盘文档
- `docs/复盘记录/INDEX.md` — 索引更新
- `docs/失败案例/{YYYY}/FC-{ID}.md` — 失败案例文件（如需创建）
- `docs/失败案例/FAILURE-INDEX.md` — 失败案例索引更新
- 技能文件更新（SKILL.md 规则 / Phase 检查项 / 参考文献）
- Memory/feedback 更新
- `.codebuddy/workflow/metrics/proccess_improvements.yaml` 更新（汇总检查发现重复模式时）
- CHANGELOG 更新

## 模式

只读 + 写入（分析过程只读，输出阶段写复盘文档和技能文件）

---

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="retrospect", next_phase="rule-sync"

### Step 1: 收集素材

> **重要**：所有素材优先从文件系统读取（`Read docs/...`），而非依赖上下文记忆。自动压缩可能已将对话中的制品摘要化，文件系统保留完整版本。

从以下来源收集复盘素材：

| 来源 | 什么信息 | 问什么 |
|------|---------|--------|
| Phase 6 修改记录 | 修改原因、技术难点、遇到的问题 | 为什么这次修改是必要的？ |
| Phase 2.5 门控报告 | 门控失败项 | 门控发现了什么？为什么没拦住？ |
| Phase 5.5 审核输出 | ⚠️ 建议项、不一致项 | 审核发现了什么质量问题？ |
| Phase 6.6 审计报告 | 未归档规则清单 | 审计发现了什么遗漏？ |
| 最近复盘记录 | 同类问题历史 | 是不是重复踩坑？ |
| Memory/feedback | 已沉淀的惯例和教训 | 现有沉淀是否覆盖当前场景？ |

### Step 2: 五维分析

按以下五个维度逐一分析，输出分析结果：

#### 维度一：流程缺陷

分析哪个 Phase 的流程或检查项未能预防此问题：

| 问题表现 | 应拦截 Phase | 实际拦截情况 | 改进方向 |
|---------|-------------|-------------|---------|
| {描述} | Phase N（如 Phase 2） | 未拦截 / 检查项不足 | 新增/修改检查项或流程步骤 |

#### 维度二：代码质量

分析代码中的反模式或可改进模式：

| 问题代码模式 | 正确做法 | 沉淀到 |
|------------|---------|--------|
| {反模式描述} | {推荐模式} | references/ 或 Memory |

#### 维度三：项目惯例

分析是否有新的设计决策、组件使用约束需要记录：

| 决策内容 | 约束说明 | 适用场景 |
|---------|---------|---------|
| {决策} | {约束} | {场景} |

#### 维度四：技能可用性

分析 Phase 文档自身是否有缺陷（描述不清、步骤缺失、示例不足）：

| 文档缺陷 | 影响 | 改进建议 |
|---------|------|---------|
| {描述} | {影响范围} | {改进} |

#### 维度五：根因分析（5 Why）

> 仅 BUG 修复必填此维度。其他触发类型可选。

| 轮次 | 问题 | 答案 |
|------|------|------|
| Why 1 | {问题表现} | {直接原因} |
| Why 2 | 为什么会有这个直接原因？ | {第二层原因} |
| Why 3 | ... | ... |
| Why 4 | ... | ... |
| Why 5 | 流程/技能/检查项层面的根因 | {根本原因} |

### Step 2.3: 失败案例结构化

> 将五维分析中根因明确的失败提炼为标准化失败案例，形成可检索的知识资产。
> 详细模板见 `docs/失败案例/TEMPLATE.md`。

#### 2.3.1 判定是否需要创建失败案例

| 触发条件 | 必填/建议 |
|---------|:---------:|
| 本次为 BUG 修复复盘 | **必建** |
| 同一类问题在复盘记录中第二次出现 | **必建** |
| 门控连环失败（连续 2+ 次门控失败同一检查项） | 建议创建 |
| 用户明确要求 | **必建** |
| 新增设计模式/约定复盘 | 可选（如有正例可创建"经验案例"） |

#### 2.3.2 创建失败案例文件

按以下结构生成 YAML frontmatter + Markdown 正文。参考 `docs/失败案例/TEMPLATE.md`：

```markdown
---
id: FC-{YYYYMM}-{序号（从 001 开始）}
title: "简洁描述（30 字以内）"
domain: "bug/design/process/security"
severity: "critical/major/minor"
created_at: "{YYYY-MM-DD}"
source: "retrospect"
related_retrospect: "docs/复盘记录/{文件名}"
related_modification: "docs/修改记录/{文件名}"
tags: ["{关键词1}", "{关键词2}"]
---

## 问题现象
...

## 根因分析（5 Why）
| Why 1 | ... |
...

## 未拦截的 Phase
| Phase | 应做什么 | 实际 | 缺失 |
...

## 修复方案
- 短期修复：...
- 长期预防：...

## 沉淀项
| 类型 | 目标 | 变更 |
...
```

存储路径：`docs/失败案例/{YYYY}/FC-{YYYYMM}-{XXX}.md`

#### 2.3.3 关联复盘记录

在复盘文档的前言或末尾注明：

```markdown
相关失败案例: FC-{YYYYMM}-{XXX}
```

#### 2.3.4 更新 INDEX（Read `scripts/update-index.md`）
- prepend mode: 在 `docs/失败案例/FAILURE-INDEX.md` 表顶部插入新行

### Step 2.5: Instinct 提取（新模式捕获）

> 从本次实施中识别可复用的模式，生成 Instinct。详细定义见 `references/instinct-reference.md`。

#### 2.5.1 模式识别

从五维分析结果和本次实施过程中识别可复用的模式：

| 来源 | 可提取的模式类型 |
|------|----------------|
| 维度二（代码质量）中的"正确做法" | `代码` 领域 instinct |
| 维度三（项目惯例）中的设计决策 | `领域` / `UX` 领域 instinct |
| 本次修改涉及的 VO/DTO/表结构 | `数据` 领域 instinct |
| 本次修改涉及的权限/安全处理 | `安全` 领域 instinct |
| 门控常失败项（维度一） | `流程` 领域 instinct |
| Phase 5 创建的测试类模式 | `测试` 领域 instinct |

#### 2.5.2 生成 Instinct

对每个识别的模式，按以下流程生成 instinct：

1. **确定唯一 ID**: `{domain}-{简短-kebab-case-描述}`
2. **设置置信度**: 首次提取 0.60，重复出现时读取已有 instinct 并增加置信度
3. **写入文件**: 
   - 项目级：`.codebuddy/instincts/personal/YYYY-MM-{id}.yaml`
   - 如果已存在同 id 文件：更新 `confidence`、`occurrences`、`evidence`
4. **证据链**: 在 `evidence` 中追加本次修改记录和探路报告路径

#### 2.5.3 更新 Instinct 索引

检查 `.codebuddy/instincts/INDEX.md` 存在性：
- 如果不存在：创建索引文件
- 如果存在：追加新行

```markdown
| {YYYY-MM-DD} | [{title}](./personal/{file}) | {domain} | {confidence} | {project/global} |
```

#### 2.5.4 全局提升检查

对新生成的 instinct 检查：
- 如果置信度 ≥ 0.80，建议用户执行 `/instinct-promote` 提升到全局
- 如果已有 3+ 项目存在相同模式，自动标记为全局候选

### Step 3: 生成沉淀建议

根据五维分析结果，生成具体的沉淀项：

| 类型 | 代码 | 说明 | 目标 |
|------|------|------|------|
| 流程缺陷 | **A** | 流程/规则/检查项遗漏 | SKILL.md rules / Phase 检查项 |
| 检查遗漏 | **B** | Phase 2.5/4.5/5.5 检查项不足 | 对应 Phase 的检查清单 |
| 代码模式 | **C** | 代码规范、反模式、最佳实践 | references/ 或 Memory |
| 项目惯例 | **D** | 设计决策、组件约束、约定 | Memory/feedback |
| 技能可用性 | **E** | Phase 文档可读性/完整性 | Phase 文档本身 |
| 失败案例 | **F** | BUG/设计/流程失败的结构化记录 | `docs/失败案例/FAILURE-INDEX.md`

### Step 4: 执行沉淀

按沉淀建议逐一执行：

#### 4.1 更新技能文件

根据沉淀项类型更新目标文件：

- **类型 A**: 在 SKILL.md 新增/修改强制执行规则，在对应 Phase 文档新增流程步骤
- **类型 B**: 在 Phase 2.5/4.5/5.5 的自检清单或检查表中新增项
- **类型 C**: 更新 references/ 下对应文件，或更新 Memory/feedback
- **类型 D**: 更新 Memory/feedback
- **类型 E**: 更新对应 Phase 文档的描述/步骤/示例

#### 4.2 输出复盘文档

按模板生成 `docs/复盘记录/YYYY-MM-DD-{简述}.md`，记录完整复盘分析。

#### 4.3 更新 INDEX（Read `scripts/update-index.md`）
- prepend mode: 在 `docs/复盘记录/INDEX.md` 表顶部插入新行

#### 4.4 更新 CHANGELOG

在 CHANGELOG 新增复盘驱动变更记录，包含：
- 版本号（递增）
- 变更类型（复盘驱动优化）
- 变更文件清单
- 复盘背景简要说明

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 如有决策日志产生，更新 `decisions[]` 数组引用
2. 更新 `artifacts.retrospect.path`, `artifacts.retrospect.triggered = true`
3. 参数: current_phase="retrospect", next_phase="rule-sync"

#### 4.6 Memory 同步

1. 读取复盘文档的"五维分析"结果
2. 判断哪些维度产出需要同步到 CodeBuddy MEMORY.md：
   - 项目惯例 → 创建 `{project-memory-path}/memory/feedback_{简述}.md`
   - 流程改进 → 更新对应的 Phase 文档和 SKILL.md
   - 技术沉淀 → 创建 `{project-memory-path}/memory/project_{简述}.md`
3. 更新 MEMORY.md 索引行（添加到顶部）
4. 在复盘文档中注明 "Memory 已同步: {文件列表}"

#### 4.7 失败案例归档

1. 检查 Step 2.3 创建的失败案例文件
2. 确保文件存储在 `docs/失败案例/{YYYY}/FC-{YYYYMM}-{XXX}.md`
3. 验证 `FAILURE-INDEX.md` 已追加新行
4. 在复盘文档末尾注明 "失败案例已归档: FC-{ID}"

### Step 5: 汇总检查（跨复盘趋势分析）

> 每次复盘后检查近 3 次复盘记录，发现重复模式自动触发增强沉淀。
> 这是"定期健康度检查"（`/skill-health-check`）在每次复盘的轻量版前置检查。

#### 5.1 读取历史复盘记录

1. 读取 `docs/复盘记录/INDEX.md`，获取最近 3 次复盘记录的文件路径
2. 读取每个复盘记录的"五维分析"结果和"失败案例"引用
3. 排除本次复盘自身

#### 5.2 重复模式检测

| 检测项 | 判断标准 | 触发行动 |
|--------|---------|---------|
| 相同流程缺陷出现 2+ 次 | 同类 Phase 拦截失败 | 自动触发类型 A 增强沉淀 |
| 相同代码反模式出现 2+ 次 | 同类代码质量问题 | 更新 references/ 增加示例 |
| 相同技能缺陷被多次指出 | 同一 Phase 文档问题 | 优先更新对应 Phase 文档 |
| 相同领域失败案例出现 2+ 次 | domain 字段匹配 | 生成"已知风险"专章 |
| 同一 Phase 门控失败 3+ 次 | Phase 锁定 | 建议运行 `/skill-health-check` |

#### 5.3 生成汇总洞察

如发现重复模式，生成汇总洞察追加到复盘记录末尾：

```markdown
## 汇总洞察（跨复盘）
| 重复模式 | 出现次数 | 涉及复盘 | 建议行动 |
|---------|:--------:|---------|---------|
| {模式描述} | {N} 次 | {复盘文件列表} | {增强沉淀类型} |
```

#### 5.4 更新改进追踪

如检测到重复模式，追加到 `proccess_improvements.yaml`：

```yaml
- id: IMP-{自动递增}
  date: "{YYYY-MM-DD}"
  source: "retrospect"
  issue: "汇总检查发现重复模式: {描述}"
  type: "recurring"            # 新类型：重复性问题
  file: "{相关文件}"
  fix: "{建议修复}"
  status: "open"               # 汇总发现的重复模式默认 open
```

---

## 复盘类型判定表

| 触发条件 | 复盘类型 | 必填维度 | 可省略维度 |
|---------|---------|---------|-----------|
| 本次为 BUG 修复 | BUG 复盘 | 五维全部（根因必填） | 无 |
| Phase 5.5 有 ⚠️ 建议项 | 代码审核复盘 | 流程缺陷、代码质量、项目惯例 | 技能可用性（可选） |
| Phase 2.5/4.5/5.5 门控失败重试 | 质量门控复盘 | 流程缺陷、技能可用性 | 代码质量、根因 |
| 新增设计模式/约定 | 惯例沉淀复盘 | 项目惯例 | 流程缺陷、根因 |
| Phase 6.6 审计未归档≥5 | 审计驱动复盘 | 流程缺陷、代码质量、技能可用性 | 根因 |

---

## 输出模板

```markdown
## 复盘记录（Phase 6.7）
| 维度 | 分析结果 | 沉淀项 |
|------|---------|:------:|
| 流程缺陷 | {分析} | {S01} |
| 代码质量 | {分析} | {S02} |
| 项目惯例 | {分析} | {S03} |
| 技能可用性 | {分析} | {S04} |
| 根因 | {5 Why 结论（如有）} | {S05} |

## 失败案例
| ID | 领域 | 严重度 | 文件 |
|----|:----:|:------:|------|
| FC-{ID} | {领域} | {严重度} | `docs/失败案例/{YYYY}/FC-{ID}.md` |

## 汇总洞察（如有跨复盘重复模式）
| 重复模式 | 出现次数 | 涉及复盘 | 建议行动 |
|---------|:--------:|---------|---------|
| {模式} | {N} | {文件} | {行动} |

---

> Memory 已同步: {文件列表}
> 失败案例已归档: {文件列表}
```

---

## 完整性自检

- [ ] 素材收集完成（修改记录/门控/审核/审计/历史复盘）
- [ ] 五维分析完成（流程缺陷/代码质量/项目惯例/技能可用性/根因）
- [ ] 失败案例结构化完成（如需创建：已关联复盘记录、已更新索引）
- [ ] instinct 提取完成（模式识别 + 写入文件 + 更新索引）
- [ ] 沉淀建议已生成（至少 1 项，含类型 F 失败案例评估）
- [ ] 技能文件已更新（规则/检查项/步骤）
- [ ] 复盘文档已输出到 `docs/复盘记录/`
- [ ] 失败案例已归档到 `docs/失败案例/`（如需创建）
- [ ] INDEX.md 已更新（复盘记录 & 失败案例）
- [ ] 汇总检查已完成（近 3 次复盘趋势分析）
- [ ] Memory/feedback 已同步（如涉及项目惯例）
- [ ] proccess_improvements.yaml 已更新（如汇总检查发现重复模式）
- [ ] CHANGELOG 已记录本次复盘驱动变更
- [ ] 复盘分析已遵循 communication-rules 的 concise 级别

## 约束

- 不修改代码（代码问题应在 Phase 5.5 退回 Phase 5）
- 复盘文档不修改，只追加（历史复盘不可变）
- 技能文件更新必须标注版本号和变更日期
- 复盘不可省略"沉淀"环节（即使无改进项，也需输出"本次无改进项"的确认）
- Memory/feedback 更新使用 `Write` 工具创建独立文件，不修改已有记忆中的内容
- instinct 文件使用 YAML 格式，严格遵守 `references/instinct-reference.md` 定义的结构
- instinct 不覆盖已存在文件，如有同 id 则追加 evidence 和增加置信度
- 失败案例文件不可删除，只能标注"已解决"（历史不可变）
- 汇总检查发现的重复模式必须记录到 `proccess_improvements.yaml`，类型为 `recurring`
- **沟通规则**：复盘分析使用 **concise 级别**（每条分析一行概括、直奔根因、不铺垫）。详见 `references/communication-rules.md`。
