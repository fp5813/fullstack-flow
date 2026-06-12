<p align="center">
  <img src="https://img.shields.io/badge/version-26.0.0-blue.svg" alt="Version">
  <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License">
  <img src="https://img.shields.io/badge/phases-14-orange.svg" alt="Phases">
  <img src="https://img.shields.io/badge/optimizations-13-brightgreen.svg" alt="Optimizations">
</p>

<h1 align="center">fullstack-flow</h1>
<p align="center">
  <strong>质量门控驱动的规范全栈开发流程</strong><br>
  <em>受 <a href="https://github.com/chopratejas/headroom">headroom</a> 和 <a href="https://github.com/JuliusBrussee/caveman">caveman</a> 项目启发</em>
</p>

<p align="center">
  <a href="#核心特性">核心特性</a> •
  <a href="#快速开始">快速开始</a> •
  <a href="#流程概览">流程概览</a> •
  <a href="#13-项优化">13 项优化</a> •
  <a href="#设计哲学">设计哲学</a> •
  <a href="#目录结构">目录结构</a>
</p>

---

## 核心特性

| 特性 | 说明 |
|------|------|
| **门控驱动** | 每个阶段有质量门控，不满足 100% 通过率不进入下一阶段 |
| **并行探路** | 4+ 并行子 Agent 探路，提升效率 3-5 倍 |
| **不变性门** | 10 条核心不变性，可自动验证（grep/bash） |
| **测试驱动** | 规格阶段设计测试策略，计划阶段生成测试验证任务 |
| **知识沉淀** | 每次实施后自动提取可复用模式，形成自学习闭环 |
| **Token 优化** | 沟通规则压缩 ~75% 输出 token，可压缩知识文件 ~46% |
| **MCP 统一** | 通用 MCP 配置模板，覆盖全栈/后端/前端项目 |

## 快速开始

### 安装

```bash
cd your-project
mkdir -p .codebuddy/skills
git clone https://github.com/fp5813/fullstack-flow.git .codebuddy/skills/fullstack-flow
```

### 使用

在 CodeBuddy Code 中输入：

```
/fullstack-flow
```

CodeBuddy 将加载技能并启动 **Phase 1 描述澄清**，引导你完成整个开发流程。

### 验证安装

确保项目 `.mcp.json` 中配置了 MCP 服务，参考 [通用 MCP 配置模板](references/mcp-project-templates/GENERIC.md)。

## 流程概览

```
Phase 1  描述澄清      → Gate 0: 关键词充足
  ↓
Phase 2  代码探路      ← 4 并行子 Agent（codegraph + mysql）
  ↓
Phase 2.5 质量门控     → Gate 1: 不变性门(I1/I2/I9/I10) + 35 项检查
  ↓
Phase 3  规格澄清      ← 条件分支矩阵 + 测试数据类型清单 + 方案设计
  ↓
Phase 4  实施计划      ← 并行生成 T### (实现) + V### (测试验证)
  ↓
Phase 4.5 覆盖验证     → Gate 2: 不变性门(I3/I4) + 5 条件 AND
  ↓
Phase 5  编码实现      ← FE/BE 并行 + Build-Fix 循环 + 不变性门(I5/I6)
  ↓
Phase 5.6 API 文档     ← 自动生成接口文档
  ↓
Phase 5.5 代码审核     → Gate 3: 不变性门(I7/I8) + 响应结构校验 + E2E
  ↓
Phase 6  修改记录      → Gate 4
  ↓
Phase 6.7 复盘回顾     ← 五维分析 + Instinct 提取
  ↓
Phase 6.5 规则同步     ← 业务规则更新
  ↓
Phase 6.6 规则审计     → Gate 5: 6 并行 Agent 扫描
```

## 13 项优化

受 [headroom](https://github.com/chopratejas/headroom)（上下文压缩）和 [caveman](https://github.com/JuliusBrussee/caveman)（高密度沟通）项目启发，实施了以下优化：

| # | 优化 | 来源 | 说明 |
|:-:|------|------|------|
| 1 | **不变性门** | headroom | 10 条核心不变性，每条对应可自动验证的 grep/bash 命令 |
| 2 | **决定不做清单** | headroom | 5 类 22 条禁止操作，Agent 越界时记录并回退 |
| 3 | **影子流量验证** | headroom | API 兼容性对比，新旧接口响应结构差异自动检测 |
| 4 | **本地预检=CI预检** | headroom | env-check 扩展编译/TypeScript/Lint/测试预检 |
| 5 | **决策日志已排除方案** | headroom | 记录被否决选项及原因，避免重复提出 |
| 6 | **属性测试模板** | headroom | 排序确定性/分页一致性随机验证 |
| 7 | **多Agent并行审计** | headroom | 6 并行 Agent 扫描 6 类业务规则 |
| 8 | **沟通规则** | caveman | 3 级强度（normal/concise/ultra）+ 自动清晰化 |
| 9 | **通用MCP配置模板** | 自研 | 11 章模板 + 4 个项目示例（全栈/单体/前端）|
| 10 | **方案设计与验证策略** | 自研 | 条件分支矩阵 + 测试数据类型清单 |
| 11 | **测试验证计划** | 自研 | Step 2.5 并行生成 V### 测试任务 |
| 12 | **并行子代理架构** | 自研 | 4 种工作流模式均支持 Agent Test 并行验证 |
| 13 | **响应结构校验** | headroom | Phase 5.5 字段/类型/结构/空值 4 项比对 |

## 设计哲学

### 渐进式披露（Progressive Disclosure）

```
L1 入口层（始终加载）     SKILL.md + quick-start
L2 当前 Phase 层（按需）  当前 Phase 文档 + 脚本
L3 深入层（显式访问）     其他 Phase + References
```

### 动态匹配与加载

```
索引建立 → 意图匹配 → 动态补充 → {匹配度足够？}
    ↑                              ↓ 是
    └──────── 重新匹配 ←────────── 执行 → Phase 转换
```

### 单一职责

每个 Phase/Agent/脚本只解决一类任务，避免万能 Agent。

## 目录结构

```
.
├── SKILL.md                 # 主技能定义（52 条规则）
├── CHANGELOG.md             # 版本变更记录
├── phases/                  # 13 个 Phase 文档
├── scripts/                 # 7 个标准化脚本
│   ├── invariant-gates.md   # 不变性门验证
│   ├── env-check.md         # 环境预检 + CI 预检
│   ├── build-fix.md         # Build-Fix 循环
│   ├── phase-entry-exit.md  # 入口/出口协议
│   ├── spawn-probe-agents.md
│   ├── create-decision.md   # 决策日志
│   └── update-index.md      # INDEX 更新
├── references/              # 17 个参考文档
│   ├── out-of-scope.md      # 决定不做清单
│   ├── communication-rules.md # 沟通规则
│   ├── e2e-test-rules.md    # 影子流量验证
│   ├── mcp-tools-summary.md # MCP 工具汇总
│   ├── mcp-project-templates/ # MCP 配置模板
│   └── ...
├── java-dev-standards/      # Java 开发规范
├── java-review-standards/   # Java 审查清单
├── java-test-standards/     # Java 测试规范 + 属性测试
├── vue-standards/           # Vue 编码规范
├── code-explore/            # 代码探路子技能
├── onboard-project/         # 项目接入子技能
└── audit-flow/              # 流程审计子技能
```

## 依赖

- **CodeBuddy Code CLI** — 运行环境
- **MCP 服务**：`codegraph`（代码分析）、`mysql-{子项目名}`（数据库）、`api-fetcher`（API 调用）
- **项目类型**：Java Spring Boot + Vue 3（全栈）/ 纯后端

## 版本

当前版本：**v26.0.0**

详见 [CHANGELOG.md](CHANGELOG.md)

## 许可证

MIT — 详见 [LICENSE](LICENSE)
