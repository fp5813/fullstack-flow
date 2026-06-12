---
name: fullstack-flow/java-dev-standards
description: "Java 后端开发规范（fullstack-flow 子技能）— Spring Boot 分层架构、编码规范、异常处理、日志规范、配置管理。Agent BE 在 Phase 5 时必需加载。"
user-invocable: false
---

# Java 后端开发规范

## 快速参考

| 规范 | 内容 | 适用 Phase |
|------|------|:----------:|
| 后端架构 | Spring Boot 分层 + 目录结构 | Phase 4 |
| 编码规范 | 命名/API 风格/事务/异常 | Phase 5 BE |

> **通用层**：本规范与项目无关，适用于任何 Spring Boot 项目。
> **本地化层**：项目特定约定（包名、API 前缀、数据源等）通过 `CODEBUDDY.md` 注入。
>
> 使用时同时读取本文件和 `CODEBUDDY.md`。

---

## 1. Spring Boot 开发规范

### 1.1 分层架构

```
{module}/
├── controller/       ← REST API 入口，轻薄，仅做参数校验+调用Service
│   └── {subdomain}/  ← 按业务子域分包
├── service/          ← 业务接口 + impl 实现
│   └── impl/
├── mapper/           ← MyBatis-Plus Mapper（或 JPA Repository）
├── entity/
│   ├── po/           ← 持久化对象（@TableName/@TableId）
│   ├── dto/          ← 传输对象（跨服务/跨层）
│   ├── request/      ← 请求体（@Valid 校验注解）
│   ├── response/     ← 响应体（VO 视图对象）
│   └── vo/           ← 视图对象（复杂聚合查询）
├── enums/            ← 枚举常量
├── config/           ← Spring 配置类（@Configuration）
├── exception/        ← 业务异常枚举 + 异常处理器
├── util/             ← 工具类
├── constant/         ← 常量定义
├── redis/            ← Redis 服务封装
├── annotation/       ← 自定义注解
├── interceptors/     ← 请求拦截器
└── Application.java  ← 启动入口
```

> **实际包名** 参考项目 `CODEBUDDY.md` 中的基础包路径。

### 1.2 分层职责

| 层 | 职责 | 禁止 |
|----|------|------|
| **Controller** | HTTP 入参校验、调用 Service、日志记录（开始/结束/耗时） | 不含业务逻辑 |
| **Service** | 业务编排、事务管理、调用 Mapper/第三方 | 不处理 HttpServletRequest |
| **Mapper** | 数据访问（MyBatis-Plus CRUD / 自定义 XML） | 不含业务逻辑 |
| **Entity** | 数据模型、ORM 映射注解 | 不含业务方法 |

### 1.3 Controller 规范

#### 类注解模板

```java
@Slf4j
@RestController
@Tag(name = "{模块名}")
@RequestMapping("/api/{scope}/{resource}")
@Validated
public class {Entity}Controller {
    private final {Entity}Service {entity}Service;

    public {Entity}Controller({Entity}Service {entity}Service) {
        this.{entity}Service = {entity}Service;
    }
}
```

- `@Slf4j` — 日志记录
- `@RestController` — REST 控制器
- `@Tag(name = "...")` — Swagger/OpenAPI 分组
- `@RequestMapping` — API 路径前缀（参考 CODEBUDDY.md 获取前缀约定）
- `@Validated` — 类级别参数校验
- **构造器注入**（非 `@Autowired`）

#### 方法注解模板

```java
@PostMapping("/create")
@Operation(summary = "创建记录", description = "xxx")
@AsyncLog(resName = "创建记录", resTitle = "创建记录", opType = LogOpType.ADD)
public BaseResult<Entity> create(@Valid @RequestBody CreateRequest request) {
    long start = System.currentTimeMillis();
    log.info("{}.create.start ...", domain);
    Entity result = service.create(request);
    log.info("{}.create.success id={} costMs={}", domain, result.getId(), elapsed(start));
    return new BaseResult<>(result);
}
```

#### 方法设计规范

| 操作 | HTTP | 参数位置 | 返回 | 日志操作类型 |
|------|------|---------|------|------------|
| 分页查询 | `@PostMapping("/page")` | `@RequestBody PageParam<Query>` | `BaseResult<Page<VO>>` | QUERY |
| 单条查询 | `@GetMapping("/detail")` | `@RequestParam` | `BaseResult<VO>` | QUERY |
| 新增 | `@PostMapping("/create")` | `@Valid @RequestBody` | `BaseResult<Entity>` | ADD |
| 编辑 | `@PostMapping("/update")` | `@Valid @RequestBody` | `BaseResult<Entity>` | UPDATE |
| 删除 | `@PostMapping("/delete")` | `@Valid @RequestBody` | `BaseResult<DeleteVO>` | DEL |
| 状态变更 | `@PostMapping("/{action}")` | `@Valid @RequestBody` | `BaseResult<VO>` | UPDATE |

> PUT/DELETE 建议使用 `@PostMapping` + action 名，而非 RESTful 动词，
> 具体约定查看 `CODEBUDDY.md` 中的 API 风格。

#### 参数校验

```java
// 类级别
@Validated

// 方法参数
@Valid @RequestBody CreateRequest request
@Valid @RequestBody(required = false) PageParam<Query> request
@NotBlank(message = "订单号不能为空") @RequestParam("orderNo") String orderNo

// Request 类内字段
@NotBlank(message = "名称不能为空") private String name;
@NotNull(message = "数量不能为空") private Integer quantity;
@Min(value = 1, message = "数量必须为正数") private Integer count;
```

#### 统一响应体

```java
// 通用响应包装 BaseResult<T>
// 字段：statusCode, msg, traceId, result

// 成功
return new BaseResult<>(data);
// 或
return BaseResult.success(data);

// 失败
return BaseResult.fail("错误消息");
return BaseResult.fail("CODE", "错误消息");

// 分页
Page<VO> page = service.page(query);
return new BaseResult<>(page);
```

### 1.4 Service 规范

#### 接口

```java
public interface {Entity}Service extends IService<{Entity}> {
    Page<VO> page(PageParam<Query> req);
    VO get(Long id);
    Entity create(CreateRequest req);
    Entity update(UpdateRequest req);
    void delete(DeleteRequest req);
}
```

#### 实现

```java
@Service
public class {Entity}ServiceImpl extends ServiceImpl<{Entity}Mapper, {Entity}>
        implements {Entity}Service {

    private final OtherService otherService;

    public {Entity}ServiceImpl(OtherService otherService) {
        this.otherService = otherService;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Entity create(CreateRequest req) {
        // 1. 参数校验（业务校验）
        // 2. 组装 Entity
        // 3. baseMapper.insert(entity)
        // 4. 关联操作
        // 5. 返回结果
    }
}
```

#### 事务规范

| 场景 | 注解 | 说明 |
|------|------|------|
| 单表单行写入 | 不强制要求 | `baseMapper.insert/updateById` 单行 |
| 多表操作 | `@Transactional(rollbackFor = Exception.class)` | 写操作 |
| 批量写入 | `@Transactional(rollbackFor = Exception.class)` | 循环写必须在事务内 |
| Service 调用 Service | 建议在外部 Service 入口加事务 | 避免事务传播问题 |

### 1.5 Entity 规范

```java
@Data
@TableName("table_name")
public class Entity {
    /** 主键ID */
    @TableId(value = "id", type = IdType.AUTO)
    private Long id;

    /** 创建时间 */
    @TableField("created_at")
    private LocalDateTime createdAt;

    /** 更新时间 */
    @TableField("updated_at")
    private LocalDateTime updatedAt;

    // 业务字段
    @TableField("user_id")
    private String userId;

    /** 逻辑删除标志（0 未删 1 已删） */
    @TableField("deleted")
    private Integer deleted;

    /** 扩展信息，JSON 字符串 */
    @TableField("extra")
    private String extra;
}
```

| 要素 | 规范 |
|------|------|
| 类注解 | `@Data` + `@TableName` |
| 主键 | `@TableId(value = "id", type = IdType.AUTO)` |
| 字段映射 | `@TableField("snake_case_col")` |
| 非 DB 字段 | `@TableField(exist = false)` |
| 枚举字段 | `@TableField("status")` 使用 `Integer` 存 code |
| 命名一致 | Java camelCase ↔ DB snake_case 显式映射 |

### 1.6 异常处理

#### 业务异常抛出

```java
// Service 层抛出
throw new BusinessServiceException("错误消息");
throw new BusinessServiceException("CODE", "错误消息");
```

#### 全局异常处理器

框架层统一捕获：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    // BusinessServiceException → 返回 BaseResult.fail(code, msg)
    // MethodArgumentNotValidException → 参数校验失败
    // SQLException → 数据库异常
    // RedisException → 缓存异常
    // 其他 Exception → 500 系统内部异常
}
```

#### 错误码枚举

```java
@Getter
public enum ErrorCode {
    SUCCESS("0", "操作成功"),
    PARAM_ERROR("400", "参数不正确"),
    AUTH_ERROR("401", "鉴权失败"),
    PERMISSION_ERROR("403", "权限不够"),
    DB_ERROR("600", "数据库操作失败"),
    SYSTEM_ERROR("900", "系统内部异常");
}
```

### 1.7 日志规范

#### 格式

```java
// 每接口记录开始 + 结束日志
log.info("{domain}.{action}.start key1={} key2={}", val1, val2);
// ... 业务逻辑 ...
log.info("{domain}.{action}.success key1={} costMs={}", val1, elapsed(start));
```

#### 操作日志（自定义注解）

```java
@AsyncLog(resName = "业务描述", resTitle = "操作标题", opType = LogOpType.ADD)
```

| 日志类型 | 用途 | 要求 |
|---------|------|------|
| `log.info` | 业务日志（开始/结束/耗时） | 每接口必加 |
| `@AsyncLog` | 操作审计日志（入库持久化） | 按需 |
| `log.warn` | 业务异常告警 | 异常处理器统一处理 |
| `log.error` | 系统异常告警 | 异常处理器统一处理 |

### 1.8 配置管理

- 多 Profile：`application-{profile}.yml`（local / test / uat / prod）
- 敏感信息：通过环境变量注入，不硬编码
- 公共配置：`application.yml`（跨 profile 共享）

---

## 2. 自检清单

- [ ] Controller 注解模板与实际项目一致
- [ ] Service 继承链正确（`Service` → `ServiceImpl<Mapper, Entity>`）
- [ ] Entity 注解完整（`@TableName` + `@TableId` + `@TableField`）
- [ ] 异常处理引用 `BusinessServiceException` + 错误码枚举

---

> **使用方式**：Phase 5 时与 `CODEBUDDY.md` 一起加载，合并项目特定约定生成代码。
> **代码审查**：Phase 5.5 时加载 `java-review-standards` 子技能查看审查清单。
