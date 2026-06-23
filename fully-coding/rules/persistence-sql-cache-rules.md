---
name: persistence-sql-cache-rules
description: 持久层与缓存规范。定义 Mapper SQL 风格、分页/索引/批量操作、慢 SQL 审查、缓存策略、缓存一致性及 Redis Key 规范。
---

# 持久层与缓存规范

> 基于 MyBatis-Plus 的持久层开发规范，以及 Redis 缓存使用策略。

---

## 1. Mapper SQL 与持久层规范

### 1.1 SQL 风格优先顺序

同一聚合根内保持统一，优先顺序：

1. **MyBatis-Plus Wrapper API**：单表 CRUD、简单条件查询
2. **XML**：多表 JOIN、复杂动态 SQL、分页查询，统一放在 `resources/mapper/` 下
3. **`@Select/@Update` 注解**：仅用于极简单的单条 SQL，且该聚合根内无 XML 时可用

**禁止**：同一聚合根的 Mapper 中混用注解与 XML 风格。

### 1.2 聚合根与 Mapper 的对应关系

- 一个聚合根 → 一个主表 → 一个 Mapper
- Join 其他聚合根的查询，SQL 放在**发起查询的聚合根**的 XML 中
- 复杂报表/统计查询，不归到任何聚合根 Mapper，单独建 `XxxQueryMapper`

### 1.3 持久层命名规范

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| 持久化对象 | `{表名}Po` | `UserPo`、`OrderItemPo` |
| Mapper 接口 | `{表名}Mapper` | `UserMapper`、`OrderItemMapper` |
| 表名 | `{服务名}_{业务}` | `demo_user`、`trade_order` |
| PO 字段 | 驼峰命名 | `createdAt`、`updatedAt` 作为审计字段 |
| Mapper XML | 与 Mapper 同名同路径 | 放在 `resources/mapper/` 下 |

PO / DTO / VO 后缀必须使用驼峰命名（同 `backend-code-standard-rules.md` §2.2）。

### 1.4 分页规范

- 所有列表查询必须带分页，默认 `pageSize = 20`，最大 `pageSize = 200`
- 禁止在内存中分页（先全量查再 `subList`）
- 分页参数统一使用 `PageRequest` / `PageDto`

```java
public Page<UserDto> queryUserPage(UserPageRequest request) {
    Page<UserPo> page = new Page<>(request.getPageNum(), request.getPageSize());
    LambdaQueryWrapper<UserPo> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(UserPo::getStatus, request.getStatus());
    return userMapper.selectPage(page, wrapper);
}
```

### 1.5 索引规范

- 主键、外键、业务唯一键必须建索引
- 索引命名：`idx_{表名}_{字段组合}`，唯一索引：`uk_{表名}_{字段组合}`
- 联合索引字段顺序遵循「最左前缀」原则，区分度高的放前面
- 禁止对低基数字段（如 status = 0/1）单独建索引

### 1.6 批量操作规范

- 单次批量插入/更新数量不超过 1000 条
- 优先使用 MyBatis-Plus `saveBatch()` / `updateBatchById()`
- 大数据量导入采用分片 + 异步任务

### 1.7 慢 SQL 与 SQL 审查

- 线上慢 SQL 阈值：单条执行超过 500ms
- 复杂查询上线前必须 `EXPLAIN` 检查索引命中
- 禁止 `SELECT *`，必须显式指定字段
- 禁止在 WHERE 中对字段做函数运算（导致索引失效）

---

## 2. 缓存策略

### 2.1 缓存失效时机

写操作完成后必须清理相关读缓存，防止脏读。

- `create` / `update` / `delete` 后清理聚合根缓存
- 批量操作后清理列表缓存
- 使用 `@CacheEvict` 或显式 Redis 删除

### 2.2 缓存注解位置

**禁止在 DomainService 中使用缓存注解**，缓存是应用层/查询层的横切关注点。

```java
@Service
@RequiredArgsConstructor
public class MemberAccountApplicationService {

    @Transactional
    @CacheEvict(value = "member:profile", key = "#memberId")
    public void updateProfile(Long memberId, UpdateProfileRequest request) {
        memberAccountDomainService.updateAccountProfile(memberId, ...);
    }
}
```

### 2.3 缓存 Key 命名规范

统一使用 `{服务}:{聚合}:{业务标识}` 格式：

```text
member:profile:{memberId}
member:list:{page}:{size}
order:detail:{orderId}
```

- 避免使用通配符作为 Key
- 列表缓存 Key 必须包含分页参数
- 热点对象单独缓存，避免大对象缓存

### 2.4 TTL 策略

| 数据类型 | 建议 TTL | 说明 |
|----------|----------|------|
| 用户基础资料 | 5~15 分钟 | 读多写少，可接受短暂不一致 |
| 配置 / 字典 | 1~24 小时 | 变更频率低 |
| 列表 / 分页 | 1~5 分钟 | 数据变化较快 |
| 幂等键 | 24 小时 | 与业务幂等窗口一致 |

### 2.5 缓存穿透 / 击穿 / 雪崩防护

| 问题 | 防护手段 |
|------|---------|
| 穿透 | 缓存空值（短 TTL）+ 布隆过滤器 |
| 击穿 | 互斥锁（`SETNX`）或逻辑过期 |
| 雪崩 | 随机 TTL、限流、多级缓存、集群高可用 |

### 2.6 缓存与数据库一致性

- **Cache Aside（推荐）**：读时先查缓存，未命中查库并回写；写时先更新数据库，再删除缓存
- 避免先删缓存再写库（高并发下易出现脏读）
- 强一致场景：使用分布式锁或延迟双删策略

---

## 3. Code Review 检查清单（持久层与缓存）

- [ ] 写操作后已清理相关缓存
- [ ] 缓存 Key 命名符合 `{服务}:{聚合}:{业务标识}` 规范
- [ ] 列表查询已带分页，且 pageSize 不超过上限
- [ ] 复杂 SQL 已确认索引命中
- [ ] 未使用 `SELECT *`
- [ ] 同一聚合根未混用注解与 XML SQL 风格
- [ ] 批量操作未超过 1000 条
