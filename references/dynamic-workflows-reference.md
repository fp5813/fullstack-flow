# 动态工作流（Dynamic Workflows）参考

> 根据具体任务即时编排定制化的子 Agent 执行框架，支持多种并行/协作模式。

## 快速参考

| 模式 | 英文 | 适用场景 | Phase 2 | Phase 4 | Phase 5 |
|------|------|---------|---------|---------|---------|
| 分类并执行 | classify | BUG 修复 | 定向探路 | 计划+验证并行 | 实现+测试并行 |
| 分发并汇总 | distribute | 新功能（默认） | 4 Agent 全覆盖 | 计划+验证并行 | 实现+测试并行 |
| 对抗性验证 | adversarial | 优化/安全 | +安全扫描 | 计划+验证并行 | 实现+验证循环 |
| 生成并筛选 | generate | 重构/选型 | 多角度收集 | 计划+验证并行 | 多方案对比+测试并行 |

## 概述

fullstack-flow 内置 4 种工作流模式，Phase 1 澄清后根据任务类型自动选择最优模式。用户也可通过 `/workflow-pattern` 手动切换。

## 模式选择逻辑

Phase 1 完成后，根据 `task.type` 自动匹配：

| 任务类型 | 推荐模式 | 说明 |
|---------|---------|------|
| `bug` | **分类并执行** | 精准定位问题范围，定向修复 |
| `feature` | **分发并汇总** | 多角度探路 + 拆分并行实现 |
| `refactor` | **生成并筛选** | 多种方案对比，选最优 |
| `optimization` | **对抗性验证** | 改一处 → 验证性能/安全不受损 |
| `complex` | **分发并汇总** | 复杂功能拆解，多 Agent 并行 |
| 用户手动指定 | 任意 | `/workflow-pattern <模式>` 切换 |

## 模式 1: 分类并执行（Classify & Execute）

**适用**: BUG 修复、范围明确的任务  
**流程**: 先分类 → 并行计划+验证 → 并行实现+测试

```
Phase 1 分类
    ↓
确定 BUG 类型（前端/后端/数据/权限）
    ↓
Phase 2 定向探路（只探相关层）
    ↓
Phase 4 并行子代理:
  Agent Plan: 实施计划（T### 任务分解）
  Agent Test: 测试验证计划（V### 任务分解 + 测试数据准备）
    ↓ 同时输出实施计划 + 测试验证计划
    ↓
Phase 5 并行子代理:
  Agent Impl: 代码实现
  Agent Test: TDD 测试执行（按 V### 逐一验证）
    ↓
Phase 5.5 回归验证
```

**子 Agent 编排**:
```
Agent FE: 前端探路（组件/路由/API 调用）
Agent BE: 后端探路（Controller/Service/DAO）
Agent Data: 数据验证（表结构/数据行）
    3 个 Agent 并行 → 结果汇总到探路报告
```

## 模式 2: 分发并汇总（Distribute & Aggregate）

**适用**: 新功能开发、复杂多模块任务  
**流程**: 拆解 → 分派并行 → 实现+测试并行 → 汇总

```
Phase 1 + 2 整体探路
    ↓
Phase 4 并行子代理:
  Agent Plan: 实施计划（T### 任务分解）
  Agent Test: 测试验证计划（V### 任务分解 + 条件分支/数据类型覆盖检查）
    ↓ 同时输出实施计划 + 测试验证计划
    ↓
Phase 5:
  Agent 1: 模块 A 实现 ─┐
  Agent 2: 模块 B 实现 ─┤→ 汇总合并
  Agent 3: 模块 C 实现 ─┘
  Agent Test: 并行执行 V### 测试验证 ← 新增
    ↓
Phase 5.5 整体审核
```

**汇总规则**:
- 每个子 Agent 独立产出代码变更
- 主 Agent 检查冲突（同一文件修改 → 合并策略）
- 冲突无法自动解决 → 标记"需人工合并"
- 汇总到 `code_changes[]` 数组

## 模式 3: 对抗性验证（Adversarial Verification）

**适用**: 优化、重构、安全敏感修改  
**流程**: 并行计划+验证 → 实现 → 独立审核 → 反馈循环

```
Phase 4 并行子代理:
  Agent Plan: 实施计划
  Agent Test: 测试验证计划（含安全验证点）
    ↓
Phase 5 实现 Agent 输出代码
    ↓
Phase 5.5 审核 Agent 独立审查
    ├─ 通过 → 进入 Phase 5.6
    └─ 发现 N 个问题 → 返回 Phase 5 修复
         ↓
        实现 Agent 修复 → 审核 Agent 重新审查
         ↓
        最多 3 轮循环 → 仍有问题则标记"需人工介入"
```

**审核 Agent 规则**:
- 审核 Agent 不得是同一个 Agent 实例（独立上下文）
- 审核维度：AC 覆盖 / 代码质量 / 边界条件 / 向后兼容
- 发现问题必须引用具体行号
- 3 轮后未解决 → 输出差异报告给用户

## 模式 4: 生成并筛选（Generate & Filter）

**适用**: 重构、技术选型、复杂算法  
**流程**: 多方案生成 → 比较 → 选择 → 测试验证

```
Phase 2 探路确定约束条件
    ↓
Phase 4 并行子代理:
  Agent Plan: 实施计划（多方案对比任务）
  Agent Test: 测试验证计划（每方案对应 V###）
    ↓
Phase 5:
  Agent 1: 方案 A（以性能优先）
  Agent 2: 方案 B（以可读性优先）
  Agent 3: 方案 C（以最小改动优先）
    ↓
主 Agent 对比 3 方案
    ├─ 推荐方案 + 理由
    └─ 输出对比表
    ↓
用户选择 → 采纳方案进入 Phase 5.6
    ↓
Agent Test: 执行选中方案的 V### 测试验证
```

**对比维度**: 改动量 / 性能影响 / 可维护性 / 向后兼容 / 风险等级

## Agent 编排协议

所有模式共享以下编排协议：

### 子 Agent 输入规范

主 Agent 派发时提供结构化输入：

```markdown
## 任务上下文
- **项目根目录**: {project-root}
- **任务类型**: {bug/feature/refactor/optimization}
- **工作流模式**: {模式名称}
- **目标文件**: {文件路径列表}
- **约束条件**: {技术约束 / 不能改的范围}
- **验收标准**: {AC 清单}
- **测试验证计划**: {V### 任务清单}
- **参考文档**: {探路报告 / 规格文档路径}
```

### Agent Test 专项规范

**Agent Test** 是新增的并行测试验证子 Agent，在 Phase 4 和 Phase 5 中与实现 Agent 并行工作：

#### Phase 4 Agent Test 职责
- 读取规格文档的"方案设计与验证策略"章节
- 提取条件分支矩阵、测试数据类型清单、AC 清单
- 生成 V### 测试验证任务（与 T### 实现任务并行输出）
- 检查测试数据覆盖完整性

#### Phase 5 Agent Test 职责
- 在实现 Agent 写代码的同时，按 V### 清单准备测试数据和测试脚本
- 实现 Agent 完成后，立即执行对应 V### 验证
- 输出验证结果（通过/失败 + 失败原因）
- 与实现 Agent 的代码在同一轮 Build-Fix 循环中修复

#### 输入输出

| 阶段 | 输入 | 输出 |
|:----:|------|------|
| Phase 4 | 规格文档（方案设计与验证策略章节） | V### 任务清单 + 测试数据类型准备清单 |
| Phase 5 | V### 任务清单 + 实现代码 | 测试验证结果（逐条通过/失败） |

### 子 Agent 输出规范

子 Agent 完成后返回结构化结果：

```markdown
## 执行结果
- **Agent**: {名称}
- **状态**: {success/partial/failed}
- **修改文件**: [{文件1}, {文件2}]
- **验收通过**: {N/M}
- **问题**: {如有则列出}
```

### 汇总合并规则

| 场景 | 处理方式 |
|------|---------|
| 不同文件修改 | 直接合并 |
| 同一文件非重叠区域 | 按行号顺序合并 |
| 同一文件重叠区域 | 标记冲突 → 用户确认 |
| 失败子 Agent | 输出失败原因，其他 Agent 结果保留 |

## 使用方式

```
# 查看当前工作流模式
/workflow-pattern

# 切换到指定模式
/workflow-pattern distribute
/workflow-pattern adversarial
/workflow-pattern classify
/workflow-pattern generate

# 设置默认模式（写入 state.yaml）
/workflow-pattern --set-default distribute
```
