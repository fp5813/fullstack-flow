---
name: fullstack-flow/onboard-project
description: 接入新的子项目（fullstack-flow 子技能）— 为新子项目创建 MCP 服务配置（codegraph/mysql/api-fetcher），更新技能引用文档，适配 fullstack-flow 开发流程。
allowed-tools:
  - Read
  - Write
  - Edit
  - Grep
  - Glob
  - Agent
user-invocable: true
---

# 接入新的子项目

> 当项目新增子项目（新的 Spring Boot 应用）时，使用本子技能初始化 MCP 服务和 fullstack-flow 配置适配。

## 快速概览

```
创建 MCP 服务配置（codegraph/mysql/api-fetcher）
  → 配置 MCP 服务名模式 → 适配 fullstack-flow 引用
  → 验证配置生效 → 完成
```

**核心**: 为新子项目创建 MCP 配置 + 更新 fullstack-flow 子项目引用

## 输入参数

| 参数 | 说明 | 示例 |
|------|------|------|
| **子项目名** | 用于 MCP 服务命名的简短标识 | `identity-check` |
| **子项目路径** | 相对于项目根目录的路径 | `identity-check` |
| **server.port** | Spring Boot 启动端口 | `8080` |
| **context-path** | 应用上下文路径 | `/identity-check` |
| **是否有数据库** | true / false | `true` |
| **数据库 host** | MySQL 主机地址 | `10.229.0.116` |
| **数据库 port** | MySQL 端口 | `15003` |
| **数据库名** | database 名称 | `computility` |
| **数据库用户** | 用户名 | `computility` |
| **数据库密码** | 密码 | `Aa123456!@#` |

> 数据库信息可选（仅当有数据库时填写）。

## 执行流程

### Step 1: 获取项目信息

交互式收集上述输入参数。同时收集**项目根目录**和**父 POM 路径**：
- 项目根目录（`{project-root}`）— 代码仓库的绝对路径
- 子项目路径 — 相对于项目根目录的路径
- 父 POM 路径 — 注册了 `<modules>` 的 `pom.xml` 文件路径
- 确认子项目路径在父 POM 的 `<modules>` 中已注册

### Step 2: 更新 `.mcp.json` — 添加 codegraph 服务

在 `.mcp.json` 的 `mcpServers` 中添加。`{mcp-servers-path}` 为 MCP 服务端脚本所在目录，默认为 `{project-root}/mcp-servers/`，如果已有共用的 codegraph-wrapper.js 则填写实际路径：

```jsonc
"codegraph-{子项目名}": {
  "timeout": 120000,
  "command": "node",
  "args": [
    "{mcp-servers-path}/codegraph-wrapper.js",
    "serve",
    "--mcp",
    "--no-watch",
    "-p",
    "{project-root}/{子项目路径}"
  ],
  "type": "stdio",
  "disabled": true,
  "description": "[{子项目名}] 代码结构搜索 + 调用链 + 影响分析"
}
```

> 新接入的子项目默认 `disabled: true`，需要时手动启用。

### Step 3: 更新 `.mcp.json` — 添加 mysql 服务（可选）

如果子项目有数据库：

```jsonc
"mysql-{子项目名}": {
  "timeout": 60000,
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "mysql-mcp-server"],
  "env": {
    "MYSQL_HOST": "{host}",
    "MYSQL_PORT": "{port}",
    "MYSQL_USER": "{user}",
    "MYSQL_PASSWORD": "{password}",
    "MYSQL_DATABASE": "{database}"
  },
  "disabled": true
}
```

### Step 4: 更新 `.mcp.json` — 添加 api-fetcher 服务（可选）

如果是 Spring Boot Web 应用（有 Controller），且非 example-service（无 DB 但仍有 API）：

```jsonc
"api-fetcher-{子项目名}": {
  "command": "node",
  "args": [
    "{mcp-servers-path}/api-fetcher/index.js",
    "--base-url=http://127.0.0.1:{port}{context-path}"
  ],
  "type": "stdio",
  "disabled": true
}
```

### Step 5: 更新 MCP 工具汇总文档

修改 `.codebuddy/skills/fullstack-flow/references/mcp-tools-summary.md`：

- `codegraph` 表格：添加新行 `| codegraph-{name} | {path} | ⛔ 禁用 | {desc} |`
- `MySQL 数据库` 表格：如有 DB 则添加 `| mysql-{name} | {host}:{port}/{database} | ⛔ 禁用 |`
- `API Fetcher` 表格：如有 API 则添加 `| api-fetcher-{name} | localhost:{port}{context-path} | ⛔ 禁用 |`

### Step 6: 验证配置完整性

- [ ] `.mcp.json` JSON 格式有效（无语法错误）
- [ ] `codegraph-{name}` 已添加，`-p` 路径指向正确子项目目录
- [ ] `mysql-{name}` 已添加（如有 DB），env 参数完整
- [ ] `api-fetcher-{name}` 已添加（如为 Web 应用），`--base-url` 格式正确
- [ ] `mcp-tools-summary.md` 已更新，新增行格式与原有一致
- [ ] 各服务初始状态均为 `disabled: true`（安全默认）

### Step 7: 记录决策

在 `.codebuddy/workflow/decisions/` 下创建决策记录：

```markdown
## 决策：接入子项目 {name}

**日期**：YYYY-MM-DD
**类型**：项目初始化
**详情**：
- 子项目路径：`{path}`
- context-path：`{context-path}`
- port：{port}
- 数据库：{有/无}
```

## 约束

- 新接入的子项目 MCP 服务默认 `disabled: true`，避免启动过多进程
- 子项目路径必须已在父 POM 的 `<modules>` 中注册
- codegraph 的 `-p` 路径使用绝对路径（`{project-root}/{子项目路径}`）
- 不修改已有的 MCP 服务配置，仅追加新条目
- `mcp-tools-summary.md` 中新增行保持与原表格格式对齐（`|` 分隔）

## 引用

- [MCP 配置文件]（`{project-root}/.mcp.json`）
- [MCP 工具汇总]（`.codebuddy/skills/fullstack-flow/references/mcp-tools-summary.md`）
- [通用 MCP 配置模板]（`../references/mcp-project-templates/GENERIC.md`）— 推荐新项目阅读
- [codegraph-wrapper.js]（`{mcp-servers-path}/codegraph-wrapper.js`）
- [api-fetcher]（`{mcp-servers-path}/api-fetcher/`）
