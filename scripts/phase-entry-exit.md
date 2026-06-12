---
name: fullstack-flow/scripts/phase-entry-exit
description: "Phase 入口/出口协议脚本 — 统一管理 state.yaml 的状态读取、验证、切换和 metrics 记录"
version: 1.0.0
tags: [fullstack, script, protocol, automation]
input:
  - current_phase: 当前 Phase ID（如 clarify, probe, code）
  - next_phase: 下一 Phase ID（如 probe, spec, rule-sync）
  - artifacts_key: 制品 key 名（如 probe_reports, spec_docs, code_changes）
  - artifacts_path: 制品路径（可选）
  - prereq_artifacts: 前置制品列表（如 ["probe_reports", "probe_index"]）
  - is_gate_phase: 是否为门控 Phase（true/false，默认 false）
  - gate_passed: 门控是否通过（仅 is_gate_phase=true 时使用）
output:
  - state.yaml 更新
  - phase_durations 记录
---

# Phase 入口/出口协议脚本

> 统一的入口/出口流程。所有 Phase 应通过 `Read` 此脚本执行入口/出口操作。
> 用法：Phase 文档中 `Read scripts/phase-entry-exit.md` 并按参数替换占位符。

## 入口流程

### Step 0: 读取工作流状态

1. Read `.codebuddy/workflow/state.yaml`
2. 验证 `phase.current == "{current_phase}"`
3. 验证前置制品存在：
   ```
   {prereq_artifacts}  # 前置制品列表
   ```
   用 `Read` 或 `Bash ls` 确认每个制品路径存在。

4. 设置 Phase 状态：
   ```
   phase.status = "in_progress"
   phase.started_at = 当前时间
   ```

5. 更新会话：
   ```
   session.last_activity = 当前时间
   ```

## 出口流程

### Step Final: Phase 出口

1. **更新制品引用**: （如适用）
   ```
   artifacts.{artifacts_key}.path = "{artifacts_path}"
   artifacts.{artifacts_key}.timestamp = 当前时间
   ```

2. **更新门控结果**（仅门控 Phase）:
   ```
   gates_summary.{gate_name}.status = "{passed|failed}"
   gates_summary.{gate_name}.score = {分数}
   gates_summary.{gate_name}.failed_items = [{失败项列表}]
   ```
   门控不通过时：`gate_retries += 1`

3. **Phase 完成标记**:
   ```
   phase.status = "completed"
   phase.completed_at = 当前时间
   ```

4. **进度记录**:
   ```
   progress.phases_completed.append("{current_phase}")
   ```

5. **切换到下一 Phase**:
   - 正常流转：
     ```
     phase.current = "{next_phase}"
     phase.status = "pending"
     ```
   - 门控失败时回退：
     ```
     phase.current = "{fallback_phase}"
     phases_blocked = [{reason}]
     ```
   - 工作流结束（Phase 6.6 audit完成后）：
     ```
     phase.current = null
     phase.status = "completed"
     ```

6. **记录耗时 metrics**:
   ```
   metrics_snapshot.phase_durations[{current_phase}] = 当前时间 - phase.started_at（分钟数）
   ```
   额外记录（如适用）：
   - `total_duration_min` 累加（clarify, probe 等初始 Phase）
   - `build_fixes` 计入修复次数（code Phase）
   - `gate_retries` 门控重试次数（门控 Phase）

7. **记录工具和 MCP 使用情况**:
   ```
   metrics_snapshot.tools_used.{current_phase} = {
     mcp_calls: [
       { tool: "codegraph_context", count: N, input_tokens: N, output_tokens: N },
       { tool: "mysql_execute_query", count: N },
       { tool: "api_fetcher_call", count: N }
     ],
     key_files_loaded: ["path/to/file1", "path/to/file2"]
   }
   ```
   记录当前 Phase 中使用的每个 MCP 工具的调用次数和 token 消耗。

8. **记录 token 审计**:
   ```
   metrics_snapshot.token_audit.{current_phase} = {
     input_tokens: N,
     output_tokens: N,
     mcp_tool_calls: N,
     key_files: [已加载的关键文件列表]
   }
   metrics_snapshot.token_accumulated = {
     total_tokens: 累计值,
     phases_count: 已完成 Phase 数
   }
   ```

9. **更新会话**:
   ```
   session.last_activity = 当前时间
   ```

---

## 使用方式

在 Phase 文档中替换入口/出口的对应部分为：

### 入口替换（替换 Step 0 部分）
```markdown
### Step 0: 入口（执行统一协议）

Read `scripts/phase-entry-exit.md` 并应用入口流程，参数如下：
- current_phase: "{current_phase}"
- next_phase: "{next_phase}"
- artifacts_key: "{artifacts_key}"
- prereq_artifacts: [{prereq_list}]
```

### 出口替换（替换 Phase 出口部分）
```markdown
### Step Final: Phase 出口（执行统一协议）

Read `scripts/phase-entry-exit.md` 并应用出口流程，参数如下：
- current_phase: "{current_phase}"
- next_phase: "{next_phase}"
- artifacts_key: "{artifacts_key}"
- artifacts_path: {artifacts_path}
- is_gate_phase: {true/false}
- gate_passed: {true/false}
- phase_duration: {计算耗时}
```

---

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 1.0.0 | 2026-06-07 | 初始版本：统一入口/出口协议，含 metrics 记录 |
