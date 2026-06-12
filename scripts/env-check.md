---
name: fullstack-flow/scripts/env-check
description: "环境预检脚本 — 检查项目工具链、MCP 服务就绪状态和 CI 预检（编译/TypeScript/Lint/测试）"
version: 2.0.0
tags: [fullstack, script, environment, precheck]
input:
  - project_type: "java"（后端）/ "vue"（前端）/ "both"（全栈）
  - mcp_mysql_name: mysql MCP 服务名（如 mysql-moma）
  - mcp_codegraph_name: codegraph MCP 服务名（如 codegraph）
  - mcp_api_fetcher_name: api-fetcher MCP 服务名（可选）
output:
  - 环境摘要 Markdown 表格
  - 缺失工具/服务警告（不阻塞流程）
---

# 环境预检脚本

> 执行在 clarify.md Step 0.5。检查项目工具链是否可用 + MCP 服务是否就绪。

## 前置约束

- 依赖检查不阻塞流程（缺失时输出警告，继续执行）
- 环境摘要作为后续 Phase 的参考上下文

## 执行流程

### Step 0.5.1: 项目工具链检查

根据 `project_type` 检查对应的命令行工具：

| 检查项 | 命令 | 输出格式 |
|-------|------|---------|
| Node.js | `which node && node --version` | `node {version}` 或 `❌ node missing` |
| npm | `which npm && npm --version` | `npm {version}` 或 `❌ npm missing` |
| JDK/Maven | `which mvn && mvn --version \| head -1` | `Maven {version} / JDK {version}` 或 `❌ mvn missing` |
| Git | `which git && git version` | `git {version}` 或 `❌ git missing` |

**分支逻辑**：
- `project_type = "java"`：检查 mvn + git（node 可选）
- `project_type = "vue"`：检查 node + npm + git
- `project_type = "both"`：全部检查

### Step 0.5.2: MCP 服务就绪检查

检查指定的 MCP 服务是否可用：

| 服务 | 检查方式 | 就绪条件 |
|------|---------|---------|
| `{mcp_codegraph_name}` | 尝试调用一次 codegraph API | 返回正常 |
| `{mcp_mysql_name}` | 尝试执行一次简单查询 | 返回正常 |
| `{mcp_api_fetcher_name}`（可选）| 检查服务清单 | 已定义 |

### Step 0.5.3: 输出环境摘要

输出标准 Markdown 表格：

```markdown
## 环境摘要
| 项目 | 状态 |
|------|:----:|
| 工作目录 | {pwd 结果} |
| 项目类型 | {java/vue/both} |
| Node.js | ✅/❌ {version} |
| npm | ✅/❌ {version} |
| JDK/Maven | ✅/❌ {version} |
| Git | ✅/❌ {version} |
| MCP codegraph | ✅/❌ |
| MCP mysql-{项目} | ✅/❌ |
```

### Step 0.5.4: CI 预检（本地模拟 CI gate）

在本地模拟 CI pipeline 的核心检查，确保提交前发现低级问题：

| 检查项 | 命令 | 适用类型 | 通过条件 |
|-------|------|:-------:|---------|
| 编译检查 | `mvn compile -pl {module} -DskipTests 2>&1 \| tail -5` | java | 退出码 0 |
| TypeScript 检查 | `npx vue-tsc --noEmit 2>&1 \| tail -10` | vue | 退出码 0 |
| Lint 检查 | `mvn checkstyle:check 2>&1 \| tail -5` 或 `npx eslint .` | both | 退出码 0 |
| 测试预检 | `mvn test -pl {module} 2>&1 \| grep -E 'Tests run:.*Failures: 0'` | java | 无失败 |

**分支逻辑**：
- `project_type = "java"`：编译 + lint + 测试预检
- `project_type = "vue"`：TypeScript + lint
- `project_type = "both"`：全部检查

> CI 预检是本地执行的快速检查（≤2 分钟），不等同于完整 CI pipeline。
> 预检失败 → 在 Phase 5 的 Build-Fix 循环中修复。
> 如果 Phase 1 时项目尚未修改代码，编译检查可跳过（在 Phase 5 Build-Fix 时执行）。

### Step 0.5.5: 依赖缺失处理

- 缺失项仅输出警告，不阻止流程
- 如果是 MCP 服务缺失，在后续 Phase（probe/audit）的依赖项中再次验证
- 将缺失项记录到 state.yaml 的 `warnings[]` 数组（如有扩展）

---

## 使用方式

在 clarify.md（Phase 1）的 Step 0.5 替换为：

```markdown
### Step 0.5: 环境预检

Read `scripts/env-check.md` 并执行，参数如下：
- project_type: "both"
- mcp_mysql_name: "mysql-{子项目名}"
- mcp_codegraph_name: "codegraph"
- mcp_api_fetcher_name: "api-fetcher"
```

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 2.0.0 | 2026-06-12 | 新增 Step 0.5.4 CI 预检（编译/TypeScript/Lint/测试检查） |
| 1.0.0 | 2026-06-07 | 初始版本：工具链检查 + MCP 就绪检查 |
