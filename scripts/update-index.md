---
name: fullstack-flow/scripts/update-index
description: "INDEX.md 统一更新脚本 — 支持 prepend（表顶插入）和 append（追加）两种模式"
version: 1.0.0
tags: [fullstack, script, index, automation]
input:
  - index_path: INDEX.md 文件路径（如 docs/探路报告/INDEX.md）
  - row_content: 要插入的行内容（Markdown 表格行）
  - mode: "prepend"（表顶插入）或 "append"（追加）
output:
  - INDEX.md 行已更新
---

# INDEX.md 统一更新脚本

> 统一管理所有 INDEX.md 的更新操作。所有生成制品的 Phase 应通过此脚本更新 INDEX。

## 更新模式

### 模式一：prepend（表顶插入 — 默认）

用于探路报告/规格文档/实施计划/修改记录/接口文档 等按时间倒序排列的 INDEX。

**操作**：
1. Read `{index_path}` 获取当前内容
2. 在表格分隔行（`|---`）之后、第一条数据行之前插入新行
3. 插入内容格式：
   ```
   | {日期} | [{标题}](./{文件名}) | {一行摘要} |
   ```
4. Write 回原文件

**使用示例**：
```
Read scripts/update-index.md 并执行 prepend 模式：
- index_path: "docs/探路报告/INDEX.md"
- row_content: "| 2026-06-07 | [用户管理模块探路报告](./2026-06-07-用户管理模块.md) | 新增用户查询接口 |"
```

### 模式二：append（追加）

用于复盘记录/失败案例/Instinct 等按时间正序排列的 INDEX。

**操作**：
1. Read `{index_path}` 获取当前内容
2. 在表格最后追加新行
3. 插入内容格式：
   ```
   | {ID} | {日期} | {标题} | {类型/领域} | {严重度/置信度} | {来源} | {链接} |
   ```
4. Write 回原文件

**使用示例**：
```
Read scripts/update-index.md 并执行 append 模式：
- index_path: "docs/失败案例/FAILURE-INDEX.md"
- row_content: "| FC-202606-001 | 2026-06-07 | NPE in OrderService | bug | critical | retrospect | docs/复盘记录/xxx.md |"
```

---

## 通用约束

- 不要修改 INDEX.md 的标题和头部说明文字
- 插入后确保表格列对齐格式正确
- 如果 INDEX.md 不存在，先创建基本结构（标题 + 表头 + 分隔行）

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 1.0.0 | 2026-06-07 | 初始版本：支持 prepend/append 两种模式 |
