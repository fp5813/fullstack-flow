---
name: fullstack-flow/java-test-standards
description: "Java 后端测试规范（fullstack-flow 子技能）— Python 脚本方式，通过 requests 调用 API + 数据库查询 + 日志验证接口正确性。Agent BE-Test 在 Phase 5 TDD 时必需加载。"
user-invocable: false
---

# Java 后端测试规范

## 1. 测试理念与原则

| 原则 | 说明 |
|------|------|
| **不修改项目代码** | 测试脚本放在 `tests/api/` 下，不修改任何业务代码 |
| **API 层验证** | 只测 Controller 层接口，不测内部方法 |
| **数据库查询准备数据** | 优先通过 MCP MySQL 查询数据库获取已有数据，不满足时通过 API 调用创建补充 |
| **三步骤验证** | 调用 API → 查看响应 → 查询数据库确认 |
| **Python 脚本驱动** | 使用 `python tests/api/test_{entity}.py` 运行 |

> **本规范独立于 java-dev-standards**，测试 Agent 不需要加载开发规范。
> **环境要求**：Python 3.8+，`pip install requests`。

---

## 2. 目录结构与命名规范

```
项目根目录/
├── tests/
│   └── api/                    ← API 测试脚本目录
│       ├── test_order.py
│       └── test_user.py
├── src/                        ← 业务代码（不修改）
└── ...
```

| 规范 | 说明 |
|------|------|
| 命名规则 | `test_{entity}.py`（如 `OrderController` → `test_order.py`） |
| 运行方式 | `python tests/api/test_{entity}.py` |
| 环境依赖 | Python 3.8+，需安装 `requests` 库 |

> **环境检查**：运行测试前确认 `python --version` 和 `pip show requests` 正常。

---

## 3. 测试脚本结构

### 3.1 基本模板

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
    # === 1. 准备数据 ===
    # 优先查询数据库获取已有数据，不足时通过 API 创建补充
    existing_id = query_db("SELECT id FROM some_table WHERE ...")
    if existing_id:
        test_data_id = existing_id
    else:
        token = login("admin", "admin123")
        resp = requests.post(f"{BASE_URL}/api/xxx/create",
            json={"name": f"测试数据-{uuid.uuid4()}"},
            headers={"Authorization": f"Bearer {token}"})
        test_data_id = resp.json().get("result", {}).get("id")

    # === 2. 调用目标 API ===
    token = login("admin", "admin123")
    resp = requests.get(f"{BASE_URL}/api/xxx/detail",
        params={"id": test_data_id},
        headers={"Authorization": f"Bearer {token}"})

    # === 3. 验证响应 ===
    print(f"Response: {resp.text}")
    assert_contains(resp.text, '"statusCode":"0"')
    assert_contains(resp.text, '"result":')

    # === 4. 查询数据库验证数据状态 ===
    db_record = query_db(f"SELECT * FROM some_table WHERE id = '{test_data_id}'")
    print(f"DB Record: {db_record}")
    assert_contains(db_record, "expected_field_value")

    # === 5. 检查日志输出 ===
    # 控制台会输出 log.info 日志，人工确认：
    # xxx.detail.start ... / xxx.detail.success ... costMs=...

    print("PASS: 接口验证成功")

def login(username, password):
    """调用登录接口获取 token"""
    resp = requests.post(f"{BASE_URL}/api/auth/login",
        json={"username": username, "password": password})
    return resp.json().get("result", {}).get("token")

def assert_contains(text, expected):
    """断言字符串包含"""
    if expected not in text:
        print(f"FAIL: 期望值 '{expected}' 未找到")
        print(f"实际内容: {text}")
        sys.exit(1)

def query_db(sql):
    """通过 MCP MySQL 工具查询数据库"""
    # 实际测试时通过 MCP mysql-{project}.execute_query 执行
    # 此函数为占位实现
    print(f"[DB Query] {sql}")
    return None

if __name__ == "__main__":
    main()
```

### 3.2 完整示例

```python
"""
订单 API 测试脚本
运行: python tests/api/test_order.py
"""
import requests
import uuid
import sys

BASE_URL = "http://localhost:8080"

def main():
    # === 1. 查询数据库获取已有用户 ID ===
    user_id = query_db("SELECT id FROM sys_user WHERE username = 'admin'")
    if not user_id:
        resp = requests.post(f"{BASE_URL}/api/user/create",
            json={"username": "admin", "password": "admin123"})
        user_id = resp.json().get("result", {}).get("id")

    # === 2. 登录获取 token ===
    login_resp = requests.post(f"{BASE_URL}/api/auth/login",
        json={"username": "admin", "password": "admin123"})
    token = login_resp.json().get("result", {}).get("token")
    headers = {"Authorization": f"Bearer {token}"}
    print(f"Login OK, token={token[:20]}...")

    # === 3. 调用目标 API：创建订单 ===
    order_resp = requests.post(f"{BASE_URL}/api/order/create",
        json={"userId": user_id, "amount": 100},
        headers=headers)
    print(f"Order Response: {order_resp.text}")
    assert_contains(order_resp.text, '"statusCode":"0"')
    order_id = order_resp.json().get("result", {}).get("id")

    # === 4. 查询数据库确认订单已创建 ===
    db_order = query_db(f"SELECT * FROM orders WHERE id = '{order_id}'")
    print(f"DB Order: {db_order}")
    assert_contains(str(db_order), "amount=100")
    assert_contains(str(db_order), "status=0")

    # === 5. 查看控制台日志输出 ===
    # 预期看到:
    # order.create.start ...
    # order.create.success id=xxx costMs=yyy

    print("PASS: 订单创建成功")

def assert_contains(text, expected):
    if expected not in str(text):
        print(f"FAIL: 期望值 '{expected}' 未找到")
        print(f"实际内容: {text}")
        sys.exit(1)

def query_db(sql):
    """通过 MCP MySQL 工具查询数据库"""
    print(f"[DB Query] {sql}")
    return None

if __name__ == "__main__":
    main()
```

---

## 4. 数据准备规范

| 规则 | 说明 |
|------|------|
| **优先查询数据库** | 使用 MCP MySQL `execute_query` 查询系统中已有数据作为测试输入（如字典表、配置表、用户数据） |
| **API 创建补充** | 数据库中没有合适数据时，通过 API 调用链创建（先父级 → 再子级） |
| **UUID 标识** | 通过 API 创建的数据使用 `uuid.uuid4()` 或时间戳避免数据冲突 |
| **不直接 INSERT** | 禁止通过 SQL INSERT 直接写数据库，始终通过 API 创建数据 |
| **不修改项目代码** | 测试脚本放在 `tests/api/` 目录下，不修改任何业务代码 |

> **与旧规范的关键差异**：
> - 旧规范：必须通过 API 接口调用造数，不允许从数据库直接 INSERT
> - 新规范：**优先查询数据库获取已有数据**，不满足时通过 API 创建补充

### 4.1 按测试数据类型准备数据

| 数据类型 | 准备方式 | 示例 |
|---------|---------|------|
| **正常数据** | 查询数据库获取已有正常记录，或通过 API 创建标准数据 | `SELECT * FROM sys_user WHERE status=0 LIMIT 3` |
| **边界数据** | 通过 API 创建边界值数据（最大值、最小值、空值） | 创建长度为 1 / 255 的字符串字段 |
| **异常数据** | 通过 API 创建异常场景数据，或构造非法参数调用 | 传入不存在的 ID、缺失必填字段 |
| **状态/枚举数据** | 使用 `SELECT DISTINCT` 查询所有枚举值，逐一验证 | `SELECT DISTINCT status FROM sys_user` |
| **组合条件数据** | 同时满足多个过滤条件的数据（用于验证条件组合） | 特定状态 + 特定分类的数据 |

> 测试数据类型在 Phase 3 规格文档的"测试数据类型清单"中定义。
> Phase 5 TDD 阶段按此清单逐一准备对应数据。

---

## 5. TDD 流程

### 阶段一 Red：先写测试

1. 为每个涉及的 Controller 创建 `tests/api/test_{entity}.py`
2. 测试脚本定义"契约"（期望的接口行为）
3. 运行 `python tests/api/test_{entity}.py`
4. 预期结果：测试失败（因代码尚未修改，API 返回结果与预期不一致）
5. 记录测试失败输出，作为代码实现的输入

### 阶段二 Green：实现代码

1. 按测试定义的契约实现 Controller/Service/Mapper
2. 每次修改后运行测试确认通过
3. 代码实现到测试通过为止，不过度设计

### 阶段三 Refactor：重构优化

1. 测试通过后，如有必要对代码进行重构
2. 提取重复逻辑、优化命名
3. 重构后必须重新运行测试确认通过

### 并行执行中的 TDD

**distribute 模式（默认）**:
```
TDD 前置（全部 BE 测试先写）:
  Agent BE-Test: 为所有涉及的 Controller 创建 test_{entity}.py → Red 验证
    ↓
Agent FE: 前端实现        Agent BE: 后端实现
    ↓                          ↓
合并验证: 运行全部 test_{entity}.py → Green 通过
```

**已存在测试脚本**：追加新接口的测试函数，保留原有测试函数。运行全部测试确保回归通过。

---

## 6. 验证方法

### 6.1 验证步骤

每次测试按以下三步验证：

| 步骤 | 方法 | 验证内容 |
|------|------|---------|
| 1. 调用 API | `requests.post/get` | 接口可访问、不抛异常 |
| 2. 查看响应 | `assert_contains` 检查 JSON | 状态码、业务字段值正确 |
| 3. 查询数据库 | MCP MySQL `execute_query` | 数据持久化结果与预期一致 |

### 6.2 验证要点

| 验证方式 | 方法 | 适用场景 |
|---------|------|---------|
| 响应内容 | `print` + `assert_contains` | 检查接口返回的 JSON 字段 |
| 状态码 | `assert_contains(resp.text, '"statusCode":"0"')` | 确认业务成功 |
| 日志输出 | 查看控制台 `log.info` 输出 | 确认日志规范（开始/结束/耗时） |
| 数据库查询 | MCP MySQL `execute_query` | 确认数据持久化结果 |

### 6.3 错误处理

```python
if expected not in text:
    print(f"FAIL: 期望值 '{expected}' 未找到")
    print(f"实际响应: {text}")
    sys.exit(1)
```

---

## 6.5 属性测试模板（Property Testing）

> 属性测试验证"对于所有满足某条件的输入，某属性始终成立"，覆盖手工测试想不到的边界情况。

### 适用场景

| 场景 | 示例 | 验证属性 |
|------|------|---------|
| 排序/过滤 | 列表接口的排序参数 | 排序结果确定性（同一输入两次结果一致） |
| 数据转换/映射 | VO↔DTO 字段映射、枚举转换 | 转换前后数据一致性 |
| 数值计算 | 金额计算、统计汇总 | 计算结果的数学恒等式 |
| 字符串处理 | 格式化、拼接、截断 | 格式一致性（长度、字符集） |
| 分页 | 列表接口的分页参数 | 总条数一致性（page×size 不超总数） |

### Python 属性测试模板

```python
import random
import string
import sys

# ============ 随机数据生成辅助函数 ============

def random_string(max_len=20):
    """生成随机字符串"""
    length = random.randint(1, max_len)
    return ''.join(random.choices(string.ascii_letters + string.digits, k=length))

def random_status():
    """生成随机状态值"""
    return random.choice([0, 1, 2])

def random_enum(enum_values):
    """从枚举值列表中随机选一个"""
    return random.choice(enum_values)

# ============ 属性测试示例 ============

def test_property_sorting_stability():
    """
    属性测试：排序结果应该是确定性的
    （同一输入两次排序结果一致）
    """
    print("=== 属性测试: 排序确定性 ===")
    token = login("admin", "admin123")
    headers = {"Authorization": f"Bearer {token}"}

    for i in range(5):
        page = random.randint(1, 3)
        size = random.choice([10, 20, 50])

        resp1 = requests.get(f"{BASE_URL}/api/xxx/list",
            params={"page": page, "size": size},
            headers=headers)
        resp2 = requests.get(f"{BASE_URL}/api/xxx/list",
            params={"page": page, "size": size},
            headers=headers)

        if resp1.text == resp2.text:
            print(f"  ✅ 第 {i+1} 轮: 排序确定性通过")
        else:
            print(f"  ❌ 第 {i+1} 轮: 排序结果不一致")
            sys.exit(1)
    print("PASS: 排序确定性属性测试通过")


def test_property_pagination_consistency():
    """
    属性测试：分页结果总条数应一致
    （不同 page 参数的 total 字段应相同）
    """
    print("=== 属性测试: 分页一致性 ===")
    token = login("admin", "admin123")
    headers = {"Authorization": f"Bearer {token}"}

    totals = []
    for i in range(3):
        resp = requests.get(f"{BASE_URL}/api/xxx/list",
            params={"page": i + 1, "size": 10},
            headers=headers)
        data = resp.json()
        total = data.get("result", {}).get("total", 0)
        totals.append(total)

    if len(set(totals)) == 1:
        print(f"  ✅ 分页 total 一致: {totals[0]}")
    else:
        print(f"  ❌ 分页 total 不一致: {totals}")
        sys.exit(1)
    print("PASS: 分页一致性属性测试通过")
```

### 审查清单新增

- [ ] 数据转换/映射逻辑已添加属性测试（随机输入验证）
- [ ] 属性测试覆盖了边界值和异常值组合

---

## 7. 测试覆盖要求

| 场景 | 要求 |
|------|------|
| 正常流程 | 每个涉及修改的 Controller 至少 1 条 |
| 异常流程 | 参数校验失败、业务异常至少 1 条 |
| 边界值 | 空列表、最大值、最小值（按需） |
| **属性测试** | 涉及数据转换/排序/计算时，至少 1 条属性测试（随机输入验证） |

> **回归要求**：已存在的测试脚本中追加新函数时，必须运行全部已有函数确认回归通过。

---

## 8. 审查清单

### 8.1 测试完整性

- [ ] 每个涉及修改的 Controller 有对应 `tests/api/test_{entity}.py`
- [ ] 测试覆盖了新增接口的正常流程
- [ ] 测试覆盖了参数校验失败、业务异常等异常流程
- [ ] 已存在的测试脚本追加了新函数，运行全部测试回归通过
- [ ] 数据转换/映射逻辑已添加属性测试（随机输入验证）
- [ ] 属性测试覆盖了边界值和异常值组合

### 8.2 测试代码质量

- [ ] 测试脚本不修改任何业务代码
- [ ] 测试数据优先查询数据库获取，不满足时通过 API 创建补充
- [ ] 不使用 SQL INSERT 直接写数据库
- [ ] 验证包含"响应检查 + 数据库确认"两步
- [ ] 使用 `uuid.uuid4()` 避免数据冲突
- [ ] 测试数据按类型（正常/边界/异常/枚举/组合）分类准备
- [ ] 每个枚举值/状态值至少有一条测试覆盖

### 8.3 测试可执行性

- [ ] Python 3.8+ 环境可用（`python --version`）
- [ ] `requests` 库已安装（`pip show requests`）
- [ ] 测试可独立运行（不依赖其他测试的执行顺序）
- [ ] 控制台输出清晰的 PASS/FAIL 标识

---

> **使用方式**：Phase 5 时由 Agent BE-Test 在 TDD 流程中加载。
> Phase 5.5 时按第 8 章审查清单逐项审核测试代码。
> 第 6.5 章属性测试模板在涉及数据转换/排序/计算时按需加载。
