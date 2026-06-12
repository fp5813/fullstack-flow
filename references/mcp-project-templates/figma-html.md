---
name: fullstack-flow/references/mcp-project-templates/figma-html
description: "figma-html-master（HTML to Figma 转换工具）项目 MCP 配置 — 纯前端 Node/TypeScript 项目，基于 GENERIC 通用模板"
version: 1.0.0
tags: [fullstack, mcp, project, figma, frontend]
---

# MCP 配置：figma-html-master（HTML to Figma）

> 本文档记录 figma-html-master 项目的 MCP 服务配置。
> 通用配置模板见 [GENERIC.md](./GENERIC.md)，本文件基于该模板生成。

## 项目信息

- **项目名**: figma-html-master（@builder.io/html-to-figma）
- **技术栈**: 纯前端（Node.js + TypeScript + Webpack）
- **项目类型**: 单体项目（无子模块）
- **MCP 配置位置**: 无（建议创建 `.mcp.json`）

## 项目类型判定

| 特征 | 值 | 依据 |
|------|-----|------|
| 技术栈 | Node.js + TypeScript | package.json 配置 |
| 类型 | 纯前端 | 无 pom.xml，无 Spring Boot |
| 构建工具 | Webpack + tsc | package.json scripts |
| 用途 | HTML ↔ Figma 转换 | 浏览器插件 + Node 库 |

根据 GENERIC 模板的[项目类型矩阵](./GENERIC.md#项目类型矩阵)：
- `codegraph` → ❌ 不适用（纯前端项目，非 Java）
- `mysql` → ❌ 不适用（无后端数据库）
- `api-fetcher` → ❌ 不适用（无后端 API）

## MCP 配置建议

作为纯前端项目，figma-html-master **不需要** Java 项目的 MCP 服务（codegraph/mysql/api-fetcher）。但根据项目开发需要，可按需添加：

### 可选服务

| 服务 | 用途 | 建议 |
|------|------|------|
| `playwright` | 浏览器自动化 E2E 测试 | ⚠️ 可选（Figma 插件测试需要浏览器环境） |
| `context7` | GitHub 官方文档检索 | ⚠️ 可选（查阅 Builder.io 文档） |

### playwright 配置模板

如果需要在 Figma 插件开发中做浏览器自动化测试：

```json
{
  "mcpServers": {
    "playwright": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@playwright/mcp"]
    }
  }
}
```

### kb-search 配置模板

如果项目有本地知识库：

```json
{
  "mcpServers": {
    "kb-search": {
      "type": "stdio",
      "command": "python",
      "args": ["{knowledge-base-path}/scripts/kb_mcp_server.py"]
    }
  }
}
```

## 与 GENERIC 模板的差异

| 对比项 | GENERIC 模板 | figma-html 项目 | 说明 |
|--------|-------------|----------------|------|
| 项目类型 | 全栈/纯后端 | **纯前端** | GENERIC 模板的矩阵中"纯前端"标记为 ❌ 不适用 |
| codegraph | ✅ 必配 | ❌ 不适用 | 纯前端项目无需代码结构分析 MCP |
| mysql | ✅ 按需 | ❌ 不适用 | 无后端数据库 |
| api-fetcher | ✅ 按需 | ❌ 不适用 | 无后端 API |
| playwright | ⚠️ 可选 | ⚠️ 可选 | 可用于 Figma 插件 E2E 测试 |

> **结论**：figma-html-master 是 GENERIC 模板项目类型矩阵中的"纯前端"类型，不需要 Java 项目的 MCP 服务。
> 如果项目未来引入后端，可参照 GENERIC 模板补充配置。

## 相关参考

- [通用 MCP 配置模板](./GENERIC.md)
- [MCP 工具汇总（通用指引）](../mcp-tools-summary.md)
