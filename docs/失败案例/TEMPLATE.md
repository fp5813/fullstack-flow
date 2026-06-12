# 失败案例模板

```yaml
---
id: FC-YYYYMM-XXX
title: "案例标题"
domain: "bug/design/process/security"
severity: "critical/major/minor"
created_at: "YYYY-MM-DD"
source: "retrospect"  # 来源：retrospect / audit-flow / manual
related_retrospect: "docs/复盘记录/YYYY-MM-DD-{简述}.md"
related_modification: "docs/修改记录/{文件名}"
tags: ["关键词1", "关键词2"]
---

## 问题现象

描述问题的具体表现，包括：
- 发生了什么？
- 在什么场景下出现？
- 影响范围是什么？

## 根因分析（5 Why）

| 轮次 | 问题 | 答案 |
|------|------|------|
| Why 1 | {问题表现} | {直接原因} |
| Why 2 | 为什么会有这个直接原因？ | {第二层原因} |
| Why 3 | ... | ... |
| Why 4 | ... | ... |
| Why 5 | 流程/技能/检查项层面的根因 | {根本原因} |

## 未拦截的 Phase

| Phase | 应做什么 | 实际发生了什么 | 缺失了什么 |
|-------|---------|---------------|-----------|
| Phase N | {预期检查项} | {实际行为} | {缺失的检查/规则} |

## 修复方案

1. 短期修复：{本次如何修复}
2. 长期预防：{沉淀到系统的预防措施}

## 沉淀项

| 沉淀类型 | 目标文件 | 变更内容 |
|---------|---------|---------|
| A | SKILL.md | {新增/修改的规则} |
| B | Phase 文档 | {新增的检查项} |
| C/D/E | {其他} | {变更内容} |

## 类似案例

- `FC-YYYYMM-XXX` - {类似案例简述}
- `FC-YYYYMM-XXX` - {类似案例简述}

---

> **预防措施验证**: 下次 audit-flow 或 skill-health-check 运行时自动检查预防措施已实施。
```

## 字段说明

| 字段 | 必填 | 说明 |
|------|:----:|------|
| id | ✅ | 格式 `FC-YYYYMM-XXX`，按月+序号 |
| title | ✅ | 简短描述，30 字以内 |
| domain | ✅ | bug(代码缺陷) / design(设计缺陷) / process(流程漏洞) / security(安全风险) |
| severity | ✅ | critical(阻断性) / major(重要) / minor(轻微) |
| source | ✅ | retrospect(复盘产生) / audit-flow(审计发现) / manual(人工录入) |
| related_retrospect | ✅ | 关联复盘记录路径 |
| tags | 可选 | 便于检索的关键词 |

## 创建规则

1. **触发条件**: 复盘五维分析中，凡出现以下任一情况必须创建失败案例：
   - BUG 修复复盘（必建）
   - 同一类问题第二次出现（必建）
   - 门控连环失败（建议创建）
   - 用户明确要求（必建）
2. **存储位置**: `docs/失败案例/YYYY/FC-YYYYMM-XXX.md`
3. **索引维护**: 创建后更新 `docs/失败案例/FAILURE-INDEX.md`
4. **关联性**: 在复盘文档中注明 `相关失败案例: FC-YYYYMM-XXX`
