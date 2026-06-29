---
name: backend-code-standard-rules
description: 通用编码规范。定义 Java 后端开发的代码格式、命名规范、空值处理、日志、并发安全、Lombok 使用、工具类、注释与 Code Review 检查清单。
---

# 通用编码规范

> Java 后端系统开发通用规范，与业务框架无关。

---

## 1. 代码格式

### 1.1 分支语句必须换行

`if`、`for`、`while`、`try` 等语句体即使只有一行，也必须使用大括号并换行。

### 1.2 运算符前后空格

```java
int result = a + b;
boolean flag = count > 0 && count < 100;
```

### 1.3 方法参数逗号后空格

```java
public UserDto createUser(String username, String email, String phone) { ... }
```

### 1.4 类成员顺序

```java
public class Example {
    // 1. 静态常量
    public static final int MAX_SIZE = 100;

    // 2. 实例常量
    private final String name;

    // 3. 静态变量（或改用 Lombok @Slf4j，无需显式声明 log）
    private static Logger log = LoggerFactory.getLogger(Example.class);

    // 4. 实例变量
    private Long id;
    private String email;

    // 5. 构造器
    public Example(String name) { this.name = name; }

    // 6. 公有方法
    public void doSomething() { ... }

    // 7. 私有方法
    private void helper() { ... }
}
```

---

## 2. 命名规范

### 2.1 方法命名

| 场景 | 前缀 | 示例 | 事务 |
|------|------|------|------|
| 查询单个 | `query` | `queryUser(Long userId)` | 无 |
| 查询列表 | `query` + List | `queryUserList(...)` | 无 |
| 分页查询 | `query` + Page | `queryUserPage(...)` | 无 |
| 创建 | `create` | `createUser(...)` | 有 |
| 更新 | `update` | `updateUser(...)` | 有 |
| 删除 | `delete` | `deleteUser(...)` | 有 |
| 业务动作 | 动词 | `upgradeLevel(...)`、`cancelOrder(...)` | 有 |

规则：
- 读操作不加 `@Transactional`，写操作加 `@Transactional`
- 查询方法返回 `Optional<T>` 或 `List<T>`，不返回 `null` 表示空列表

### 2.2 类名后缀（驼峰强制）

PO / DTO / VO 等作为类名后缀时，**必须使用驼峰命名**，禁止全部大写：

| 类型 | 正确 | 错误 |
|------|------|------|
| 持久化对象 | `UserPo`、`OrderItemPo` | ~~`UserPO`~~、~~`OrderItemPO`~~ |
| 数据传输对象 | `UserDto`、`OrderPageDto` | ~~`UserDTO`~~、~~`OrderPageDTO`~~ |
| 值对象 | `AddressVo`、`MoneyVo` | ~~`AddressVO`~~、~~`MoneyVO`~~ |

原因：类名属于大驼峰（UpperCamelCase），后缀作为类名的一部分应遵循驼峰规则。

### 2.3 布尔变量与方法

布尔变量名的前缀使用 `has`、`can`、`should` 前缀，不允许直接用 `isXxx`：
布尔方法命名的前缀使用 `is`，允许直接使用 lombok 的 getter 注解：

```java
// 推荐
private boolean active;
public boolean isActive() { ... }

private boolean hasPermission;
public boolean canUpgrade() { ... }

// 避免
private boolean isActive;      // 部分序列化框架会生成歧义 getter
private boolean flag;          // 无意义
public boolean check() { ... } // 返回值语义不明
```

### 2.4 集合变量

集合变量名体现复数或类型：

```java
// 推荐
private List<User> userList;
private Map<Long, User> userMap;
private Set<String> tagSet;

// 避免
private List<User> users;      // 与单数 user 区分度低
private Map<Long, User> data;  // 无意义
```

### 2.5 服务启动类命名

Spring Boot 服务入口类（带 `@SpringBootApplication` 的类）统一命名为 `{Xxx}AppRunner`，文件名 `{Xxx}AppRunner.java`。禁止用 `XxxApplication`、`XxxMain`、`XxxBootstrap` 等命名，避免与应用服务类 `XxxApplicationService` 混淆。

```java
// ✅ 正确
@SpringBootApplication
public class AdminBizAppRunner { ... }

// ❌ 错误
@SpringBootApplication
public class AdminBizApplication { ... }
```

> 应用服务类遵循 DDD 分层规范命名为 `XxxApplicationService`（见 `backend-ddd-layer-rules.md`），与启动类 `XxxAppRunner` 是两个不同角色，不可混用。批量重命名服务类时，启动类是命名例外，不要套用 `XxxApplicationService` 规则。

---

## 3. 空值与集合处理

### 3.1 空值判断

```java
// ✅ 正确
if (Objects.isNull(user)) { ... }
if (Objects.nonNull(user)) { ... }

// ✅ Optional 处理可能为空的值
Optional<User> optUser = userRepository.findById(id);
User user = optUser.orElseThrow(() -> new RuntimeException("用户不存在"));
String name = optUser.map(User::getName).orElse("匿名");
```

### 3.2 集合初始化

```java
// ✅ 返回空集合而非 null
public List<User> queryUsers() {
    return userMapper.selectList();
}
```

> MyBatis-Plus 等框架保证 `selectList()` 返回非 null 集合；若调用外部 SDK 或手写 JDBC，需防御性返回 `Collections.emptyList()`。

### 3.3 Stream 使用

```java
// ✅ 简单转换用 Stream
List<String> names = users.stream()
    .filter(u -> u.getStatus() == 1)
    .map(User::getName)
    .collect(Collectors.toList());

// ✅ 复杂逻辑提取方法
List<UserDto> dtos = users.stream()
    .map(this::convertToDto)
    .collect(Collectors.toList());
```

---

## 4. 日志规范

### 4.1 日志级别使用

| 级别 | 使用场景 | 示例 |
|------|---------|------|
| `ERROR` | 系统异常，需要人工介入 | 数据库连接失败、MQ 发送失败 |
| `WARN` | 业务异常，可自动恢复 | 用户不存在、库存不足 |
| `INFO` | 业务关键节点 | 用户创建成功、订单支付完成 |
| `DEBUG` | 调试信息 | 方法入参、SQL 语句 |
| `TRACE` | 最详细的追踪 | 方法内部每一步状态 |

### 4.2 日志输出规范

```java
// ✅ 使用占位符
log.info("用户创建成功: userId={}, username={}", userId, username);

// ✅ 异常信息必须记录
log.error("订单处理失败: orderId={}", orderId, e);

// ❌ 字符串拼接（无条件执行）
log.info("用户创建成功: userId=" + userId + ", username=" + username);
```

### 4.3 敏感信息脱敏

```java
log.info("用户登录: phone={}, idCard={}",
    DesensitizeUtil.maskPhone(phone),
    DesensitizeUtil.maskIdCard(idCard));
```

---

## 5. 并发安全

### 5.1 线程安全类选择

| 场景 | 推荐 | 不推荐 |
|------|------|--------|
| 计数 | `AtomicLong`、`LongAdder` | `synchronized` |
| 缓存 | `ConcurrentHashMap` | `HashMap` + 同步 |
| 队列 | `ConcurrentLinkedQueue` | `LinkedList` |
| 服务内定时任务 | `ScheduledExecutorService` | `Timer` |
| 服务间/分布式调度任务 | 由统一管理平台（如自研调度中心）统一编排 | XXL-Job / Elastic-Job / Quartz 等独立调度框架 |

### 5.2 避免共享可变状态

```java
// ✅ 使用原子类
private final AtomicInteger count = new AtomicInteger(0);
public void increment() {
    count.incrementAndGet();
}
```

### 5.3 日期时间处理

```java
// ✅ Java 8 Time API
LocalDateTime now = LocalDateTime.now();
LocalDate date = LocalDate.parse("2024-01-01");
```

---

## 6. 注释与文档

### 6.1 建议写注释

| 场景 | 说明 |
|------|------|
| 复杂算法/业务规则 | 代码本身无法表达设计意图 |
| 非常规写法 | 存在更直观写法但选择了当前方案 |
| 外部依赖/约束 | 与第三方系统交互的约定 |
| TODO/FIXME | 临时方案，后续需要优化 |

### 6.2 禁止写注释

| 场景 | 反例 |
|------|------|
| 代码本身已自解释 | `int i = 0; // 定义变量 i` |
| 注释与代码不符 | `// 创建用户` 实际在删除用户 |
| 用注释替代命名 | 应改为方法名 `calculateTotalAmount()` |

### 6.3 方法注释标准

- **公有方法**（`public`）：建议写 Javadoc，说明参数、返回值、异常
- **私有方法**：一般不需要 Javadoc，除非逻辑复杂
- **Getter/Setter**：不需要注释（Lombok 生成即可）

---

## 7. 工具类使用

### 7.1 优先使用标准库

| 场景 | 推荐 |
|------|------|
| 字符串判空 | `StringUtils.isBlank()` (Apache) |
| 对象判空 | `Objects.isNull()` / `Objects.nonNull()` |
| 集合判空 | `CollectionUtils.isEmpty()` (Spring) |
| Bean 拷贝 | `BeanUtils.copyProperties()` (Spring) |
| JSON 处理 | Jackson `ObjectMapper` |
| 日期格式化 | `DateTimeFormatter` |

---

## 8. Lombok 使用规范

### 8.1 推荐注解

| 注解 | 场景 | 说明 |
|------|------|------|
| `@Getter` / `@Setter` | POJO、DTO、PO | 替代手写 getter/setter |
| `@ToString` | 调试类 | 注意排除敏感字段 `@ToString.Exclude` |
| `@EqualsAndHashCode` | 值对象 | 基于业务字段生成，禁止在聚合根上使用 |
| `@Slf4j` | 需要日志的类 | 替代 `LoggerFactory.getLogger()` |
| `@RequiredArgsConstructor` | Service / Controller | 配合 `final` 字段实现构造器注入 |
| `@Data` | 简单数据类 | **谨慎使用**，禁止用于聚合根/领域实体 |

### 8.2 使用示例

#### POJO / DTO

```java
@Getter
@Setter
@ToString
public class UserDto {
    private Long id;
    private String username;
    private String email;
}
```

#### 领域服务 + 构造器注入

```java
@Slf4j
@RequiredArgsConstructor
@Service
public class UserAggrDomainService {
    private final UserAggrRepository userAggrRepository;
    private final ApplicationEventPublisher eventPublisher;
}
```

#### 排除敏感字段

```java
@Getter
@Setter
@ToString(exclude = "password")
public class UserLoginDto {
    private String username;
    private String password;
}
```

### 8.3 禁止场景

| 场景 | 原因 |
|------|------|
| `@Data` 用于聚合根/领域实体 | 生成 `equals`/`hashCode` 基于所有字段，可能破坏实体同一性 |
| `@AllArgsConstructor` 用于公共类 | 暴露过多构造参数，破坏封装 |
| `@SneakyThrows` 用于业务代码 | 隐藏受检异常，破坏异常契约 |
| `@Builder` 用于 PO / 数据库实体 | 与 MyBatis 等框架的对象创建方式存在兼容风险 |

---

## 9. Code Review 检查清单

### 9.1 基础项

- [ ] `if` / `for` / `while` 体即使只有一行也使用大括号
- [ ] 不存在空 `catch` 块
- [ ] 方法参数不超过 5 个（超过考虑封装为 DTO）
- [ ] 类行数不超过 500 行
- [ ] 方法行数不超过 50 行
- [ ] 不存在魔法数字（提取为常量）
- [ ] 日志使用占位符而非字符串拼接
- [ ] 敏感信息在日志中已脱敏

### 9.2 空值与异常

- [ ] 方法返回值不返回 `null` 表示空集合
- [ ] `Optional` 不滥用（不用 `Optional` 作字段或方法参数）
- [ ] 异常保留原始 cause（不吞掉堆栈）
- [ ] 异常不用于流程控制

### 9.3 并发与安全

- [ ] 共享可变状态使用线程安全类
- [ ] 日期处理使用 Java 8 Time API
- [ ] 敏感数据不在日志中明文输出

### 9.4 Lombok 使用

- [ ] 聚合根/领域实体未使用 `@Data`
- [ ] 公共类未使用 `@AllArgsConstructor`
- [ ] 业务代码未使用 `@SneakyThrows`
- [ ] `@ToString` 已排除敏感字段

### 9.5 工具类与持久层

- [ ] 优先使用标准库与成熟工具类（Apache Commons、Spring、Jackson）
- [ ] 复杂 SQL 已确认索引命中
- [ ] 未使用 `SELECT *`

---

## 10. 接口契约与序列化

> Controller 对外契约的统一规范，必须遵守。

### 10.1 统一响应

Controller 返回 `com.grace.common.api.dto.DataResponse<T>`（或 `ListResponse` / `PageResponse`）。禁止自定义响应包装类（如 `XxxResult` / `XxxResponse` 包装类、裸 `ResponseEntity<Map>`）。

```java
// ✅ 正确
public DataResponse<UserDto> queryUser(...) { ... }

// ❌ 错误：自定义包装类（code 用 int、缺 detail/disclaimer/success 等标准字段）
public XxxResponse queryUser(...) { ... }
```

### 10.2 请求体

`@RequestBody` 直接接收业务 DTO（`XxxRequest`）或 `Map<String, Object>`。禁止自定义统一请求包装类（如 `{api, version, data}` 结构）把业务数据再包一层后用 `getData()` 取。

```java
// ✅ 正确
public DataResponse<Void> update(@RequestBody UpdateUserRequest req) { ... }
public DataResponse<?> query(@RequestBody Map<String, Object> req) { ... } // 动态字段

// ❌ 错误：自定义包装 + getData()
public DataResponse<?> query(@RequestBody XxxRequestWrapper req) {
    Object data = req.getData();
}
```

### 10.3 字段命名一律 camelCase

DTO / VO / PO / 响应字段全部 camelCase。禁止用 Jackson 注解把字段转 snake_case：

| ❌ 错误 | 原因 |
|---|---|
| `@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)` | 类级强制 snake_case，破坏前后端 camelCase 契约 |
| `@JsonProperty("user_name")` | 字段级强制 snake_case，同上 |

对接外部 snake_case 系统（第三方/下游服务）时，在适配层（ApplicationService / Converter）用 `map.get("snake_case")` 转换，DTO 本身保持 camelCase。

### 10.4 ObjectMapper 容错

`ObjectMapper.convertValue` 把前端 body 转 DTO 时，必须关闭未知字段失败，避免前端多传/改名字段直接抛 `UnrecognizedPropertyException` → 500。优先注入 Spring 的 `ObjectMapper`，不要 `new ObjectMapper()`。

```java
// ✅ 正确：注入 + 关闭未知字段失败
private final ObjectMapper objectMapper;

private <T> T parse(Object data, Class<T> clazz) {
    if (data instanceof Map<?, ?> map) {
        return objectMapper.copy()
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                .convertValue(map, clazz);
    }
    return null;
}

// ❌ 错误：默认 strict，前端字段不匹配即 500
new ObjectMapper().convertValue(map, XxxRequest.class);
```
