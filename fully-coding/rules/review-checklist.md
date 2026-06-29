---
name: review-checklist
description: Step 6 轻量代码评审清单。默认覆盖所有规则维度，发现具体问题时再按需读取完整规则文件。
---

# Step 6 Review Checklist

> 本文件是评审入口清单，不替代完整规则。若某项命中风险，读取对应规则文件确认细节和修复要求。

## 输入

- `{ts}-userStory.md` 验收标准
- `{ts}-codingLog.md` Step 3-5
- 本次代码 diff
- `{{config.knowledge_dir}}/{{config.error_code_doc}}`
- 相关 `{{config.feature_impl_glob}}` 文档

## DBA 检查

| 维度 | 检查点 | 需要展开读取 |
|---|---|---|
| 表结构 | 字段类型、长度、是否可空、默认值、注释、审计字段 | `persistence-sql-cache-rules.md` |
| 索引 | 主键、唯一索引、联合索引顺序、查询条件覆盖 | `persistence-sql-cache-rules.md` |
| SQL/Mapper | Wrapper 优先；复杂 SQL 走 XML；同一聚合根禁止注解/XML 混用 | `persistence-sql-cache-rules.md` |
| 数据一致性 | 事务边界、唯一性、并发写入、幂等约束 | `persistence-sql-cache-rules.md`; `error-handling-rules.md` |
| Redis/缓存 | Key 前缀、TTL、数据结构、缓存一致性 | `persistence-sql-cache-rules.md` |
| DDL 风险 | 删除表/列、字段类型变更、数据迁移、不可逆变更 | `workflow-rules.md` §8; `git-rules.md` |

## 代码评审检查

| 维度 | 检查点 | 需要展开读取 |
|---|---|---|
| 需求满足 | 是否覆盖全部 AC、需求假设和歧义处理 | `userStory.md` |
| DDD 分层 | api/app/domain/infrastructure/dao 依赖方向；聚合根职责；领域服务边界 | `backend-ddd-layer-rules.md` |
| 新聚合根 | DTO、PO、Mapper、Aggregate、DomainService、Repository、ApplicationService、Controller 是否齐全 | `implementation-checklist-rules.md` |
| 编码规范 | 命名、泛型、Lombok、魔法值、日志、注释、类型安全 | `backend-code-standard-rules.md` |
| 接口契约 | 响应统一 `DataResponse`（禁自定义包装类）；`@RequestBody` 直接 DTO/Map（禁 `{api,version,data}` 包装）；DTO 字段 camelCase（禁 `@JsonNaming(SnakeCaseStrategy)` / `@JsonProperty(snake_case)`）；`convertValue` 关 `FAIL_ON_UNKNOWN_PROPERTIES` | `backend-code-standard-rules.md` |
| 异常/错误码 | `[L][CC][SS][D]`；优先已有枚举；新增错误码记录 Step 8 更新 | `error-handling-rules.md`; 错误码文档 |
| 安全 | SQL 注入、XSS、越权、硬编码密钥、敏感信息输出 | `security-rules.md` |
| 幂等性 | 写操作是否有幂等键、唯一约束、重试语义或明确说明无需幂等 | `workflow-rules.md`; `persistence-sql-cache-rules.md` |
| 性能 | N+1、循环远程调用、大对象未分页、慢查询风险 | `persistence-sql-cache-rules.md` |
| 测试 | 主路径、异常路径、边界条件、关键写操作幂等性测试 | 现有测试规范 |
| 兼容性 | 已发布 API、数据契约、事件契约是否破坏 | `workflow-rules.md` §6 |

## 分级

- `auto-fix`：默认级别。凡可通过修改代码、SQL、配置、测试或文档解决的问题，都标记为 auto-fix。
- `block`：仅限致命且无法自动修复的问题：可外部利用的重大安全漏洞、生产数据不可逆损坏风险、已发布接口无法兼容的破坏、需求矛盾无法判断。

判断原则：不确定时选 `auto-fix`；“改动量大”不是 block 理由。

## 输出要求

每个问题必须包含：

```markdown
#### 问题 N [auto-fix|block]
- 位置：`path/to/file:line`
- 现象：
- 风险：
- 修复方案：
```

最终结论：`通过` / `需自动修复（N 项）` / `阻塞（N 项）`。
