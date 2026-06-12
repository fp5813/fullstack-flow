---
name: fullstack-flow/phases/api-doc
description: "API 接口文档生成：根据实际代码主动生成/更新接口文档，一个接口对应一个文档文件"
version: 1.0.0
tags: [fullstack, codebuddy-only, api-documentation]
role: codebuddy-docs
model: deepseek-v4-flash
tools: [Read, Write, Edit, Grep, Agent]
references:
  - ../references/codegraph-reference.md
---

# Phase 5.6: API 接口文档生成

> 代码改完了，接口文档不能落下。一个接口对应一个文档文件，供调用方使用。
> 此阶段在 Phase 5 代码修改完成后、Phase 5.5 审核之前执行。

## 快速概览

```
Step 0 读取状态 → Step 1 确定接口范围 → Step 2 生成文档（一个接口一个文件）
  → Step 3 自检准确性 → Step 4 更新索引 → Phase 出口 → review
```

**核心**: 每个新增/变更接口生成独立 .md 文档 + 更新 INDEX.md

## 职责

**输入**：Phase 5 代码修改清单 + Controller 代码 + Feign 客户端代码 + DTO/实体代码  
**输出**：`docs/接口文档/` 下的新增/更新接口文档  
**模式**：只读代码 + 写文档（不修改代码）

## 核心原则

- **一个接口一个文档**：每个对外暴露的接口独立一个 `.md` 文件，清晰展示请求/响应结构
- **文档编号递增**：`NN-模块名-接口名.md`，`NN` 为两位数字序号
- **文档内容完整**：必须包含接口概述、请求说明、响应说明、业务规则、调用示例、变更历史
- **索引同步更新**：新增文档后同步更新 `docs/接口文档/INDEX.md` 和 `docs/接口文档/00-全API索引.md`

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="api-doc", next_phase="review"
- 额外：验证前置制品 `artifacts.code_changes` 非空

### Step 1: 确定待生成的接口范围

扫描 Phase 5 的代码修改，提取本次修改涉及的所有新增/修改的 Controller 端点：

| 来源 | 方法 |
|------|------|
| 新增 Controller | `grep @RequestMapping + @PostMapping/@GetMapping` 提取路径和方法 |
| 修改的 Controller | 检查已有接口的变更（参数/路径/返回类型） |
| Feign 客户端 | 与 Controller 的路径和签名做比对验证 |

### Step 2: 收集接口信息

对每个待生成的接口，收集以下信息：

| 信息 | 来源 | 方法 |
|------|------|------|
| 接口路径 | Controller `@RequestMapping` + 方法注解 | Read 文件 |
| HTTP 方法 | `@PostMapping/@GetMapping/@PutMapping/@DeleteMapping` | Read 文件 |
| 请求参数 | `@RequestBody` DTO / `@RequestParam` / `@PathVariable` | Read DTO 类 |
| 返回类型 | 方法返回类型 + `BaseResult<T>` 泛型参数 | Read 代码 |
| DTO 字段 | 请求/响应 DTO 的字段 + 类型 + 说明 | Read DTO 类（含 `@Schema` 注解） |
| 数据库表 | Mapper 操作的实体对应的表结构 | `mysql-{子项目名}.describe_table` |
| 业务规则 | Service 层的特殊逻辑（过滤/校验/事务等） | Read Service 代码 |
| Feign 客户端 | `@FeignClient` 配置 + 方法签名 | Read Feign 接口 |

### Step 3: 生成接口文档

按照标准模板为每个接口生成独立文档文件 `docs/接口文档/NN-模块名-接口名.md`：

```markdown
# 接口名称

## 接口概述

- **接口路径**: ...
- **Content-Type**: `application/json`
- **是否需要登录**: 是/否
- **请求头要求**: ...
- **调用方**: ...
- **新增日期**: ...

---

## 请求

### 请求体

```json
{...}
```

### 请求参数字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|

---

## 响应

### 响应体结构

```json
{...}
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|

---

## 业务规则

| 规则 | 说明 |
|------|------|

---

## 调用示例

...

---

## 变更历史

| 日期 | 变更说明 |
|------|---------|
```

### Step 4: 验证文档准确性

生成文档后进行自检：

| 检查项 | 方法 | 通过条件 |
|--------|------|---------|
| 路径准确 | 对比 `@RequestMapping` | 完全一致 |
| 请求参数完整 | 对比 DTO 字段 | 所有必填参数已覆盖 |
| 返回字段完整 | 对比 DTO/实体字段 | 响应字段与实际一致 |
| 业务规则覆盖 | 对比 Service 逻辑 | 关键业务规则已记录 |
| 调用示例可执行 | 检查 Feign 或 curl 示例 | 语法正确 |

### Step 5: 同步索引文件（Read `scripts/update-index.md`）

执行 prepend 模式（2 个 INDEX 文件）：
1. index_path: "docs/接口文档/INDEX.md", row_content: "| 模块名 | [NN-模块名-接口名.md](NN-模块名-接口名.md) | 当前日期 |"
2. index_path: "docs/接口文档/00-全API索引.md", row_content: "| 模块名 | HTTP方法 | 接口路径 | 说明 | [NN-模块名-接口名.md](NN-模块名-接口名.md) |"

### Step 6: 标准表格输出

汇总本次生成的接口文档，输出标准表格：

```markdown
## API 接口文档生成（Phase 5.6）
| 接口 | 文件 | 状态 |
|------|------|------|
| `POST /api/internal/users/batch` | `01-内部用户-批量查询用户.md` | 🆕 新增 |
| `POST /api/internal/messages/send` | `02-内部系统-发送站内信.md` | 🆕 新增 |
```

## 完整性自检

- [ ] 所有新增/修改的 Controller 接口均已生成文档
- [ ] 每个文档包含完整的请求/响应字段说明
- [ ] 业务规则已记录（如有）
- [ ] Feign 调用示例已提供（内部接口时）
- [ ] `docs/接口文档/INDEX.md` 已同步
- [ ] `docs/接口文档/00-全API索引.md` 已同步
- [ ] 文档编号不重复、递增

## 约束

- 不改代码，只写文档
- 一个接口一个独立 `.md` 文件
- 文档中的接口路径、参数、返回类型必须与实际代码一致
- 涉及真实数据示例时，使用测试环境的脱敏数据
- 标注生成/更新日期

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 `artifacts.api_docs`：文档路径列表和时间戳
2. 参数: current_phase="api-doc", next_phase="review"
