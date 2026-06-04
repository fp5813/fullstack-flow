# fullstack-flow

全栈开发流程 — 质量门控驱动的规范开发（探路→规格→计划→代码→记录→沉淀）

## 概述

`fullstack-flow` 是一套 CodeBuddy Code 技能体系，将开发任务通过 12 个 Phase 标准化执行，确保每次修改有探路、有规格、有计划、有审核、有记录、有复盘。

**核心理念**：写代码前先过门控。每一步有制品、有检查、有记录，避免 AI 直接生成导致的返工和遗漏。

## 安装

将本仓库克隆到项目的 `.codebuddy/skills/fullstack-flow/` 目录：

```bash
cd your-project
mkdir -p .codebuddy/skills
git clone https://github.com/fp5813/fullstack-flow.git .codebuddy/skills/fullstack-flow
```

## 使用

在 CodeBuddy Code 会话中输入：

```
/fullstack-flow
```

CodeBuddy 将加载技能并启动 Phase 1（描述澄清），引导你完成整个开发流程。

## 流程速览

| Phase | 说明 | 门控 |
|-------|------|:--:|
| **Phase 1** 描述澄清 | 多轮 Q&A，提取探路关键词 | Gate 0 |
| **Phase 2** 代码探路 | 4 并行 Agent，生成结构化探路报告 | — |
| **Phase 2.5** 质量门控 | 35 项完整性检查 | Gate 1: 100% |
| **Phase 3** 规格澄清 | What/Goal/Scope/AC | — |
| **Phase 4** 实施计划 | 分解为标准化任务清单 | — |
| **Phase 4.5** 覆盖验证 | AC/文件/术语/数据追溯 | Gate 2: 100% |
| **Phase 5** 最小修改 | FE/BE 并行，影响范围自检 | — |
| **Phase 5.5** 文档审核 | 探路报告/规格/API 一致性 | Gate 3 |
| **Phase 6** 修改记录 | 前后代码对比 + 回滚方案 | Gate 4 |
| **Phase 6.5** 规则同步 | 业务规则更新 | — |
| **Phase 6.6** 规则审计 | 6 类模式全覆盖扫描 | Gate 5 |
| **Phase 6.7** 复盘回顾 | 五维分析 → 技能改进沉淀 | — |

## 目录结构

```
.
├── SKILL.md              # 主技能定义
├── CHANGELOG.md           # 变更记录
├── phases/                # 12 个 Phase 子技能
│   ├── clarify.md
│   ├── probe.md
│   └── ...
├── references/            # 参考文档
│   ├── spec-driven-development.md
│   ├── codegraph-reference.md
│   └── ...
├── change-record/         # 子技能: 修改记录生成
│   └── java-standards/    # 子技能: Java 编码规范
├── code-explore/          # 子技能: 代码探路
├── onboard-project/       # 子技能: 项目初始化
├── audit-flow/            # 子技能: 流程审计
└── vue-standards/         # 子技能: Vue 编码规范
```

## 依赖

- CodeBuddy Code CLI
- MCP 服务：`codegraph`（代码分析）、`mysql-{子项目名}`（数据库）、`api-fetcher`（API 调用）
- 项目需要符合 Vue3 + Spring Boot 技术栈

## 版本

当前版本：**v17.0.0**

详见 [CHANGELOG.md](CHANGELOG.md)
