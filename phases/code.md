---
name: fullstack-flow/phases/code
description: "最小修改：按实施计划执行，FE/BE 并行子代理，过影响范围自检，在 IDE 中编译验证。"
version: 4.0.0
tags: [fullstack, codebuddy-only, code, implementation]
role: codebuddy-coder
model: deepseek-v4-pro
tools: [Read, Write, Edit, Grep, Agent]
references:
  - ../references/spec-driven-development.md
---

# Phase 5: 最小修改

> 按计划执行，最小影响原则。TDD 先行（Red→Green→Refactor），Build-Fix 自动修复。

## 快速概览

```
5.0 读取工作流状态 → 5.1 影响范围自检 → 5.2 工作流模式选择
  → 5.3 任务分配 → 5.3.5 TDD (Red→Green→Refactor)
  → 5.3.6 并行执行 → 5.3.7 TDD 回归 → 5.4 合并验证
  → 5.4.5 Build-Fix（编译失败时循环）
  → 5.4.6 不变性门验证（I5/I6）→ Phase 出口 → api-doc
```

**核心原则**: 测试先于代码 / 最小修改 / 编译通过方可推进 / build-fix 最多 3 轮

## 职责

**输入**：实施计划（`docs/实施计划/`）  
**输出**：修改后的代码文件  
**模式**：编码

## 职责

**输入**：实施计划（`docs/实施计划/`）  
**输出**：修改后的代码文件  
**模式**：编码

## 流程

### Step 0: 入口（Read `scripts/phase-entry-exit.md` 入口流程）

- parameters: current_phase="code", next_phase="api-doc"
- 额外：验证前置制品 `artifacts.plan.path` 存在且 `artifacts.gate_4_5.status == "passed"`

### 5.1 影响范围自检（修改前逐项确认）

| 检查项 | 问题 |
|--------|------|
| 前后端同步 | 改 Vue 时 Entity 要加字段？ |
| Service 接口 | Impl 改了，Interface 要改？ |
| 权限注解 | 新接口需要 @RequiresPermissions？ |
| 事务边界 | 多表操作需要 @Transactional？ |
| 文档同步 | 修改是否影响已有文档/业务规则？ |

### 5.1.5 ⚡ 异步竞态分析（修改 async 函数前必查）

修改包含 `await` 的异步函数时，分析以下竞态场景并标注处理方式：

| 场景 | 问题 | 处理方式 |
|------|------|---------|
| API 响应时值已变 | `await` 期间 props/state 被外部修改，旧响应覆盖新状态 | API 返回后检查 `originalValue !== currentValue` |
| 并发请求 | 多次调用在 await 期间堆积，后返回的旧结果覆盖 | 放弃返回前检查值是否仍匹配 |
| 守卫失效 | `initializing` 等普通变量守卫在响应式系统外 | 改用 ref 或捕获 originalValue 比较 |

### 5.2 工作流模式选择

从 `state.yaml` 读取 `workflow.pattern` 选择执行策略：

| 模式 | 执行策略 | 说明 |
|------|---------|------|
| `classify` | 单 Agent 顺序执行 | BUG 修复：定位 → 修改 → 验证 |
| `distribute` | FE/BE 并行（默认） | 新功能：前端 + 后端并行实现 |
| `adversarial` | 实现 + 审核循环 | 优化/安全：实现 Agent 输出 → 审核 Agent 审查 |
| `generate` | 多方案对比 | 重构：生成 2~3 方案 → 对比选最优 |

> 详细模式定义见 `references/dynamic-workflows-reference.md`。

### 5.3 任务分配

根据计划将任务拆为前端（Vue/API/data.ts/枚举）和后端（Controller/Service/Mapper/Entity）两组。

### 5.3.5 TDD: 先写 API 测试，再实现代码（涉及后端 Controller 时必做）

> **TDD 原则**: 先写测试（Red）→ 实现代码（Green）→ 重构优化（Refactor）
> Java 项目统一走 API 接口测试，不创建单元测试类。
>
> 详细测试规范见 `java-test-standards` 子技能（Read `../java-test-standards/SKILL.md`）。
> 开发规范见 `java-dev-standards` 子技能（Read `../java-dev-standards/SKILL.md`）。

#### 5.3.5.0 用户确认测试流程

在执行任何测试之前，向用户展示测试计划并请求确认：

```markdown
**请确认测试流程：**

1. **测试范围**: {涉及的 Controller 列表}
2. **测试方式**: Python 脚本 + `python tests/api/test_{entity}.py`
3. **数据准备**: {通过查询哪些数据库表准备数据 / 通过 API 创建哪些数据}
4. **验证方式**: 调用接口 → 查看响应 → 查询数据库确认

是否按以上流程执行测试？(yes/no)
```

- 用户确认 → 进入测试执行
- 用户提出修改 → 调整后重新展示
- 用户拒绝 → 回退 Phase 4 调整计划

#### 5.3.5.1 识别影响范围

在写任何代码之前，先确认本次修改涉及的所有后端 Controller 接口：
- 从实施计划中提取涉及的 Controller 列表
- 使用 `codegraph` / `Grep` 搜索 `@RequestMapping` 或 `@XxxMapping` 确认接口路径
- 使用 `Find References` 确认前端 API 调用路径

#### 5.3.5.2 阶段一 Red: 先写 API 测试

在 `tests/api/` 目录下，为每个涉及的 Controller 创建 `test_{entity}.py`：

| 规范 | 说明 |
|------|------|
| 命名规则 | `test_{entity}.py`（如 `OrderController` → `test_order.py`） |
| 运行方式 | `python tests/api/test_{entity}.py`（Python 3.8+，需安装 `requests` 库） |
| 启动命令 | `python tests/api/test_{entity}.py` |

**测试脚本内容**（在实现代码前写好）：
```python
"""
{Entity} API 测试脚本
运行: python tests/api/test_{entity}.py
"""
import requests
import uuid
import sys

BASE_URL = "http://localhost:8080"

def main():
    # 1. 准备数据（优先查询数据库，不足时通过 API 创建补充）
    token = login("admin", "admin123")
    parent_id = query_db("SELECT id FROM parent WHERE ...")
    if not parent_id:
        resp = requests.post(f"{BASE_URL}/api/parent/create",
            json={"name": f"测试数据-{uuid.uuid4()}"},
            headers={"Authorization": f"Bearer {token}"})
        parent_id = resp.json().get("result", {}).get("id")

    # 2. 调用目标 API（此时应返回预期错误或空结果，因代码尚未修改）
    resp = requests.get(f"{BASE_URL}/api/xxx/detail",
        params={"id": parent_id},
        headers={"Authorization": f"Bearer {token}"})

    # 3. 验证结果
    assert_contains(resp.text, "expectedField")
    print(f"PASS: {resp.text}")

def login(username, password):
    resp = requests.post(f"{BASE_URL}/api/auth/login",
        json={"username": username, "password": password})
    return resp.json().get("result", {}).get("token")

def assert_contains(text, expected):
    if expected not in text:
        print(f"FAIL: 期望值 '{expected}' 未找到")
        sys.exit(1)

def query_db(sql):
    print(f"[DB Query] {sql}")
    return None

if __name__ == "__main__":
    main()
```

**测试数据准备规则**：详见 `java-test-standards` 子技能第 4 章「数据准备规范」。

**首次运行 Red 验证**：
- 运行 `python tests/api/test_{entity}.py`
- 预期结果：测试失败（因代码尚未修改，API 返回结果与预期不一致）
- 记录测试失败输出，作为代码实现的输入

#### 5.3.5.3 阶段二 Green: 实现代码使测试通过

在测试已定义契约的基础上实现代码：

| TDD 步骤 | 动作 | 验证 |
|---------|------|------|
| Red | 创建测试类 + 运行确认失败 | 测试输出显示预期失败 |
| Green | 实现 Controller/Service/Mapper 代码 | 再次运行测试 → **通过** |

**实现原则**：
- 测试类定义了"做什么"（契约），代码实现"怎么做"
- 不实现测试未覆盖的功能（YAGNI）
- 代码实现到测试通过为止，不过度设计

#### 5.3.5.4 阶段三 Refactor: 重构优化（可选）

测试通过后，如有必要对代码进行重构：
- 提取重复逻辑
- 优化命名
- 保持测试继续通过

**重构约束**：重构后必须重新运行测试确认通过。

#### 5.3.5.5 并行执行与 TDD 集成

TDD 在并行执行中的工作方式：

**distribute 模式（默认）**:
```
TDD 前置（全部 BE 测试先写）:
  Agent BE-Test: 为所有涉及的 Controller 创建 ApiTest → Red 验证
    ↓
Agent FE: 前端实现        Agent BE: 后端实现
    ↓                          ↓
合并验证: 运行全部 ApiTest → Green 通过
```

**classify 模式**:
```
Agent: 创建 ApiTest → Red 验证 → 实现代码 → Green 通过
```

**已存在测试类**：追加新接口的测试方法，保留原有测试方法。运行全部测试确保回归通过。

**无后端修改时**：跳过此步骤。

**验证**：代码实现完成后，运行 `python tests/api/test_{entity}.py` 确认测试全部通过（Green）。

### 5.3.6 并行执行

有依赖 → 串行（如 FE 依赖 BE，则 BE 先完成）。无依赖 → 并行（默认情况）。

**Agent FE**: 前端修改 — 加载 `vue-standards` 子技能 + 必读 `references/vue-reactivity.md`、`references/vue-component-data-flow.md`、`references/vue-composables.md`，按计划逐一执行，每完成一个任务打勾。验证：IDE 中编译 + `vue-tsc --noEmit`（如项目配置了 TypeScript 严格模式）

**Agent BE**: 后端修改 — 在 TDD 测试（5.3.5）已就绪的基础上，按计划逐一实现代码使测试通过。每完成一个任务运行一次测试确认 Green。验证：`python tests/api/test_{entity}.py` 运行测试全部通过。

> **上下文提示**：子代理返回主会话时，输出格式限定为"任务完成清单 + 关键决策点"的摘要形式（每代理 ≤1K tokens），避免完整对话日志填充主会话上下文。

### 5.3.7 TDD 测试回归（Green 验证）

所有代码实现完成后：
1. 运行全部 `tests/api/test_{entity}.py` 确认全部通过
2. 如有新增接口，确认测试覆盖了新增功能
3. 如有修改接口，确认测试验证了变更行为
4. 确认 `mvn test-compile` 编译通过

### 5.4 合并验证

| 验证项 | 方法 | 通过条件 |
|--------|------|---------|
| 编译通过 | IDE 编译 / `mvn compile` / `vue-tsc --noEmit` | 无编译错误 |
| 测试通过 | `npx vitest run` / `python tests/api/test_{entity}.py` 运行测试脚本 | 全部通过 |
| 配置引用 | Grep 检查引用的接口路径/方法名一致 | 前后匹配 |

### Step 4: Build-Fix（Read `scripts/build-fix.md`）

Read `scripts/build-fix.md` 并执行：
  - project_type: "java"（后端）或 "vue"（前端）或 "both"（全栈）
  - max_rounds: 3

修复记录自动计入 `metrics_snapshot.build_fixes`。

### Step 4.5: 不变性门验证（I5/I6）

Build-Fix 循环通过后，执行不变性门验证确认编译和测试状态：

Read `scripts/invariant-gates.md` 并执行以下不变性门的检查：

| 不变性 | 验证内容 | 命令 | 阈值 |
|--------|---------|------|:----:|
| **I5** | 编译通过 | 检查 Build-Fix 最终退出码 | 退出码 == 0 |
| **I6** | 测试通过 | 检查测试运行结果 | 全部测试通过 |

I5 和 I6 在 Build-Fix 循环中已隐式验证，此处为显式确认并记录结果。
不变性门验证结果追加到 `docs/修改记录/` 或 Phase 制品中。

## 自检

- [ ] 所有计划任务已执行并打勾
- [ ] FE/BE 子代理都已返回
- [ ] **I5 不变性门（编译通过）已验证**
- [ ] **I6 不变性门（测试通过）已验证**
- [ ] I5/I6 验证结果已记录到 Phase 制品
- [ ] IDE 编译验证通过
- [ ] 如编译/测试失败，已执行 build-fix 循环
- [ ] build-fix 未超过 3 轮上限
- [ ] 影响范围自检清单已确认
- [ ] 测试流程已获用户确认
- [ ] 修改范围未超出规格 Scope

## 约束

- 严格按计划执行，不自行增删任务。最小修改（不改无关代码）。不改规格。不引入新依赖（除非计划中明确）。
- **用户确认前置**：5.3.5.0 未获用户确认测试流程不得执行测试。
- **Phase 5.5 退回处理**：如审核发现代码问题退回 Phase 5，优先修复退回项，依次处理。如修复涉及计划外任务，在修改记录中标注"Phase 5.5 退回修复"并简要说明原因，无需更新实施计划。最小修改原则在退回修复时略微放宽，允许修复与退回项直接相关的紧邻代码异味。
- **不变性门不可跳过**：I5/I6 在 Build-Fix 循环通过后必须显式确认并记录。

### Phase 出口（Read `scripts/phase-entry-exit.md` 出口流程）

1. 更新 `artifacts.code_changes` 列表（记录每个修改文件的 path/type/summary）
2. 参数: current_phase="code", next_phase="api-doc"
3. 额外：将 I5/I6 不变性门结果写入 `metrics_snapshot.gate_2_5`
