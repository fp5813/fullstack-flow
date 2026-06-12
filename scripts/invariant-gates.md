---
name: fullstack-flow/scripts/invariant-gates
description: "不变性门验证脚本 — 10 条核心不变性的自动化验证方法。每条不变性对应一个可执行的 grep/bash 验证命令。"
version: 1.1.0
tags: [fullstack, script, gate, invariant, automation]
input:
  - phase: 当前 Phase ID（quality-gate | coverage-check | review）
  - probe_report_path: 探路报告路径（quality-gate 阶段需要）
  - spec_path: 规格文档路径（coverage-check 阶段需要）
  - plan_path: 实施计划路径（coverage-check 阶段需要）
  - api_doc_dir: 接口文档目录（review 阶段需要）
  - controller_dir: Controller 源码目录（review 阶段需要）
  - frontend_api_dir: 前端 API 文件目录（review 阶段需要）
output:
  - gates_result: 每条不变性的通过/失败状态 + 证据行数
---

# 不变性门验证脚本

> 10 条核心不变性的自动化验证方法。每条不变性提供可执行的 grep/bash 命令。
> 每条不变性标注 `✅ 可自动验证` 或 `⚠️ 半自动（需人工确认）`。

## 不变性列表

---

### I1 — 探路报告包含 file:line 引用

**关联检查项**: C01, C02, C03  
**可验证性**: ✅ 可自动验证  
**阈值**: 至少 3 处 `file:line` 引用（java/vue/ts/js/xml）

```bash
# 检查探路报告中的 file:line 引用数量
grep -cP '\.(java|vue|ts|js|xml):\d+' "{probe_report_path}"

# 详细列出所有引用（供证据输出）
grep -nP '\.(java|vue|ts|js|xml):\d+' "{probe_report_path}"
```

**通过条件**: `count >= 3`

---

### I2 — L0-L3 每层至少 1 个 file:line 引用

**关联检查项**: C04, C05-C08, C09-C12, C13-C16  
**可验证性**: ✅ 可自动验证  
**阈值**: L0(页面层) ≥1, L1(Controller 层) ≥1, L2(Service 层) ≥1, L3(Mapper 层) ≥1

```bash
# L0 页面层 — Vue 组件引用
grep -cP '\.vue:\d+' "{probe_report_path}"

# L1 API 入口层 — Controller 引用
grep -cP 'Controller\.java:\d+' "{probe_report_path}"

# L2 Service 层 — Service 实现引用
grep -cP 'Service[a-zA-Z]*\.java:\d+' "{probe_report_path}"

# L3 Mapper/XML 层 — Mapper 引用
grep -cP 'Mapper\.java:\d+' "{probe_report_path}"
grep -cP '\.xml:\d+' "{probe_report_path}"
```

**通过条件**: `L0>=1 AND L1>=1 AND L2>=1 AND (L3_mapper>=1 OR L3_xml>=1)`

---

### I3 — 规格文档 AC 可追踪到探路报告

**关联检查项**: Phase 4.5 AC 覆盖率检查  
**可验证性**: ⚠️ 半自动（辅助 grep，最终需人工确认）  
**阈值**: 规格文档中的每个 AC 至少引用探路报告中的一个章节

```bash
# 提取规格文档中的所有 AC 编号
echo "规格文档 AC 清单:"
grep -oP 'AC\d+' "{spec_path}"

# 检查每个 AC 在探路报告中是否有对应引用
echo "AC 引用检查:"
all_found=true
for ac in $(grep -oP 'AC\d+' "{spec_path}"); do
  count=$(grep -c "$ac" "{probe_report_path}")
  echo "  $ac: $count 次引用"
  if [ "$count" -eq 0 ]; then
    all_found=false
  fi
done

if $all_found; then
  echo "I3: ⚠️ 辅助检查通过"
else
  echo "I3: ⚠️ 部分 AC 在探路报告中无引用，需人工确认"
fi
```

**通过条件**: 每个 AC 在探路报告中至少有 1 处引用（辅助检查，最终人工确认映射关系合理性）

---

### I4 — 实施计划 T### 任务对应到 AC

**关联检查项**: Phase 4.5 孤儿任务检测  
**可验证性**: ✅ 可自动验证  
**阈值**: 每个 T### 任务关联至少一个 AC

```bash
# 提取实施计划中的所有 T### 任务
echo "实施计划 T### 任务:"
grep -oP 'T\d+' "{plan_path}"

# 提取实施计划中所有 AC 引用
echo "实施计划 AC 引用:"
grep -oP 'AC\d+' "{plan_path}"

# 找出未关联任何 AC 的 T### 任务（孤儿任务）
echo "孤儿任务检查:"
orphan_count=0
while IFS= read -r line; do
  task=$(echo "$line" | grep -oP 'T\d+')
  ac_ref=$(echo "$line" | grep -oP 'AC\d+')
  if [ -n "$task" ] && [ -z "$ac_ref" ]; then
    echo "  ⚠️ 孤儿任务: $task"
    orphan_count=$((orphan_count + 1))
  fi
done < <(grep -n 'T[0-9]' "{plan_path}")

if [ "$orphan_count" -eq 0 ]; then
  echo "I4: ✅ 通过 (0 个孤儿任务)"
else
  echo "I4: ❌ 不通过 ($orphan_count 个孤儿任务未标注原因)"
fi
```

**通过条件**: 无未关联 AC 的孤儿任务（允许标注 prefactor/技术任务原因）

---

### I5 — 修改的代码通过编译

**关联检查项**: Phase 5 Build-Fix 出口  
**可验证性**: ✅ 可自动验证  
**阈值**: `mvn compile` 退出码为 0

```bash
# 运行编译检查
mvn compile -pl {affected_module} -DskipTests 2>&1 | tail -20
BUILD_SUCCESS=$?

# 检查退出码
if [ $BUILD_SUCCESS -eq 0 ]; then
  echo "I5: ✅ 编译通过"
else
  echo "I5: ❌ 编译失败"
fi
```

**通过条件**: 退出码 == 0

---

### I6 — 修改的代码通过测试

**关联检查项**: Phase 5 Build-Fix 出口, Gate 2.5  
**可验证性**: ✅ 可自动验证  
**阈值**: `mvn test` 退出码为 0，全部测试通过

```bash
# 运行测试
mvn test -pl {affected_module} 2>&1 | grep -E 'BUILD|Tests run|FAIL'
TEST_EXIT=$?

# 检查结果
if [ $TEST_EXIT -eq 0 ]; then
  echo "I6: ✅ 测试通过"
else
  echo "I6: ❌ 测试失败"
fi
```

**通过条件**: 退出码 == 0

---

### I7 — 接口文档覆盖所有新增 Controller 方法

**关联检查项**: Phase 5.5 文档完整性扫描  
**可验证性**: ✅ 可自动验证  
**阈值**: 新增/修改的 Controller 方法在接口文档中有对应文档文件

```bash
# 从 Controller 中提取所有 @RequestMapping 基路径
BASE_PATH=$(grep -rPn '@RequestMapping\("[^"]+"' "{controller_dir}" \
  | grep -oP '"[^"]+"' | tr -d '"' | head -1)
echo "Controller 基路径: $BASE_PATH"

# 提取所有 @*Mapping 方法和路径
echo "Controller API 方法:"
grep -rnP '@(Get|Post|Put|Delete|Patch)Mapping\("[^"]+"' "{controller_dir}" \
  | grep -oP '"[^"]+"' | tr -d '"' | sort > /tmp/controller_paths.txt
cat /tmp/controller_paths.txt

# 拼接完整路径（如果基路径存在）
echo "完整 API 路径:"
if [ -n "$BASE_PATH" ]; then
  while IFS= read -r path; do
    echo "$BASE_PATH$path"
  done < /tmp/controller_paths.txt > /tmp/full_controller_paths.txt
else
  cp /tmp/controller_paths.txt /tmp/full_controller_paths.txt
fi
cat /tmp/full_controller_paths.txt

# 列出接口文档目录
echo "接口文档文件:"
ls "{api_doc_dir}"/*.md 2>/dev/null

# 对比检查
echo "路径覆盖检查:"
all_covered=true
while IFS= read -r path; do
  matched=$(grep -rlF "$path" "{api_doc_dir}"/*.md 2>/dev/null)
  if [ -z "$matched" ]; then
    echo "  MISSING: $path (无对应文档)"
    all_covered=false
  else
    echo "  ✅ $path → $(basename $matched)"
  fi
done < /tmp/full_controller_paths.txt

if $all_covered; then
  echo "I7: ✅ 通过"
else
  echo "I7: ❌ 不通过"
fi
```

**通过条件**: 所有新增/修改的 Controller 方法在接口文档中有对应文档

---

### I8 — 前端 API 调用路径有对应接口文档

**关联检查项**: Phase 5.5 文档完整性扫描  
**可验证性**: ✅ 可自动验证  
**阈值**: 前端 `defHttp.get/post` 中引用的 API 路径在接口文档中有记录

```bash
# 从前端 API 文件中提取所有 API 调用路径
# 支持单引号、双引号和模板字符串（反引号）
grep -rPn "defHttp\.(get|post|put|delete)\(['\`\"/]" "{frontend_api_dir}" \
  | grep -oP "\(['\`\"/]\K[^'\"\`\)]+" \
  | sort -u > /tmp/frontend_api_paths.txt
echo "前端 API 路径:"
cat /tmp/frontend_api_paths.txt

# 从接口文档中提取所有 API 路径（反引号包裹的路径）
grep -rnP '\`/[^\`]+\`' "{api_doc_dir}"/*.md 2>/dev/null \
  | grep -oP '\`\K[^\`]+' \
  | sort -u > /tmp/api_doc_paths.txt
echo "接口文档 API 路径:"
cat /tmp/api_doc_paths.txt

# 对比
echo "路径覆盖检查:"
all_covered=true
while IFS= read -r path; do
  clean_path=$(echo "$path" | tr -d "'\"")
  matched=$(grep -c "$clean_path" /tmp/api_doc_paths.txt 2>/dev/null)
  if [ "$matched" -eq 0 ]; then
    echo "  MISSING: $clean_path (无对应文档)"
    all_covered=false
  else
    echo "  ✅ $clean_path"
  fi
done < /tmp/frontend_api_paths.txt

if $all_covered; then
  echo "I8: ✅ 通过"
else
  echo "I8: ❌ 不通过"
fi
```

**通过条件**: 所有前端引用的 API 路径在接口文档中有记录

---

### I9 — 探路报告无修改建议/实施方案

**关联检查项**: C26, C27  
**可验证性**: ✅ 可自动验证  
**阈值**: 探路报告中不包含实施方案关键词

```bash
# 检查是否包含禁止关键词（使用 -o 计数，同一行可能含多个关键词）
grep -oPi '(修改建议|实施方案|修改代码|代码修改|建议修改|修改如下)' "{probe_report_path}" | wc -l
```

**通过条件**: `count == 0`

---

### I10 — 涉及 DB 时探路报告包含数据采样

**关联检查项**: C31, C32, C33, C34, C35  
**可验证性**: ✅ 可自动验证  
**阈值**: 涉及 DB 时至少包含 1 条 SQL 查询或表结构描述

```bash
# 检查是否包含 SQL 关键词（使用 -o 计数而非 -c，因为同一行可能含多个关键词）
grep -oPi '(SELECT|INSERT|UPDATE|DELETE|FROM|WHERE|TABLE|table_name|字段|数据行)' "{probe_report_path}" | wc -l
```

**通过条件**: `count >= 3`（涉及 DB 时）；不涉及 DB 时跳过

> **注意**：SQL 关键词可能集中在同一行（如 `SELECT * FROM`），`grep -c` 按行计数会低估。本命令已改用 `grep -oPi | wc -l` 按匹配次数计数。

---

## 按 Phase 分组

| Phase | 必检不变性 | 条件 |
|-------|-----------|------|
| quality-gate | I1, I2, I9, I10 | 探路报告完整后 |
| coverage-check | I3, I4 | 规格+计划就绪后 |
| code (Phase 5 出口) | I5, I6 | 代码修改后 |
| review (Phase 5.5) | I7, I8 | 接口文档生成后 |

## 验证结果输出格式

所有不变性门验证完成后，输出以下格式的结果表格：

```markdown
## 不变性门验证结果
| ID | 不变性 | 状态 | 证据 |
|----|--------|:----:|------|
| I1 | file:line 引用完整 | ✅ | 找到 12 处引用 |
| I2 | 分层引用完整 | ✅ | L0:3 L1:4 L2:3 L3:2 |
| I3 | AC→探路报告可追踪 | ⚠️ 辅助检查 | AC01:3次 AC02:1次 |
| I4 | T###→AC 映射完整 | ✅ | 0 个孤儿任务 |
| I5 | 编译通过 | ✅ | BUILD SUCCESS |
| I6 | 测试通过 | ✅ | 12 passed, 0 failed |
| I7 | 接口文档覆盖完整 | ✅ | 5/5 方法已覆盖 |
| I8 | 前端 API 文档覆盖 | ✅ | 8/8 路径已覆盖 |
| I9 | 无修改建议 | ✅ | 0 处违规 |
| I10 | 数据采样完整 | ✅ | 找到 7 处 SQL 引用 |
| **通过率**: {pass}/{total} = {percent}% |
```

---

## 使用方式

在 Phase 文档的对应步骤中引用：

```markdown
### Step X: 不变性门验证

Read `scripts/invariant-gates.md` 并执行以下不变性门：
- phase: "quality-gate" | "coverage-check" | "review"
- 对应输入参数（探路报告路径、规格文档路径等）

执行 grep/bash 命令验证每项不变性，输出结果表格追加到当前制品。
```

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 1.1.0 | 2026-06-12 | 修复 I3/I4/I7/I8 命令：I3 增加状态输出和退出码判断；I4 补充孤儿任务检测完整 shell 实现；I7 增加 @RequestMapping 基路径拼接和完整路径提取；I8 增加反引号模板字符串支持和 tr 清理 |
| 1.0.0 | 2026-06-12 | 初始版本：10 条核心不变性 + 自动化验证方法 |
