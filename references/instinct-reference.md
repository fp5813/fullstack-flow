# Instinct（直觉）机制参考

## 快速参考

| 概念 | 说明 |
|------|------|
| **Instinct** | 从项目执行历史中自动提取的可复用模式 |
| **置信度** | 0.0~1.0，首次 0.60，复现 +0.05~0.10，确认 +0.15 |
| **存储路径** | 项目级 `.codebuddy/instincts/`，全局级 `~/.CODEBUDDY/instincts/` |
| **Promotion** | 置信度 ≥ 0.80 提升为全局 |
| **集成 Phase** | Phase 6.7 提取 / Phase 6.5 领域捕获 |

## 什么是 Instinct？

Instinct 是从项目开发执行历史中自动提取的可复用模式。每次 fullstack-flow 流程走完一个完整的工作流（Phase 1 → ... → Phase 6.7），系统会在复盘回顾阶段分析本次实施中产生的代码、文档和流程数据，识别出值得复用的模式并记录为 Instinct。

## Instinct 生命周期

```
开发执行 → Phase 6.7 复盘 → 模式提取 → Instinct (YAML) → 复用验证
                ↓                          ↓
          置信度 < 阈值             置信度 ≥ 阈值
               ↓                          ↓
        下次执行继续观察            → 提升为全局 Instinct
                                      ↓
                                /evolve → 生成技能/命令
```

## 存储结构

### 项目级

```
.codebuddy/instincts/
├── INDEX.md                  # 索引（含概要统计）
├── personal/                 # 自动学习（项目专属）
│   └── yyyy-MM-{slug}.yaml
└── inherited/                # 导入/继承
    └── yyyy-MM-{source}.yaml
```

### 用户级

```
~/.CODEBUDDY/instincts/
├── INDEX.md                  # 全局索引
├── personal/                 # 自动学习（全局）
└── inherited/                # 导入
```

## Instinct 格式

```yaml
---
id: domain-specific-pattern-id    # 唯一 ID，使用 kebab-case
title: "人类可读标题"
domain: "数据"                     # 领域分类：代码/数据/测试/安全/UX/流程/领域
trigger: "触发条件描述"             # 什么场景下应触发此 instinct
context: "适用上下文"               # 适用的项目技术栈或场景范围
pattern: |                         # 模式描述（多行 Markdown）
  1. 步骤一
  2. 步骤二
  3. 步骤三
confidence: 0.85                   # 置信度 0.0~1.0
strength: "strong"                 # strong / medium / weak
evidence:                          # 证据来源
  - path: "docs/修改记录/2026-06-03-xxx.md"
    summary: "该模式在此修改记录中被验证"
scope: "project"                   # project / global
source: "fullstack-flow"           # 来源流程
created: "2026-06-04"              # 创建日期
updated: "2026-06-04"              # 更新日期
occurrences: 3                     # 出现次数
---
```

### 字段说明

| 字段 | 必填 | 说明 |
|------|:----:|------|
| `id` | ✅ | 唯一标识，`{domain}-{简短描述}` |
| `title` | ✅ | 人类可读标题 |
| `domain` | ✅ | 领域：`代码` `数据` `测试` `安全` `UX` `流程` `领域` |
| `trigger` | ✅ | 何时使用此 instinct |
| `pattern` | ✅ | 具体的模式/步骤描述 |
| `confidence` | ✅ | 0.0~1.0，基于出现的频率和一致性 |
| `scope` | ✅ | `project`（本项目可用）或 `global`（所有项目可用） |
| `evidence` | — | 证明此模式有效的来源文件列表 |
| `context` | — | 什么场景适用（如 "Spring Boot + MyBatis Plus"） |

## 置信度规则

| 条件 | 初始置信度 | 每次复发增加 |
|------|:---------:|:----------:|
| 首次提取 | 0.60 | — |
| 第二次出现（同一 id） | — | +0.10 |
| 每次后续出现 | — | +0.05 |
| 用户手动确认 | — | +0.15 |
| 最大置信度 | — | 1.00 |

## 领域分类

| 领域 | 说明 | 示例 |
|------|------|------|
| `代码` | 代码实现模式 | 错误处理、异步竞态、数据转换 |
| `数据` | 数据操作模式 | 字段映射、VO/DTO 结构、表设计 |
| `测试` | 测试模式 | 测试类结构、Mock 方式、边界值 |
| `安全` | 安全模式 | 权限注解、SQL 注入防护、XSS |
| `UX` | 用户体验模式 | 弹框大小控制、表单回显、加载状态 |
| `流程` | 流程改进模式 | 门控常失败项、探路最佳实践、Phase 优化 |
| `领域` | 业务领域模式 | 状态流转、业务规则、常量定义 |

## Promotion 规则

| 条件 | 动作 |
|------|------|
| 置信度 ≥ 0.80 & scope=project | 建议 promotion 到全局（用户级） |
| 同一 instinct 在 3+ 不同项目中存在 | 自动标记为全局候选 |
| 用户手动执行 `/instinct-promote` | 从项目级复制到用户级 |

## 与 fullstack-flow 的集成

| Phase | 集成点 |
|-------|--------|
| **Phase 6.7 复盘回顾** | 实施完成后提取模式 → 生成 instinct YAML |
| **Phase 6.5 规则同步** | 涉及业务规则变更时提取领域 instinct |
| **Phase 5.5 代码审核** | 审核结果中标记可复用的模式（由 Phase 6.7 统一处理） |

## 使用方式

```
# 查看本项目所有 Instinct
/instinct-status

# 将项目 Instinct 提升为全局
/instinct-promote <id>

# 聚类 Instinct 生成技能/命令
/instinct-evolve [--generate]
```
