---
name: fullstack-flow/references/mcp-project-templates/TEMPLATE
description: "MCP 项目初始化模板 — 新项目接入 fullstack-flow 时记录 MCP 配置的标准化模板。推荐先阅读 GENERIC.md 通用模板。"
version: 1.1.0
tags: [fullstack, mcp, template, project]
---

# MCP 项目初始化模板

> 新项目接入 fullstack-flow 时，按此模板记录 MCP 配置。
> **推荐先阅读 [通用 MCP 配置模板](./GENERIC.md)**，了解标准化命名和服务配置规则后再使用本模板。
> 完成后移入 `mcp-project-templates/` 目录。

## 项目信息

- **项目名**: {项目名}
- **技术栈**: {Java Spring Boot / Vue 3 / 全栈}
- **子模块**: {子模块1, 子模块2, ...}
- **MCP 配置位置**: `.mcp.json`

## MCP 服务清单

| 服务类型 | 服务名 | 配置位置 | 用途 | 默认状态 |
|---------|-------|---------|------|:-------:|
| codegraph | `codegraph` | `.mcp.json` | 全项目代码结构搜索/调用链 | ✅ |
| codegraph | `codegraph-{子模块}` | `.mcp.json` | 子模块代码分析 | ✅ |
| mysql | `mysql-{子模块}` | `.mcp.json` | 数据库表结构和数据查询 | ✅ |
| api-fetcher | `api-fetcher-{子模块}` | `.mcp.json` | 后端 API 调用/数据采样 | ✅ |

## 数据库连接

| 服务名 | 主机 | 端口 | 数据库名 | 用途 |
|-------|:----:|:----:|---------|------|
| `mysql-{子模块}` | {host} | {port} | {db_name} | {说明} |

## 使用优先级

1. **codegraph** — 首选，一次调用获取全部上下文
2. **codegraph-{子模块}** — 子模块聚焦，更快更精准
3. **mysql-{子模块}** — 查询数据库表结构
4. **api-fetcher-{子模块}** — 调用 API 获取真实响应数据

## 相关参考

- [MCP 工具汇总（通用指引）](../mcp-tools-summary.md)
- [codegraph 使用指南](../codegraph-reference.md)
- [api-fetcher 使用指南](../api-fetcher-reference.md)
