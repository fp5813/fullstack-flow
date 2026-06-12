---
name: fullstack-flow/references/mcp-project-templates/archive
description: "archive（档案管理系统）项目 MCP 配置 — 独立于通用指引的项目级配置"
version: 1.1.0
tags: [fullstack, mcp, project, archive]
---

# MCP 配置：archive（档案管理系统）

> 本文档记录 archive 项目的 MCP 服务配置。通用使用指引见 `../mcp-tools-summary.md`。

## 项目信息

- **项目名**: archive（档案管理系统）
- **技术栈**: 全栈（JeecgBoot 单体 + Vue 3）
- **子模块**: 单体项目（archive-server + archive-web 子目录）
- **MCP 配置位置**: `.mcp.json`

## 当前 MCP 配置

| MCP 服务名 | 状态 | 用途 |
|-----------|:----:|------|
| `codegraph` | ✅ 已启用 | 代码结构分析、调用链追踪 |
| `api-fetcher` | ⛔ 已禁用（在 `.mcp.json` 中可通过 `mcpServers` 启用） | 后端 API 调用/数据采样 |

> **说明**: codegraph 已启用，支持 Phase 2 代码探路。api-fetcher 仍处于禁用状态，需在 `.mcp.json` 中从 `disabledMcpServers` 移至 `mcpServers` 方可启用。

## 使用建议

- **代码分析**: 启用 `codegraph` 支持 Phase 2 代码探路（自动调用链追踪）
- **API 采样**: 启用 `api-fetcher` 支持 Phase 3 数据采样（需后端 8080 端口运行）
- **数据库查询**: 项目使用 MySQL/达梦数据库，可添加 `mysql-archive` 服务对应

## 相关参考

- [MCP 工具汇总（通用指引）](../mcp-tools-summary.md)
- [codegraph 使用指南](../codegraph-reference.md)
- [api-fetcher 使用指南](../api-fetcher-reference.md)
