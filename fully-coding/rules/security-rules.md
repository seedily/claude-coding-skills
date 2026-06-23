---
name: security-rules
description: 安全开发规范。定义 SQL 注入防护、XSS 防护、敏感信息处理、越权校验、硬编码密钥与输入校验等安全要求。
---

# 安全开发规范

> 适用于 Java 后端服务的通用安全开发要求，是代码评审的强制检查项。

---

## 1. SQL 注入防护

### 1.1 强制使用参数化查询

禁止通过字符串拼接构造 SQL、表名、字段名或条件片段。

```java
// ❌ 错误：字符串拼接 SQL
String sql = "SELECT * FROM demo_user WHERE username = '" + username + "'";

// ✅ 正确：MyBatis-Plus Wrapper 参数化查询
LambdaQueryWrapper<UserPo> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(UserPo::getUsername, username);
userMapper.selectOne(wrapper);
```

### 1.2 动态排序与白名单

若接口支持前端传入排序字段，必须后端白名单校验：

```java
private static final Set<String> ALLOWED_SORT_FIELDS = Set.of("createdAt", "id");

public void validateSortField(String sortField) {
    if (!ALLOWED_SORT_FIELDS.contains(sortField)) {
        throw new BizInputException("100001", "非法排序字段");
    }
}
```

### 1.3 复杂动态 SQL

使用 MyBatis XML 时，`${}` 仅用于**不可被用户影响**的静态标识（如固定表名），且需代码评审特别确认；所有用户输入必须使用 `#{}`。

---

## 2. XSS 防护

### 2.1 输出转义

返回给前端渲染的文本字段必须进行 HTML 转义，防止反射型/存储型 XSS。

- 推荐使用自动转义机制（如 Spring 的 `HtmlUtils.htmlEscape`）
- 富文本场景使用白名单标签过滤器（如 OWASP Java HTML Sanitizer）

### 2.2 输入校验

```java
@NotBlank
@Pattern(regexp = "[\\u4e00-\\u9fa5a-zA-Z0-9_]{1,32}")
private String nickname;
```

---

## 3. 敏感信息处理

### 3.1 日志脱敏

敏感字段（手机号、身份证、银行卡、密码、Token）禁止明文输出到日志：

```java
log.info("用户登录: phone={}", DesensitizeUtil.maskPhone(phone));
```

### 3.2 接口返回脱敏

```java
@JsonSerialize(using = MaskPhoneSerializer.class)
private String phone;
```

### 3.3 配置与密钥

- 禁止在代码中硬编码数据库密码、API Key、私钥
- 生产环境密钥通过环境变量、KMS、Vault 等安全管理
- 配置文件中的 `password`、`secret`、`key` 等字段使用占位符或加密存储

---

## 4. 越权校验

### 4.1 接口入口校验

每个涉及资源操作的接口必须校验当前用户是否有权操作该资源：

```java
public void deleteOrder(Long orderId, Long currentUserId) {
    OrderAggr order = orderRepository.findById(orderId)
        .orElseThrow(() -> new ResourceNotFoundException("200001", "订单不存在"));
    if (!order.getOwnerId().equals(currentUserId)) {
        throw new PermissionDeniedException("300001", "无权操作该订单");
    }
    order.delete();
}
```

### 4.2 禁止依赖前端隐藏按钮做权限控制

权限控制必须在后端完成，前端仅做体验层面的隐藏。

---

## 5. 输入校验与边界防御

### 5.1 统一入口校验

Controller 层使用 Bean Validation 对 Request DTO 进行校验：

```java
@PostMapping("/create")
public UserDto create(@RequestBody @Valid CreateUserRequest request) { ... }
```

### 5.2 文件上传

- 限制文件类型（白名单）
- 限制文件大小
- 禁止用户控制文件存储路径
- 存储时使用随机文件名，禁止保留原始扩展名执行

### 5.3 反序列化安全

- 禁止反序列化不可信来源的数据
- Jackson 关闭默认类型（`defaultTyping`）
- 对 `ObjectMapper` 禁用不安全的反序列化特性

---

## 6. 会话与鉴权

### 6.1 Token 安全

- Token 使用 HTTPS 传输
- 设置合理过期时间
- 禁止在日志中输出完整 Token

### 6.2 幂等性与重放

核心写接口必须实现幂等，参见 `error-handling-rules.md` §6。

---

## 7. Code Review 安全清单

- [ ] 不存在 SQL 字符串拼接
- [ ] 不存在 `${}` 拼接用户输入
- [ ] 敏感字段已脱敏（日志 + 接口返回）
- [ ] 无硬编码密码 / Key / Secret
- [ ] 资源操作已做越权校验
- [ ] Request DTO 已加 `@Valid` 校验
- [ ] 文件上传已限制类型与大小
- [ ] 反序列化来源可信
- [ ] Token / Session 未明文打印

---
