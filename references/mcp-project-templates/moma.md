---
name: fullstack-flow/references/mcp-project-templates/moma
description: "moma（墨码）项目 MCP 配置 — 单体 Java Spring Boot 项目，基于 GENERIC 通用模板"
version: 1.0.0
tags: [fullstack, mcp, project, moma]
---

# MCP 配置：moma（墨码）

> 本文档记录 moma 项目的 MCP 服务配置。
> 通用配置模板见 [GENERIC.md](./GENERIC.md)，本文件基于该模板生成。

## 项目信息

- **项目名**: moma（墨码）
- **技术栈**: 纯后端（Java Spring Boot + LangChain4j）
- **项目类型**: 单体项目（无子模块）
- **MCP 配置位置**: `.mcp.json`

## 项目类型判定

| 特征 | 值 | 依据 |
|------|-----|------|
| 技术栈 | Java 21 + Spring Boot | pom.xml 配置 |
| 类型 | 纯后端 | 无 package.json |
| 模块 | 单体 | 单模块项目 |
| MCP 服务端 | 本地 wrapper 脚本 | `mcp-servers/codegraph-wrapper.js` |

根据 GENERIC 模板的[项目类型矩阵](./GENERIC.md#项目类型矩阵)：
- `codegraph` → ✅ 必配（纯后端项目）
- `mysql` → ⚠️ 按需（项目当前无数据库配置）
- `api-fetcher` → ⚠️ 按需（有 Controller，但目前未启用）

## 当前 MCP 配置

### 服务清单

| MCP 服务名 | 类型 | 状态 | 用途 |
|-----------|:----:|:----:|------|
| `codegraph` | stdio | ✅ 启用 | 代码结构分析、调用链追踪、影响分析 |

### 配置详情

```json
{
  "mcpServers": {
    "codegraph": {
      "timeout": 120000,
      "command": "node",
      "args": [
        "d:/AiModel/MoMa/moma/mcp-servers/codegraph-wrapper.js",
        "serve",
        "--mcp",
        "--no-watch",
        "-p",
        "d:/AiModel/MoMa/moma"
      ],
      "type": "stdio",
      "disabled": false,
      "description": "[moma] 代码结构搜索 + 调用链 + 影响分析"
    }
  }
}
```

### 配置说明

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 命令 | `node` | 使用本地 Node.js 运行 wrapper 脚本 |
| 脚本 | `mcp-servers/codegraph-wrapper.js` | 本地 MCP 服务端包装器 |
| 扫描路径 | `d:/AiModel/MoMa/moma` | 项目根目录 |
| 超时 | 120000ms | 2 分钟超时 |

> **与 GENERIC 模板的差异**：GENERIC 模板推荐 `npx @tencent-ai/codegraph-mcp`，但 moma 项目已有本地 `codegraph-wrapper.js`。两者功能等价，使用本地脚本可避免每次 npx 下载。

## 按需可添加的服务

以下服务当前未配置，根据 GENERIC 模板建议，需要时可添加：

### mysql（数据库查询）

如果后续项目引入数据库，按 GENERIC 模板添加：

```json
"mysql-moma": {
  "type": "stdio",
  "command": "cmd",
  "args": ["/c", "npx", "-y", "@tencent-ai/mysql-mcp"],
  "env": {
    "DB_HOST": "localhost",
    "DB_PORT": "3306",
    "DB_NAME": "moma",
    "DB_USER": "root",
    "DB_PASSWORD": "${DB_PASSWORD}"
  }
}
```

### api-fetcher（API 调用）

如果需要在 Phase 2/3 中采样 API 数据，按 GENERIC 模板添加：

```json
"api-fetcher-moma": {
  "type": "stdio",
  "command": "cmd",
  "args": ["/c", "npx", "-y", "@tencent-ai/api-fetcher-mcp"],
  "env": {
    "BASE_URL": "http://localhost:8080"
  }
}
```

> **注意**：添加后需将服务名从 `disabledMcpServers` 移出或设置 `"disabled": false`。

## GENERIC 模板符合度

| 检查项 | GENERIC 要求 | moma 现状 | 状态 |
|--------|-------------|-----------|:----:|
| codegraph 命名规范 | `codegraph` 或 `codegraph-{module}` | `codegraph` ✅ | ✅ |
| mysql 命名规范 | `mysql-{module}` | 未配置（无数据库） | ⚠️ 按需 |
| api-fetcher 命名规范 | `api-fetcher-{module}` | 未配置 | ⚠️ 按需 |
| 安全凭证管理 | 密码不硬编码 | 无密码配置 | ✅ 无风险 |
| disabledMcpServers | 默认禁用非必需服务 | 无不活跃服务 | ✅ |

## 相关参考

- [通用 MCP 配置模板](./GENERIC.md)
- [MCP 工具汇总（通用指引）](../mcp-tools-summary.md)
- [codegraph 使用指南](../codegraph-reference.md)
