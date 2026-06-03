# MCP 工具汇总

> 本项目（computility 算力平台）包含多个子项目，MCP 服务按子项目命名隔离。
> 开发流程中涉及哪个子项目，就使用对应子项目的 MCP 服务。

## 子项目 MCP 映射

### codegraph（代码结构分析）

| MCP 服务名 | 扫描范围 | 默认状态 | 适用场景 |
|-----------|---------|---------|---------|
| `codegraph` | **全项目** computility 根目录 | ✅ 启用 | 跨模块调用链、框架→业务追踪 |
| `codegraph-identity-check` | `identity-check/` | ✅ 启用 | identity-check 业务代码分析 |
| `codegraph-framework` | `computility-framework/` | ⛔ 禁用 | 框架层代码开发 |
| `codegraph-example-service` | `example-service/` | ⛔ 禁用 | 示例服务开发 |
| `codegraph-example-simple` | `example-simple/` | ⛔ 禁用 | 综合示例开发 |

### MySQL 数据库

| MCP 服务名 | 数据库 | 默认状态 |
|-----------|-------|---------|
| `mysql-identity-check` | `10.229.0.116:15003/computility` | ✅ 启用 |
| `mysql-example-simple` | `10.220.0.118:3306/outsource` | ⛔ 禁用 |

### API Fetcher

| MCP 服务名 | 后端地址 | 默认状态 |
|-----------|---------|---------|
| `api-fetcher-identity-check` | `localhost:8080/identity-check` | ✅ 启用 |
| `api-fetcher-example-service` | `localhost:8081/service` | ⛔ 禁用 |
| `api-fetcher-example-simple` | `localhost:8080/simple` | ⛔ 禁用 |

### 其他

| MCP 服务名 | 用途 | 默认状态 |
|-----------|------|---------|
| `Context7` | GitHub 官方文档检索 | ✅ 启用 |
| `playwright` | 浏览器自动化（仅 macOS/Linux） | 按需 |

## 使用原则

1. **全项目搜索** → 用 `codegraph`（跨模块追踪调用链）
2. **子项目聚焦** → 用 `codegraph-{子项目}`（更快更精准）
3. **查数据库** → 用 `mysql-{子项目}` 查询对应数据库表结构
4. **调 API** → 用 `api-fetcher-{子项目}` 获取对应后端接口响应

## 使用优先级

1. **codegraph_context** — 首选，一次调用获取全部上下文
2. **codegraph_node** — 深入关键符号 details + trail
3. **api-fetcher-{subproject}** — 调用对应子项目的 API 获取真实响应数据
4. **mysql-{subproject}** — 查对应数据库表结构

## 详细参考

- [codegraph 使用指南](./codegraph-reference.md)
- [api-fetcher 使用指南](./api-fetcher-reference.md)
