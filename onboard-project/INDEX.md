# 接入新子项目

**技能路径**: `fullstack-flow/onboard-project`  
**用途**: 当项目新增子项目时，自动创建对应的 MCP 服务和适配 fullstack-flow 流程

## 使用方式

在对话中输入 `/fullstack-flow/onboard-project` 或描述"接入新子项目"即可触发。

## 执行结果

| 制品 | 说明 |
|------|------|
| `.mcp.json` 新增条目 | codegraph / mysql / api-fetcher 各 1 条 |
| `mcp-tools-summary.md` | 子项目 MCP 映射表更新 |
| 决策记录 | 记录子项目接入信息 |

## 默认行为

- 新子项目的所有 MCP 服务默认 `disabled: true`
- 需要时通过编辑 `.mcp.json` 将 `disabled` 改为 `false`
- 新子项目必须在父 POM 的 `<modules>` 中已注册
