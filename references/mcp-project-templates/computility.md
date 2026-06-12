---
name: fullstack-flow/references/mcp-project-templates/computility
description: "computility（算力平台）项目 MCP 配置 — 独立于通用指引的项目级配置"
version: 1.0.0
tags: [fullstack, mcp, project, computility]
---

# MCP 配置：computility（算力平台）

> 本文档记录 computility 项目的 MCP 服务配置。通用使用指引见 `../mcp-tools-summary.md`。

## 项目信息

- **项目名**: computility（算力平台）
- **技术栈**: 全栈（Java Spring Boot + Vue 3）
- **子模块**: identity-check, computility-manage, computility-framework, example-service, example-simple
- **MCP 配置位置**: `.mcp.json`

## MCP 服务清单

### codegraph（代码结构分析）

| MCP 服务名 | 扫描范围 | 默认状态 | 适用场景 |
|-----------|---------|:-------:|---------|
| `codegraph` | 全项目 computility 根目录 | ✅ 启用 | 跨模块调用链、框架→业务追踪 |
| `codegraph-identity-check` | `identity-check/` | ✅ 启用 | identity-check 业务代码分析 |
| `codegraph-computility-manage` | `computility-manage/` | ✅ 启用 | 管理后台业务代码分析 |
| `codegraph-framework` | `computility-framework/` | ⛔ 禁用 | 框架层代码开发 |
| `codegraph-example-service` | `example-service/` | ⛔ 禁用 | 示例服务开发 |
| `codegraph-example-simple` | `example-simple/` | ⛔ 禁用 | 综合示例开发 |

### MySQL 数据库

| MCP 服务名 | 数据库 | 默认状态 |
|-----------|-------|:-------:|
| `mysql-identity-check` | `10.229.0.116:15003/computility` | ✅ 启用 |
| `mysql-example-simple` | `10.220.0.118:3306/outsource` | ⛔ 禁用 |

### API Fetcher

| MCP 服务名 | 后端地址 | 默认状态 |
|-----------|---------|:-------:|
| `api-fetcher-identity-check` | `localhost:8080/identity-check` | ✅ 启用 |
| `api-fetcher-computility-manage` | `localhost:8080/computility-manage` | ✅ 启用 |
| `api-fetcher-example-service` | `localhost:8081/service` | ⛔ 禁用 |
| `api-fetcher-example-simple` | `localhost:8080/simple` | ⛔ 禁用 |

### 其他

| MCP 服务名 | 用途 | 默认状态 |
|-----------|------|:-------:|
| `Context7` | GitHub 官方文档检索 | ✅ 启用 |
| `playwright` | 浏览器自动化（仅 macOS/Linux） | 按需 |
