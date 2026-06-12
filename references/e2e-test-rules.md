---
name: fullstack-flow/references/e2e-test-rules
description: "E2E 测试规则 — API 层翻页/排序/过滤测试 + 影子流量验证"
version: 2.0.0
tags: [fullstack, reference, test, e2e, api]
---

# E2E 测试规则（API 层）

> 当修改涉及后端 API（Controller/Service/SQL）时，优先使用 API 层 E2E 测试而非 Playwright UI 测试。
> API 层测试更快、更稳定、不需要浏览器环境。

## 适用场景

| 场景 | 推荐测试方式 |
|------|------------|
| 后端 API 翻页/排序/过滤 | API 层 ✅ |
| 数据权限/SQL 正确性 | API 层 + MCP MySQL ✅ |
| **API 兼容性（影子流量）** | API 层新旧对比 ✅ |
| 前端 UI 交互逻辑 | Playwright UI ❌ |
| 前后端联调 | API 层 + Playwright 混合 |

## 最小测试用例（翻页场景）

### 基准数据准备

```powershell
# 用 MCP MySQL 获取列表基准数据
SELECT id, maintitle, fonds_no_code, archive_status_code, input_date
FROM {table}_wyj
WHERE fonds_no_code = '{target_fonds}'
  AND archive_status_code IN ({statuses})
ORDER BY {sort_column} {sort_order}, id {sort_order}
```

### 8 步标准用例

```
标题X 在列表位置=n
├── 下条 n+1 → 验证返回ID与 sorted[n+1] 一致
├── 上条 n-1 → 验证与下条互为逆序
├── 首条    → 验证返回 sorted[0]
├── 末条    → 验证返回 sorted[-1]
├── 首条→上条 → 验证循环回到末条
├── 末条→下条 → 验证循环回到首条
└── 连续下条×5 → 验证全部返回在预期范围内
```

## 排序确定性规则

翻页排序必须有确定性 tiebreaker，仅靠业务字段排序不够：

```sql
-- ✅ 正确（确定性）
ORDER BY input_date DESC, id DESC

-- ❌ 错误（非确定性，同值记录顺序由数据库内部决定）
ORDER BY input_date DESC
```

**验证方法**：连续 5 次下条的 ID 序列必须严格单调递减（`id DESC`）。

## 数据隔离验证矩阵

| 验证项 | 断言条件 | 测试方法 |
|--------|---------|---------|
| 全宗隔离 | 翻页结果 `fonds_no_code` 与基准一致 | 传入 queryCondition 或验证权限 |
| 状态隔离 | 翻页结果 `archive_status_code` 在目标范围内 | 不同 archiveStatus 参数 |
| 搜索过滤 | 翻页结果不超出 `queryCondition` 条件 | 带 queryCondition 参数 |
| 目录序列 | 翻页结果不超出 `sqlCondition` 条件 | 带 sqlCondition 参数 |
| 排序一致性 | 翻页顺序与 listData 排序一致 | 逐条对比 |

## 断言模式

| 标记 | 含义 |
|------|------|
| `✅` | 返回记录ID = 预期ID，且数据范围合法 |
| `❌ 跨全宗` | 返回了非目标全宗的记录 |
| `❌ 跨状态` | 返回了非目标状态码的记录 |
| `❌ 顺序错误` | 返回记录ID ≠ 预期ID（排序不匹配） |
| `ℹ️` | 返回自身ID（提示"数据到头了"） |

## 测试脚本模板

```powershell
$token = "<登录token>"
$headers = @{"X-Access-Token"=$token;"Content-Type"="application/json"}
$base = "http://localhost:3100/archive-api/action/archiveBusinessDataAction"

# 1. 获取基准列表
$list = Invoke-RestMethod -Uri "$base/listData" -Method Post -Headers $headers -Body (@{
  entityId="<entityId>"; archiveStatus="<status>"; pageNo=1; pageSize=1000
  sqlCondition=@{}; queryCondition=@{}; column="input_date"; order="desc"
} | ConvertTo-Json)
$sorted = $list.result.records | Sort-Object -Property input_date, id

# 2. 从第n条开始测试翻页
$startId = $sorted[$n].id
for ($i=0; $i -lt 5; $i++) {
  $res = Invoke-RestMethod -Uri "$base/getData" -Method Post -Headers $headers -Body (@{
    id=$startId; entityId="<entityId>"; sqlCondition=@{}
    action="next"; archiveStatus="<status>"
  } | ConvertTo-Json)
  $got = $res.result
  # 断言...
  $startId = $got.id
}
```

## 6. 影子流量验证（API 兼容性对比）

> 当修改涉及 API 响应格式变更（字段增删、类型变化、数据结构调整）时，
> 使用影子流量模式对比新旧接口行为，确保向后兼容。

### 适用场景

| 场景 | 验证重点 |
|------|---------|
| 修改现有 API 的响应字段 | 字段增删、类型变化 |
| 修改排序/过滤逻辑 | 排序确定性、过滤条件一致性 |
| 修改数据聚合逻辑 | 聚合结果一致性 |
| 修改分页逻辑 | 分页参数行为一致性 |

### 验证流程

```
Phase 2 探路 → 记录旧接口样本（保存响应 JSON）
  ↓
Phase 5 实现 → 用相同参数调用新接口
  ↓
Phase 5.5 审核 → 对比新旧响应 JSON 结构差异
  ↓
差异判定 → 新增字段 ✅ / 删除字段 ❌ / 类型变化 ❌ / 值变化 ⚠️
```

### 结构对比规则

| 对比结果 | 判定 | 处理方式 |
|---------|:----:|---------|
| 新增字段（旧无新有） | ✅ 向后兼容 | 无需处理 |
| 删除字段（旧有新无） | ❌ 破坏兼容 | 标注需确认，如非有意则为 BUG |
| 类型变化（同名字段类型不同） | ❌ 破坏兼容 | 标注需确认 |
| 值变化（同参数返回值不同） | ⚠️ 需人工确认 | 需确认是否为预期行为 |
| 排序顺序变化 | ⚠️ 需人工确认 | 需确认是否影响前端展示 |

### Python 测试脚本模板

```python
import requests
import json
import sys

def shadow_verify(api_path, params, old_sample_path):
    """
    影子流量验证：对比新旧接口响应

    参数:
        api_path: API 路径（如 /api/user/list）
        params: 请求参数字典
        old_sample_path: 旧接口响应样本文件路径（JSON）
    """
    print(f"=== 影子流量验证: {api_path} ===")
    token = login("admin", "admin123")
    headers = {"Authorization": f"Bearer {token}"}

    # 加载旧接口样本
    with open(old_sample_path, 'r') as f:
        old_response = json.load(f)

    # 调用新接口
    resp = requests.get(f"{BASE_URL}{api_path}",
        params=params, headers=headers)
    new_response = resp.json()

    # 结构对比（仅对比 result 层字段）
    old_result = old_response.get("result", {})
    new_result = new_response.get("result", {})

    if isinstance(old_result, dict) and isinstance(new_result, dict):
        old_fields = set(old_result.keys())
        new_fields = set(new_result.keys())

        added = new_fields - old_fields
        removed = old_fields - new_fields

        if removed:
            print(f"  ❌ 字段被删除: {removed}")
            sys.exit(1)
        if added:
            print(f"  ✅ 新增字段: {added}")
        print("PASS: 影子流量验证通过")
    else:
        print("  ⚠️ result 非 dict 类型，跳过字段级对比")
```

### 审查清单

- [ ] 涉及 API 响应格式变更时已执行影子流量验证
- [ ] 新旧接口对比结果已记录到审核报告
- [ ] 破坏兼容的变更已标注并在规格中声明

## 变更记录

| 版本 | 日期 | 变更 |
|:----:|------|------|
| 2.0.0 | 2026-06-12 | 新增第 6 章影子流量验证（API 兼容性对比 + Python 测试脚本模板） |
| 1.0.0 | 2026-06-07 | 初始版本：8 步翻页标准用例 + 排序确定性 + 数据隔离矩阵 |
