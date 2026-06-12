---
name: fullstack-flow/scripts/build-fix
description: "Build-Fix 自动循环脚本 — 编译失败后自动分类错误类型、执行修复策略、最多循环 N 轮"
version: 1.0.0
tags: [fullstack, script, build, fix, automation]
input:
  - project_type: "java"（后端）或 "vue"（前端）或 "both"
  - max_rounds: 最大修复轮数（默认 3）
  - build_command: 构建命令（根据 project_type 确定）
  - test_command: 测试命令（可选）
output:
  - 编译通过或修复后结果
  - build_fixes 计数
---

# Build-Fix 自动循环脚本

> 编译/测试失败后自动执行：捕获输出 → 分类错误 → 修复 → 重验证，最多循环 N 轮。

## 执行流程

### Step 1: 运行构建命令并捕获输出

根据 `project_type` 运行对应的构建命令：

| 类型 | 构建命令 |
|:----:|---------|
| java | `mvn compile -pl {module} -DskipTests 2>&1 \| tee /tmp/build_output.txt` |
| vue | `npx vue-tsc --noEmit 2>&1 \| tee /tmp/build_output.txt` |
| both | 先 java 再 vue 顺序执行 |

输出保存到临时文件用于后续分析。

### Step 2: 错误分类检测

读取构建输出，按以下 6 种类型分类错误：

| 分类 | 匹配模式 | 错误类型 | 严重度 |
|:----:|---------|---------|:------:|
| 语法错误 | 输出包含 `error:` 且行号指向源代码 | 编译语法 | 高 |
| 类型不匹配 | 输出包含 `incompatible types` 或 `Type mismatch` | 类型系统 | 高 |
| 缺失导入 | 输出包含 `cannot find symbol` 或 `Cannot find name` | 依赖/导入 | 中 |
| 方法签名 | 输出包含 `no suitable method` | API 使用错误 | 高 |
| 测试断言 | 输出包含 `FAIL:` 或 `expected but was` | 测试逻辑 | 低 |
| TypeScript | 输出包含 `error TS` 或 `Type 'X' is not assignable` | TS 类型 | 中 |

**输出**：按严重度排序的错误列表，每项包含 `[分类] 文件:行号: 错误描述`。

### Step 3: 修复策略

根据错误分类执行对应的修复策略：

#### 3.1 语法错误
- 定位错误文件行号
- 修正语法（缺失括号、分号、花括号等）
- Read 错误文件检查上下文

#### 3.2 类型不匹配
- 检查赋值两侧的类型定义
- 确认是否需要强制转换或修改类型声明
- 确认实体类/VO 字段类型一致性

#### 3.3 缺失导入
- 确认缺失的符号来自哪个包
- 如果是项目内生类：检查是否未被 import
- 如果是外部依赖：检查 pom.xml/package.json 是否已引入

#### 3.4 方法签名
- Read 目标方法定义
- 检查调用参数是否匹配参数列表

#### 3.5 测试断言
- Read 测试代码，确认期望值与实际值的差异
- 如果是业务变更导致：更新测试断言
- 如果是代码错误：修正业务逻辑

#### 3.6 TypeScript
- 检查类型定义与实际值的 shape 是否一致
- 使用 TypeScript 类型守卫或类型断言修正

### Step 4: 循环控制

```
round = 1
while round <= max_rounds AND 编译未通过:
  执行修复策略
  重新运行构建命令
  if 编译通过:
    输出 "Build 通过，共修复 {round} 轮"
    break
  else:
    if round >= max_rounds:
      输出 "Build 超过最大轮数 ({max_rounds})，需人工介入"
      break
    else:
      round += 1
      分析新的错误输出，继续循环
```

### Step 5: 验证与记录

1. 编译通过后，运行测试命令（如提供）：`mvn test -pl {module}` 或 `npx vitest run`
2. 记录修复历史（追加到修改记录或 Phase 制品）：
   ```
   ## Build-Fix 记录
   | 轮次 | 错误类型 | 修复内容 | 结果 |
   |------|---------|---------|:----:|
   | 1 | 语法错误 | 修复括号缺失 | 编译通过 |
   ```
3. 更新 metrics：`build_fixes += {round}`

---

## 使用方式

在 code.md（Phase 5）的 Step 4 替换为：

```markdown
### Step 4: Build-Fix 循环

Read `scripts/build-fix.md` 并执行，参数如下：
- project_type: "java"
- max_rounds: 3
```

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 1.0.0 | 2026-06-07 | 初始版本：6 种错误分类 + 3 轮自动修复 |
