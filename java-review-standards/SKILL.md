---
name: fullstack-flow/java-review-standards
description: "Java 后端代码审查清单（fullstack-flow 子技能）— 6 类审查项：后端/安全/API/性能/数据完整性/文档。Agent BE 在 Phase 5.5 时加载。"
user-invocable: false
---

# Java 后端代码审查清单

> **使用方式**：Phase 5.5 时由 Agent BE 加载，按以下清单逐项审核后端代码。
> 开发规范参考 `java-dev-standards` 子技能。

---

## 1. 后端审查

- [ ] `@Transactional(rollbackFor = Exception.class)` — 默认只回滚 RuntimeException
- [ ] `BusinessServiceException` 替代 `RuntimeException` 抛出业务异常
- [ ] 写操作有 `@AsyncLog` 操作日志注解
- [ ] 每个接口有开始/成功/耗时日志（`log.info`）
- [ ] 构造器注入，没有 `@Autowired` 字段注入
- [ ] 参数校验：`@Valid` / `@NotBlank` / `@NotNull` 完整
- [ ] `BaseResult<T>` 统一返回（不用 `Map` / `String` 裸返回）
- [ ] `@RequestMapping` 路径与模块一致（参考 `CODEBUDDY.md`）

## 2. 安全审查

- [ ] 新增接口有权限注解（`@RequiresPermissions` / Shiro / Spring Security）
- [ ] MyBatis XML 中 `#{}` 替换 `${}`（防 SQL 注入）
- [ ] 用户 ID 从上下文中获取（`UserContext`），不从请求参数接收
- [ ] 敏感字段脱敏（密码、手机号、身份证）
- [ ] 文件上传限制类型和大小

## 3. API 设计审查

- [ ] 分页接口使用统一 `PageParam<T>` 包装参数
- [ ] 响应体使用 DTO/VO（不直接暴露 Entity）
- [ ] `@Operation(summary = "xxx")` 描述完整
- [ ] 向前兼容：新增字段不影响旧客户端
- [ ] 枚举值返回 code + name（不要裸 code）

## 4. 性能审查

- [ ] N+1 查询检测：循环内查数据库 → 改为批量预加载（尤其是多线程场景，参考 FC-005）
- [ ] 分页使用 MyBatis-Plus `Page` 对象
- [ ] 批量操作使用批量方法（`saveBatch` / `updateBatchById`）
- [ ] 大表查询有索引
- [ ] Feign 调用设置超时
- [ ] 数据管道字段有 null 保护（默认值兜底），不假设数据源字段非空（参考 FC-006）

## 5. 数据完整性审查

- [ ] 新建表时评估联合唯一索引需求（尤其是用于分组统计的字段组合，参考 FC-001）
- [ ] 幂等性场景有明确的 DB 层约束保护（唯一索引比业务层更可靠，参考 FC-001）
- [ ] 数据写入采用先查后写/upsert 模式，避免 delete→re-insert（参考 FC-001/FC-004）
- [ ] 清理/删除类方法不添加过度过滤条件，确保所有遗留记录被处理（参考 FC-003）
- [ ] 兜底/异常处理包含足够诊断信息，能区分不同异常来源（参考 FC-007）
- [ ] 规避使用异常控制业务正常路径（try-catch 中不应含预期业务路径，参考 FC-004）
- [ ] 代码注释与实现逻辑逐行核对，注释声明的行为必须有对应代码（参考 FC-002）

## 6. 文档审查

- [ ] Controller `@Tag` + `@Operation` 注解完整
- [ ] Entity 字段 `@Schema(description = "xxx")` 完整
- [ ] `CODEBUDDY.md` 中的项目约定全部遵循

---

## 7. 审查自检

- [ ] 每类审查项已逐项核对
- [ ] 发现问题已记录到审核报告
- [ ] 标注了审核日期
