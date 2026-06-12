---
name: fullstack-flow/references/mcp-project-templates/GENERIC
description: "通用 MCP 配置模板 — 覆盖全栈/后端/前端项目的标准化 MCP 服务配置，含命名规范、启停规则、安全凭证管理和多环境配置"
version: 1.0.0
tags: [fullstack, mcp, template, generic, project]
---

# 通用 MCP 配置模板

> 任何项目接入 fullstack-flow 时，按此模板配置 MCP 服务。
> 通用使用指引见 `../mcp-tools-summary.md`。

## 项目类型矩阵

| 类型 | codegraph | mysql | api-fetcher | 适用场景 |
|:----:|:---------:|:-----:|:-----------:|---------|
| **全栈** | ✅ 必配 | ✅ 按需 | ✅ 按需 | Java Spring Boot + Vue 3 项目 |
| **纯后端** | ✅ 必配 | ✅ 按需 | ✅ 按需 | 仅有 Java 后端服务 |
| **纯前端** | ❌ 不适用 | ❌ 不适用 | ❌ 不适用 | 仅有 Vue/React 前端项目（无需 MCP） |

> **全栈** 为默认推荐配置。以下模板以全栈项目为例。
> 实际项目配置示例见 `computility.md`（多模块）和 `archive.md`（单体）。

## 标准化服务命名

| 服务类型 | 命名模式 | 必配 | 用途 |
|---------|---------|:----:|------|
| codegraph（全项目） | `codegraph` | ✅ | 跨模块代码结构搜索、调用链追踪、影响分析 |
| codegraph（子模块） | `codegraph-{module}` | 按需 | 子模块聚焦分析，更快更精准 |
| mysql | `mysql-{module}` | 按需 | 数据库表结构和数据查询 |
| api-fetcher | `api-fetcher-{module}` | 按需 | 后端 API 调用/数据采样 |

> `{module}` 替换为子模块名（如 `identity-check`、`computility-manage`）。
> 单体项目无需 `codegraph-{module}`，只配 `codegraph` 即可。

## 快速开始（复制即用）

> 不想读全文？以下是最简配置，复制到 `.mcp.json` 替换 `{}` 占位符即可。

```json
{
  "mcpServers": {
    "codegraph": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@tencent-ai/codegraph-mcp"]
    }
  },
  "disabledMcpServers": []
}
```

**后续按需添加**（从 `disabledMcpServers` 移到 `mcpServers`）：
| 场景 | 添加的服务 |
|------|-----------|
| 需要查数据库 | `mysql-{module}` |
| 需要调 API | `api-fetcher-{module}` |
| 多模块项目 | `codegraph-{module}`（每个模块一个） |

## 通用 .mcp.json 模板

```json
{
  "mcpServers": {
    "codegraph": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@tencent-ai/codegraph-mcp"]
    },
    "codegraph-{module}": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@tencent-ai/codegraph-mcp", "--scan-path", "{project-root}/{module}"]
    },
    "mysql-{module}": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@tencent-ai/mysql-mcp"],
      "env": {
        "DB_HOST": "{host}",
        "DB_PORT": "{port}",
        "DB_NAME": "{database}",
        "DB_USER": "{username}",
        "DB_PASSWORD": "{password}"
      }
    },
    "api-fetcher-{module}": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@tencent-ai/api-fetcher-mcp"],
      "env": {
        "BASE_URL": "http://localhost:{port}/{context-path}"
      }
    },
    "kb-search": {
      "type": "stdio",
      "command": "python",
      "args": ["{knowledge-base-path}/scripts/kb_mcp_server.py"]
    },
    "playwright": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@playwright/mcp"]
    },
    "context7": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "@upstash/context7-mcp"]
    }
  },
  "disabledMcpServers": [
    "api-fetcher-{module}",
    "mysql-{module}",
    "kb-search",
    "playwright"
  ]
}
```

> **注意**：实际项目配置需要将 `{module}`、`{project-root}`、`{host}`、`{port}`、`{database}`、`{username}`、`{password}`、`{context-path}` 替换为真实值。
> 参考示例：多模块项目见 `computility.md`，单体项目见 `archive.md`。

## 配置说明

### codegraph

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 命令 | `npx -y @tencent-ai/codegraph-mcp` | 通用安装命令 |
| `--scan-path` | 子模块路径 | 仅 `codegraph-{module}` 需要，指定扫描范围 |
| 无 env | — | codegraph 无需额外环境变量 |

**启用规则**：
- 全栈/纯后端项目：`codegraph` 始终启用
- 多模块项目：每个活跃子模块添加 `codegraph-{module}`
- 单体项目：仅需 `codegraph`

### mysql

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 命令 | `npx -y @tencent-ai/mysql-mcp` | 通用安装命令 |
| `DB_HOST` | 数据库主机地址 | 如 `10.229.0.116` |
| `DB_PORT` | 数据库端口 | 如 `15003`、`3306` |
| `DB_NAME` | 数据库名 | 如 `computility` |
| `DB_USER` | 数据库用户 | 如 `computility` |
| `DB_PASSWORD` | 数据库密码 | 从安全凭证管理获取 |

**启用规则**：
- 有数据库且需要 Phase 2 数据采样时：启用
- 纯前端/无数据库项目：禁用
- 默认加入 `disabledMcpServers`，按需启用

**安全注意事项**：
- `DB_PASSWORD` 勿提交到版本控制
- 建议通过环境变量注入，而非硬编码在 `.mcp.json`
- 生产数据库禁止配置到此服务（仅开发/测试库）

### api-fetcher

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 命令 | `npx -y @tencent-ai/api-fetcher-mcp` | 通用安装命令 |
| `BASE_URL` | 后端 API 基地址 | 如 `http://localhost:8080/{context-path}` |
| 端口 | 对应 Spring Boot `server.port` | 如 `8080`、`8081` |

**启用规则**：
- 需要 Phase 2/3 数据采样时：启用
- 纯前端项目：禁用
- 默认加入 `disabledMcpServers`，按需启用

**前提条件**：
- 后端服务必须在本地运行（`mvn spring-boot:run` 或 IDE 启动）
- 服务启动后才能调用 API

### 可选服务

| 服务 | 用途 | 启用条件 |
|------|------|---------|
| `kb-search` | 项目知识库搜索 | 存在 `.knowledge-base/` 目录时 |
| `playwright` | 浏览器自动化 E2E 测试 | 需要 Phase 5.5 UI 验证时（仅 macOS/Linux） |
| `context7` | GitHub 官方文档检索 | 需要查阅开源库文档时 |

## 多环境配置

```json
{
  "mcpServers": {
    "mysql-{module}": {
      "env": {
        "DB_HOST": "${DB_HOST}",      // 从系统环境变量读取
        "DB_PORT": "${DB_PORT}",
        "DB_NAME": "${DB_NAME}",
        "DB_USER": "${DB_USER}",
        "DB_PASSWORD": "${DB_PASSWORD}"
      }
    }
  }
}
```

**推荐做法**：
1. 在 `.mcp.json` 中使用 `${VAR}` 占位符引用环境变量
2. 在 `.env` 或系统环境变量中设置实际值
3. 不同环境使用不同的 `.env` 文件（`.env.dev`、`.env.test`）
4. `.env` 文件加入 `.gitignore`，避免泄露凭证

## 安全凭证管理

| 敏感信息 | 存储位置 | 禁止操作 |
|---------|---------|---------|
| 数据库密码 | 环境变量 / `.env`（已 gitignore） | 硬编码在 `.mcp.json` |
| API Token | 环境变量 / 运行时注入 | 硬编码在配置中 |
| 生产环境地址 | 仅在 CI/CD 中配置 | 配置到本地 `.mcp.json` |

**安全检查清单**：
- [ ] `.mcp.json` 不含明文密码
- [ ] `.env` 文件已加入 `.gitignore`
- [ ] 生产数据库地址未出现在配置中
- [ ] 仅开发/测试数据库可通过 MCP 访问

## 启停规则速查

| 服务 | 启动条件 | 停止条件 | 状态检查 |
|------|---------|---------|---------|
| codegraph | 项目初始化时自动启用 | — | 调用 `codegraph_search` 验证 |
| mysql | Phase 2 数据采样前按需启用 | 数据采样完成后 | 调用 `execute_query` 验证 |
| api-fetcher | Phase 2/3 数据采样前按需启用 | 数据采样完成后 | 调用 `api_list` 验证 |
| playwright | Phase 5.5 E2E 验证前按需启用 | 验证完成后 | 调用 `browser_navigate` 验证 |

## 与 fullstack-flow 集成

| Phase | 依赖的 MCP 服务 | 说明 |
|-------|----------------|------|
| Phase 1 clarify | — | 仅环境预检，检查服务就绪状态 |
| Phase 2 probe | `codegraph` / `codegraph-{module}` / `mysql-{module}` | 代码探路和数据采样 |
| Phase 3 spec | `api-fetcher-{module}`（可选） | API 数据采样验证 VO/DTO |
| Phase 5 code | — | 编译/测试，不依赖 MCP |
| Phase 5.5 review | `api-fetcher-{module}` / `playwright` | E2E 验证 |
| Phase 6.6 audit | `codegraph` | 业务规则模式扫描 |

> 新增子项目时，可使用 `/fullstack-flow/onboard-project` 子技能自动生成 MCP 配置。

## 故障排查

| 症状 | 可能原因 | 解决方法 |
|------|---------|---------|
| codegraph 返回空 | `--scan-path` 路径错误 | 检查路径是否指向源码目录 |
| mysql 连接失败 | 主机/端口/凭证错误 | 验证 DB_HOST/DB_PORT/DB_PASSWORD |
| mysql 连接失败 | 数据库未运行 | 检查 MySQL 服务状态 |
| api-fetcher 超时 | 后端服务未启动 | 运行 `mvn spring-boot:run` |
| api-fetcher 403 | 未登录/Token 过期 | 先调用 `api_login` |
| playwright 不可用 | Windows 不支持 | 仅在 macOS/Linux 下使用 |

## 相关参考

- [MCP 工具汇总（通用指引）](../mcp-tools-summary.md)
- [codegraph 使用指南](../codegraph-reference.md)
- [api-fetcher 使用指南](../api-fetcher-reference.md)
- [项目初始化模板](./TEMPLATE.md)
- [多模块项目配置示例](./computility.md)
- [单体项目配置示例](./archive.md)
- [子项目接入流程](../../onboard-project/SKILL.md) — 自动生成 MCP 配置
