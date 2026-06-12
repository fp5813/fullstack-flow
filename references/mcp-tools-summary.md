---
name: fullstack-flow/references/mcp-tools-summary
description: "MCP 工具汇总（通用指引）— 命名模式、使用原则和优先级。项目级配置见 mcp-project-templates/"
version: 2.1.0
tags: [fullstack, mcp, reference, universal]
---

# MCP 工具汇总（通用指引）

> 本文档记录 MCP 服务的通用使用原则和优先级。**项目级具体配置**请查阅 `mcp-project-templates/` 目录下的对应项目文件。

## 通用命名模式

| 服务类型 | 命名模式 | 用途 |
|---------|---------|------|
| codegraph | `codegraph` / `codegraph-{子项目名}` | 代码结构搜索/调用链/影响分析 |
| mysql | `mysql-{子项目名}` | 数据库表结构和数据查询 |
| api-fetcher | `api-fetcher-{子项目名}` | 后端 API 调用/数据采样 |

> 命名模式在所有项目中保持一致，`{子项目名}` 替换为实际子模块名称。

## 使用原则

1. **全项目搜索** → 用 `codegraph`（跨模块追踪调用链）
2. **子项目聚焦** → 用 `codegraph-{子项目}`（更快更精准）
3. **查数据库** → 用 `mysql-{子项目}` 查询对应数据库表结构
4. **调 API** → 用 `api-fetcher-{子项目}` 获取对应后端接口响应

## 使用优先级

1. **codegraph_context** — 首选，一次调用获取全部上下文
2. **codegraph_node** — 深入关键符号细节 + trail
3. **api-fetcher-{subproject}** — 调用对应子项目的 API 获取真实响应数据
4. **mysql-{subproject}** — 查对应数据库表结构

## 项目级配置

各项目的具体 MCP 服务清单、数据库地址和 API 地址在独立文件中维护：

| 项目 | 文件 | 说明 |
|------|------|------|
| **通用模板（推荐）** | [mcp-project-templates/GENERIC.md](./mcp-project-templates/GENERIC.md) | **覆盖全栈/后端项目的标准化 MCP 配置，含命名规范、启停规则、安全凭证管理** |
| moma（墨码） | [mcp-project-templates/moma.md](./mcp-project-templates/moma.md) | 纯后端单体项目，基于 GENERIC 模板生成 |
| figma-html（HTML to Figma） | [mcp-project-templates/figma-html.md](./mcp-project-templates/figma-html.md) | 纯前端 Node/TypeScript 项目，GENERIC 矩阵中的"纯前端"类型 |
| computility（算力平台） | [mcp-project-templates/computility.md](./mcp-project-templates/computility.md) | 含 5 个子模块的 MCP 配置 |
| archive（档案管理系统） | [mcp-project-templates/archive.md](./mcp-project-templates/archive.md) | JeecgBoot 单体项目 MCP 配置 |
| 新项目接入 | [mcp-project-templates/TEMPLATE.md](./mcp-project-templates/TEMPLATE.md) | 新项目 MCP 初始化模板 |

## 详细参考

- [通用 MCP 配置模板](./mcp-project-templates/GENERIC.md) — **推荐所有新项目从此开始**
- [codegraph 使用指南](./codegraph-reference.md)
- [api-fetcher 使用指南](./api-fetcher-reference.md)

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 2.1.0 | 2026-06-12 | 新增通用 MCP 配置模板 GENERIC.md 引用 |
| 2.0.0 | 2026-06-11 | 重构为通用指引+项目级配置分离 |
