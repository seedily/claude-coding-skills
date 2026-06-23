---
name: error-handling-rules
description: 异常处理与错误码规则。定义错误码结构、异常体系、异常处理原则与幂等性设计。
---

# 异常处理与错误码规则

> 异常体系、错误码结构、幂等性设计。

---

## 1. 错误码规范

错误码由 6 位底层码组成，逻辑上分为两段：

```text
          [L][CC]   [SS][D]
底层6位   错误性质   系统与领域
```

- `[L][CC]`：错误性质段，表示错误级别与分类
- `[SS][D]`：系统领域段，表示系统代号与领域内代码

| 段位 | 含义 | 范围（十六进制） | 说明 |
|------|------|----------------|------|
| `L` | level | `0~6` | 错误级别 / 性质 |
| `CC` | category | `00~FF` | 错误分类 / 序号 |
| `SS` | system | `00~FF` | 系统代号 |
| `D` | domain | `0~F` | 领域内代码 |

### 错误码结构示例

以错误码 `100001` 为例：

```text
  1   00   00   1
  ↑   ↑    ↑    ↑
  L   CC   SS   D
  │   │    │    └── 领域内代码：1（如 user 领域）
  │   │    └─────── 系统代号：00（如 demo 系统）
  │   └──────────── 错误分类：00（如参数校验）
  └──────────────── 错误级别：1（warn / BizInputException）
```

含义：`100001` = demo 系统 user 领域的参数校验类业务输入警告（如"用户名不能为空"）。

---

## 2. Level 段位规范

| L | 级别 / 异常 | 场景 | 建议日志级别 |
|---|-------------|------|--------------|
| `0` | 成功 | 请求处理成功 | - |
| `1` | `warn:BizInputException` | 业务输入 / 状态错误（参数缺失、状态机冲突、重复提交） | `WARN` |
| `2` | `error:ResourceNotFoundException` | 资源不存在（记录缺失、文件缺失、配置缺失） | `WARN` |
| `3` | `error:PermissionDeniedException` | 权限 / 风控 / 合规（未授权、角色不足、风控拦截） | `WARN` |
| `4` | `error:AuthProtocolException` | 协议 / 鉴权 / 签名（Token 过期、签名错误、Nonce 重放） | `WARN` |
| `5` | `error:InternalServerException` | 服务端内部错误（DB 超时、OOM、NPE、线程池满） | `ERROR` |
| `6` | `error:DependencyException` | 第三方依赖错误（支付网关失败、短信通道超时） | `ERROR` |

---

## 3. 异常体系

| L | 异常类 | 级别 | 场景 | HTTP 状态码 |
|---|--------|------|------|-------------|
| `1` | `BizInputException` | warn | 业务输入 / 状态错误 | 200 |
| `2` | `ResourceNotFoundException` | error | 资源不存在 | 200 |
| `3` | `PermissionDeniedException` | error | 权限 / 风控 / 合规 | 200 |
| `4` | `AuthProtocolException` | error | 协议 / 鉴权 / 签名 | 200 |
| `5` | `InternalServerException` | error | 服务端内部错误 | 200 |
| `6` | `DependencyException` | error | 第三方依赖错误 | 200 |

> HTTP 层统一返回 200，真实错误通过响应体中的业务错误码与异常类型表达。

---

## 4. 异常使用示例

```java
// 聚合根 / DomainService 中抛出
throw new BizInputException("100001", "用户名不能为空");
throw new BizInputException("100003", "会员等级只能升级，不能降级");

// 资源不存在
throw new ResourceNotFoundException("200001", "用户不存在");

// 权限 / 风控
throw new PermissionDeniedException("300001", "当前角色无操作权限");

// 协议 / 鉴权
throw new AuthProtocolException("400001", "Token 已过期");

// 服务端内部错误
throw new InternalServerException("500001", "数据库查询超时");

// 第三方依赖错误
throw new DependencyException("600001", "短信通道调用失败", e);
```

---

## 5. 异常处理原则

### 5.1 捕获后包装再抛出

```java
try {
    paymentService.charge(order);
} catch (PaymentException e) {
    log.error("支付失败: orderId={}", orderId, e);
    throw new DependencyException("600001", "支付网关调用失败", e);
}
```

### 5.2 保留原始异常链

```java
catch (SQLException e) {
    throw new RuntimeException("数据库查询失败", e);  // 保留 cause
}
```

### 5.3 禁止空 catch

```java
// ❌ 错误：吞掉异常
catch (Exception e) {
    // 空 catch
}
```

### 5.4 不要用于流程控制

```java
// ❌ 错误：异常用于分支判断
try {
    userRepository.findById(userId);
} catch (Exception e) {
    createUser(...);  // 用异常判断用户不存在
}

// ✅ 正确：用 Optional 判断
Optional<User> userOpt = userRepository.findById(userId);
if (userOpt.isEmpty()) {
    createUser(...);
}
```

---

## 6. 幂等性设计

外部调用可能因超时、网络抖动导致重复请求，核心写接口必须实现幂等。

### 6.1 幂等键来源

- 客户端生成 `idempotencyKey`（UUID），放入请求头 `X-Idempotency-Key`
- 服务端以 `(idempotencyKey, 接口标识)` 为唯一索引

### 6.2 实现模式

#### 数据库唯一索引（推荐）

```sql
CREATE UNIQUE INDEX uk_idempotent ON member_account (idempotency_key, api_endpoint);
```

#### Redis 分布式锁（短时效）

```java
String key = "idempotent:" + idempotencyKey;
Boolean locked = redisTemplate.opsForValue().setIfAbsent(key, "1", Duration.ofMinutes(5));
if (!Boolean.TRUE.equals(locked)) {
    throw new BizInputException(..., "重复请求");
}
```

### 6.3 Application 层检查示例

```java
@Transactional
public MemberTokenDto register(RegisterRequest request) {
    if (request.getIdempotencyKey() != null) {
        if (!idempotencyChecker.checkAndMark(request.getIdempotencyKey(), "register")) {
            throw new BizInputException("100015", "重复请求，请勿重复提交");
        }
    }
    // 后续业务逻辑...
}
```

### 6.4 过期策略

幂等键保留 24 小时，过期后自动释放。

---