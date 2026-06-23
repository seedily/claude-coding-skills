---
name: backend-ddd-layer-rules
description: DDD 分层架构与领域模型规则。定义分层职责、命名规范、聚合根设计（工厂方法、业务规则、领域事件、软删除、事件订阅）、仓储设计（转换、批量、Converter）、接口风格、依赖注入与 CQRS 策略。
---

# DDD 分层架构规则

> 基于 DDD（领域驱动设计）思想的分层开发规则，适用于 Spring Boot + MyBatis-Plus 后端服务。

---

## 1. 分层职责

### 1.1 层次职责隔离

| 层次 | 职责 | 禁止事项 |
|------|------|----------|
| **Controller** | 接收 HTTP 请求，参数校验，返回响应 | 禁止直接调用 Repository/Mapper |
| **Application** | 用例编排，事务控制，DTO 拆解/组装，调用 DomainService | 禁止写业务规则 |
| **DomainService** | 聚合根生命周期管理，PO ↔ 聚合根转换，领域事件发布，跨聚合根协调 | 禁止直接操作数据库 |
| **Aggregate** | 封装业务规则与不变量，注册领域事件 | 禁止依赖 Spring / 数据库 |
| **Repository** | 纯数据访问，操作 PO | 禁止返回聚合根 / 感知业务规则 |
| **DAO** | PO、Mapper、XML，数据库交互 | 禁止引入业务逻辑 |

### 1.2 模块划分

```
xxx-api        —— 对外契约：DTO、枚举、跨服务事件。零框架依赖。
xxx-domain     —— 领域核心：聚合根、领域服务、仓储实现。
xxx-dao        —— 持久层：PO、Mapper、XML。
xxx-app        —— 应用层 + 启动入口：Controller、ApplicationService。
```

### 1.3 分层命名规范

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| 聚合根 | `{业务}Aggr` | `UserAggr`、`OrderAggr` |
| 领域服务 | `{Aggr}DomainService` | `UserAggrDomainService` |
| 仓储 | `{Aggr}Repository` | `UserAggrRepository` |
| 应用服务 | `{Aggr}ApplicationService` | `UserAggrApplicationService` |
| Controller | `{Aggr}Controller` | `UserAggrController` |
| 配置类 | `{模块}Config` | `GraceConfig`、`KafkaConfig` |
| 请求 DTO | `{动作}{对象}Request` | `CreateUserRequest`、`UpdateOrderRequest` |
| 响应 DTO | `{对象}Dto` | `UserDto`、`OrderDto` |

> 聚合根必须保留 `Aggr` 后缀，以区别于贫血 POJO 和数据库实体。

### 1.4 领域事件命名

领域事件与事件监听器的命名规则如下：

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| 领域内事件 | `{对象}{动作}Event` | `UserCreatedEvent`、`OrderPaidEvent` |
| 跨服务事件 | `{对象}{动作}Event`（放 api 模块） | `UserCreatedEvent` |
| 事件监听器 | `{对象}EventHandler` | `UserEventHandler` |

### 1.5 聚合根包组织

```
com.example.member.domain/
├── exception/
│   └── DomainException.java
└── user/
    ├── aggregate/
    │   └── UserAggr.java
    ├── event/
    │   ├── UserCreatedEvent.java
    │   └── UserLevelUpgradedEvent.java
    └── service/
        └── UserAggrDomainService.java
```

---

## 2. 核心原则

### 2.1 依赖方向

- 外层依赖内层，内层不依赖外层
- Repository 只返回 PO，聚合根转换在 DomainService
- 仓储抽象持久化，Domain 层不感知 MyBatis/JPA

### 2.2 富领域模型

- 业务规则写在聚合根里，不是在 Service 里写 `if-else` 事务脚本
- 贫血模型（只有 getter/setter）是反模式，聚合根必须有行为
- PO 只有 getter/setter 没问题，但聚合根必须封装业务逻辑

### 2.3 事务边界

- **事务控制在 Application 层**，不在 DomainService 层
- 一个事务原则上只修改一个聚合根
- 跨聚合根操作：
  - 同库最终一致：本地事务 + 领域事件异步补偿（推荐）
  - 同库强一致：经架构评审后，可在单个 `@Transactional` 内调用多个 Repository，但需显式说明理由
  - 跨库/最终一致：本地事务 + 领域事件异步补偿
  - 长事务/Saga：状态机 + 事件驱动

---

## 3. 聚合根设计

### 3.1 工厂方法

每个聚合根必须提供两个工厂方法：

```java
public class UserAggr {
    private Long id;
    private String username;

    // 禁止外部通过默认构造器创建
    private UserAggr() {}

    // 1. 创建新聚合根 —— 触发完整的业务校验
    public static UserAggr create(String username, String email, ...) {
        validateUsername(username);
        UserAggr aggr = new UserAggr();
        aggr.username = username.trim();
        return aggr;
    }

    // 2. 重建（从持久化层恢复）—— 不触发校验，不注册事件
    public static UserAggr reconstitute(Long id, String username, ...) {
        UserAggr aggr = new UserAggr();
        aggr.id = id;
        aggr.username = username;
        return aggr;
    }
}
```

### 3.2 业务规则内聚

- 业务规则必须写在聚合根内部
- 禁止在 DomainService 中写业务 `if-else`
- 聚合根方法内注册领域事件

### 3.3 领域事件

```java
private final List<DomainEvent> domainEvents = new ArrayList<>();

private void registerEvent(DomainEvent event) {
    this.domainEvents.add(event);
}

public List<DomainEvent> pullDomainEvents() {
    List<DomainEvent> events = Collections.unmodifiableList(new ArrayList<>(domainEvents));
    domainEvents.clear();
    return events;
}
```

事件注册时机：
- `markCreated()`：首次持久化后调用（此时 id 已回填）
- 业务方法内：如 `upgradeLevel()` 中注册升级事件

### 3.4 软删除处理

#### 3.4.1 聚合根重建校验

```java
public static MemberAccountAggr reconstitute(Long id, ..., LocalDateTime deletedAt) {
    if (deletedAt != null) {
        throw new ResourceNotFoundException("...", "记录已删除或已注销");
    }
    // ... 正常重建
}
```

#### 3.4.2 Repository 默认过滤

```java
public Optional<MemberAccountPo> findById(Long id) {
    LambdaQueryWrapper<MemberAccountPo> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(MemberAccountPo::getId, id)
           .isNull(MemberAccountPo::getDeletedAt);
    return Optional.ofNullable(memberAccountMapper.selectOne(wrapper));
}
```

### 3.5 领域事件订阅

#### 3.5.1 监听注解选择

| 注解 | 适用场景 | 风险 |
|------|---------|------|
| `@EventListener` | 纯内存操作（日志、本地缓存清理） | 事务回滚时事件已执行 |
| `@TransactionalEventListener(phase = AFTER_COMMIT)` | 涉及外部系统（MQ、短信、邮件） | 仅在事务成功提交后执行 |

**默认原则**：写操作产生的事件，优先使用 `@TransactionalEventListener(phase = AFTER_COMMIT)`。

#### 3.5.2 事件存放位置

| 类型 | 位置 | 说明 |
|------|------|------|
| 跨服务事件 | `api/dto/event/` | 可被外部服务引用 |
| 领域内事件 | `domain/{aggr}/event/` | 仅本服务内部使用 |

#### 3.5.3 事件序列化规范

跨服务事件通过 MQ 投递时：

- 事件体使用 JSON，字段名驼峰命名
- 必须包含：`eventId`（UUID）、`occurredAt`（ISO-8601）、`aggregateId`、`eventType`
- 禁止在事件中传递聚合根完整状态（只传变更字段或 ID）

```java
@Getter
@Setter
@ToString
public class MemberRegisteredEvent {
    private String eventId;
    private String eventType = "MemberRegistered";
    private Long aggregateId;
    private String phoneHash;
    private LocalDateTime occurredAt;
}
```

#### 3.5.4 消费端幂等

使用 `eventId` 作为幂等键：

```java
@KafkaListener(topics = "member.event")
public void onMemberRegistered(MemberRegisteredEvent event) {
    if (eventConsumerLogRepository.existsByEventId(event.getEventId())) {
        return; // 已消费
    }
    // 业务处理...
    eventConsumerLogRepository.save(new EventConsumerLog(event.getEventId(), LocalDateTime.now()));
}
```

---

## 4. 仓储设计

### 4.1 职责边界

- Repository：纯数据访问，只操作 PO
- DomainService：持有 `toPo()` / `toAggr()` 转换方法，编排用例

### 4.2 为什么转换在 Service 而非 Repository

1. 单一职责：Repository 只做数据访问
2. 测试友好：Mock Repository 返回固定 PO 即可测试 DomainService
3. 生命周期控制：Service 层统一管理"查 → 改 → 转 → 存"流程

### 4.3 创建聚合根的完整流程（ApplicationService 中）

```java
@Transactional
public UserDto createUser(...) {
    // 1. 查重
    // 2. 创建聚合根
    UserAggr aggr = UserAggr.create(...);
    // 3. 转 PO 并持久化
    UserPo po = toPo(aggr);
    userAggrRepository.save(po);
    // 4. 回填 ID
    aggr.assignId(po.getId());
    // 5. 标记创建，注册领域事件
    aggr.markCreated();
    // 6. 发布事件
    publishEvents(aggr);
    // 7. 返回 DTO
    return toDTO(aggr);
}
```

### 4.4 PO ↔ Aggr 转换提取

当聚合根数量超过 2 个或转换逻辑超过 30 行时，将 `toPo()` / `toAggr()` 提取为独立的 **Converter** 类：

```java
@Component
public class MemberAccountConverter {
    public MemberAccountPo toPo(MemberAccountAggr aggr) { ... }
    public MemberAccountAggr toAggr(MemberAccountPo po) { ... }
}
```

DomainService 注入 Converter 使用。

### 4.5 批量操作

**禁止**：在循环里逐个调用 `aggregate.create()` + `repository.save()`。

**推荐**：Application 层组装 `List<Aggr>` → 转 `List<Po>` → MyBatis-Plus `saveBatch()`。

```java
@Transactional
public void batchCreateUsers(List<CreateUserRequest> requests) {
    List<UserAggr> aggrs = requests.stream()
        .map(req -> UserAggr.create(req.getUsername(), ...))
        .toList();
    List<UserPo> pos = aggrs.stream().map(this::toPo).toList();
    userAggrRepository.saveBatch(pos);
}
```

批量操作不注册逐条领域事件，如需事件，用批量事件。

---

## 5. 接口风格

### 5.1 RPC 风格（全部 POST）

不使用 RESTful，全部使用 POST：

所有查询类接口统一使用 POST + Body，便于参数扩展、统一脱敏与日志审计：

| 接口 | 方法 | 路径 | 参数 |
|------|------|------|------|
| 创建 | POST | `/api/users/create` | Body: CreateUserRequest |
| 查询单个 | POST | `/api/users/query` | Body: QueryUserRequest |
| 查询列表 | POST | `/api/users/queryList` | Body: UserListRequest / 无参 |
| 分页查询 | POST | `/api/users/queryPage` | Body: UserPageRequest |
| 更新 | POST | `/api/users/update` | Body: UpdateUserRequest |
| 删除 | POST | `/api/users/delete` | Body: DeleteUserRequest |

> 无参列表接口可用空 Body；简单主键查询也建议用 Body，避免 URL 长度限制与敏感信息暴露在路径中。

### 5.2 接口契约

#### 响应格式

```java
return {{config.response_wrapper_success}}(userDto);
return {{config.base_response_class}}.success();
return {{config.response_wrapper_fail}}("100001", "用户名不能为空");
```

#### 统一响应结构

```json
{
  "success": true,
  "code": "0",
  "message": "ok",
  "data": { }
}
```

失败时：

```json
{
  "success": false,
  "code": "100001",
  "message": "用户名不能为空",
  "data": null
}
```

#### 分页响应结构

```json
{
  "success": true,
  "code": "0",
  "message": "ok",
  "data": {
    "pageNum": 1,
    "pageSize": 20,
    "total": 100,
    "pages": 5,
    "list": []
  }
}
```

#### 字段与序列化约定

| 项目 | 规范 | 说明 |
|------|------|------|
| 字段命名 | 驼峰命名 | `userName`、`createdAt` |
| 枚举交互 | `code` + `name` | 后端返回 `{ "code": "1", "name": "启用" }`，禁止直接用中文 |
| 时间格式 | ISO-8601 | `2024-01-01T12:00:00`，跨时区业务使用 UTC 或带时区偏移 |
| 金额 | 分为单位的整数 / BigDecimal | 禁止用 float/double 传递金额 |
| 空值字段 | 返回 `null` 而非省略 | 便于前端类型推断；敏感字段可脱敏后返回 |
| 布尔值 | `true` / `false` | 禁止使用 `1` / `0` 或 `Y` / `N` |

#### 版本控制

- 接口路径通过 `/api/v1/users/create` 体现版本
- 同一 Controller 内禁止混合不同版本路径
- 跨版本字段变更必须通过新增字段实现，禁止删除或重命名已发布字段

---

## 6. 依赖注入

**强制构造器注入**，禁止 `@Autowired` 字段注入。推荐配合 Lombok `@RequiredArgsConstructor`：

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class UserAggrDomainService {
    private final UserAggrRepository userAggrRepository;
    private final ApplicationEventPublisher eventPublisher;
}
```

> 上述写法等价于手写全参构造器，由 Spring 自动按类型注入。

手写构造器版本（在不使用 Lombok 时参考）：

```java
@Service
public class UserAggrDomainService {
    private final UserAggrRepository userAggrRepository;
    private final ApplicationEventPublisher eventPublisher;

    public UserAggrDomainService(UserAggrRepository userAggrRepository,
                             ApplicationEventPublisher eventPublisher) {
        this.userAggrRepository = userAggrRepository;
        this.eventPublisher = eventPublisher;
    }
}
```

原因：
- `final` 保证不可变
- 依赖必填（构造时校验，不为 null）
- 无需 Spring 容器即可单元测试

---

## 7. CQRS 查询策略

读操作与写操作分离：

| 场景 | 写入路径 | 查询路径 |
|------|---------|---------|
| 聚合根详情 | DomainService → Aggr → DTO | QueryService → PO → DTO |
| 列表/分页 | -- | QueryService → Mapper → PO → DTO |
| 统计报表 | -- | QueryService → Mapper SQL → DTO |

QueryService 规范：
- 位于 `application/{aggr}/query/` 或 `infrastructure/query/`
- 命名 `{Aggr}QueryService`
- 直接调用 Repository / Mapper 读取 PO
- 不触发领域事件，不加 `@Transactional`
- 允许使用 `@Cacheable` 缓存读结果

---

## 8. 常见陷阱

| 陷阱 | 正确做法 |
|------|----------|
| Repository 返回聚合根 | Repository 只返回 PO，转换在 DomainService |
| Service 里写业务规则 | 业务规则下沉到聚合根 |
| 创建聚合根后直接返回 | `save(po)` → `setId(po.getId())` → `markCreated()` → `publishEvents()` |
| `@Autowired` 字段注入 | 构造器注入 + `final` |
| 贫血模型 | 富领域模型（封装业务行为和事件） |
| RESTful | RPC（全部 POST） |
| 领域事件在 `create()` 中注册 | 先 `save`，再 `markCreated()`（id 已回填） |
| 读操作加 `@Transactional` | 只有写操作加事务 |
| DomainService 层加 `@Transactional` | 事务控制在 Application 层 |
| PO 直接透传到 Controller | PO 只在 dao/domain 层流转，对外暴露 DTO |

---