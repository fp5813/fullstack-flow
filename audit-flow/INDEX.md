# fullstack-flow 流程审计

**技能路径**: `fullstack-flow/audit-flow`  
**用途**: 标准化全量审核 fullstack-flow 流程 + 项目配置，低风险自动修复

## 触发方式

- 对话中输入 `/fullstack-flow/audit-flow`
- 描述"审核 fullstack-flow" 或 "审计流程"

## 审计内容（20 项）

| 维度 | 检查内容 |
|------|---------|
| 注册完整性 | Phase/子技能/References 全部注册到 SKILL.md |
| 引用一致性 | 跨文件引用路径有效 |
| 无硬编码 | 无旧项目路径/本地绝对路径 |
| MCP 一致性 | 服务名与实际配置一致 |
| 状态一致性 | state.yaml 与 schema 一致 |
| 文档完整性 | 7 个 INDEX + TEMPLATE + 决策日志 + metrics 索引 |
| 版本一致性 | version 格式和 CHANGELOG 覆盖 |

## 输出

- 标准审计报告（8 维度表格）
- 自动修复记录
- 改进追踪追加到 `proccess_improvements.yaml`
