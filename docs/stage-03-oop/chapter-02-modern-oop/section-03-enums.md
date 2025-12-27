# 3.2.3 枚举（Enums）

## 概述

枚举（Enums）是 PHP 8.1 引入的特性，提供类型安全的常量集合。与传统的常量类相比，枚举具有更强的类型安全性、更好的 IDE 支持、可以定义方法、可以实现接口等优势。枚举特别适用于状态管理、配置选项、类型常量等场景。

传统上，PHP 开发者使用类常量或数组来表示一组固定的值，但这种方式缺乏类型安全，容易出现拼写错误、值不一致等问题。枚举解决了这些问题，提供了类型安全的常量集合，并且在编译时进行类型检查。

理解枚举对于编写现代 PHP 代码非常重要。枚举不仅提供了类型安全的常量，还可以定义方法、实现接口，提供了比传统常量类更强大的功能。

**主要内容**：
- 基础枚举的定义和使用
- Backed Enums（有值的枚举）的定义和使用
- 枚举方法（实例方法和静态方法）
- 枚举实现接口
- 枚举与 match 表达式的配合使用
- 枚举的数据库存储
- 枚举的使用场景和最佳实践

## 特性

- **类型安全**：编译时类型检查，避免拼写错误
- **方法支持**：枚举可以定义实例方法和静态方法
- **接口实现**：枚举可以实现接口
- **数据库友好**：Backed Enums 可以存储为数据库值
- **序列化支持**：枚举可以序列化和反序列化
- **IDE 支持**：更好的代码提示和自动完成

## 语法/定义

### 基础枚举

**语法**：`enum EnumName { case Value1; case Value2; ... }`

**组成部分**：
- `enum` 关键字：声明枚举类型
- `EnumName`：枚举名称，遵循类命名规范（`StudlyCase`）
- `case` 关键字：声明枚举值
- `Value1`、`Value2`：枚举值名称，遵循常量命名规范（`UPPER_SNAKE_CASE`）

**特点**：
- PHP 8.1+ 引入的特性
- 纯枚举值，没有关联的标量值
- 每个枚举值都是该枚举类型的单例实例

### Backed Enums（有值的枚举）

**语法**：`enum EnumName: Type { case Value1 = 'value1'; case Value2 = 'value2'; }`

**组成部分**：
- 类型声明：`: Type` 指定枚举值的类型（`int` 或 `string`）
- 枚举值：每个 `case` 必须有一个标量值

**支持的类型**：
- `int`：整数枚举
- `string`：字符串枚举

**特点**：
- 枚举值有对应的标量值
- 可以通过 `from()` 和 `tryFrom()` 从值创建枚举
- 可以通过 `value` 属性获取标量值

### 枚举方法

**语法**：
- 实例方法：`public function methodName(): ReturnType { ... }`
- 静态方法：`public static function methodName(): ReturnType { ... }`

**特点**：
- 枚举可以定义实例方法和静态方法
- 实例方法中可以使用 `$this` 引用当前枚举值
- 可以使用 `self::` 引用枚举类型

## 基本用法

### 示例 1：基础枚举

```php
<?php
declare(strict_types=1);

enum Status
{
    case PENDING;
    case ACTIVE;
    case INACTIVE;
    case DELETED;
}

// 使用枚举
$status = Status::PENDING;

// 访问枚举值的名称
echo "Status name: " . $status->name . "\n";  // Status name: PENDING

// 枚举值比较
if ($status === Status::PENDING) {
    echo "Status is pending\n";
}

// 枚举值可以用作类型声明
function processStatus(Status $status): void
{
    echo "Processing status: {$status->name}\n";
}

processStatus(Status::ACTIVE);
```

**输出**：

```
Status name: PENDING
Status is pending
Processing status: ACTIVE
```

**说明**：
- 枚举值是单例，相同枚举值使用 `===` 比较返回 `true`
- 每个枚举值都有 `name` 属性，返回枚举值的名称
- 枚举可以用作类型声明，提供类型安全

### 示例 2：Backed Enums（字符串枚举）

```php
<?php
declare(strict_types=1);

enum HttpStatus: string
{
    case OK = '200';
    case NOT_FOUND = '404';
    case SERVER_ERROR = '500';
    case BAD_REQUEST = '400';
}

// 使用枚举
$status = HttpStatus::OK;

// 访问枚举值的名称和值
echo "Name: {$status->name}, Value: {$status->value}\n";  // Name: OK, Value: 200

// 从值创建枚举
$status2 = HttpStatus::from('404');
echo "Status: {$status2->name}\n";  // Status: NOT_FOUND

// 安全地从值创建（不存在时返回 null）
$status3 = HttpStatus::tryFrom('999');
if ($status3 === null) {
    echo "Invalid status code\n";
}

// 从值创建枚举（不存在时抛出异常）
try {
    $status4 = HttpStatus::from('999');  // 抛出 ValueError
} catch (ValueError $e) {
    echo "Error: Invalid status code\n";
}
```

**输出**：

```
Name: OK, Value: 200
Status: NOT_FOUND
Invalid status code
Error: Invalid status code
```

**说明**：
- Backed Enums 有 `value` 属性，返回关联的标量值
- `from()` 方法从值创建枚举，不存在时抛出 `ValueError`
- `tryFrom()` 方法安全地从值创建，不存在时返回 `null`

### 示例 3：Backed Enums（整数枚举）

```php
<?php
declare(strict_types=1);

enum Priority: int
{
    case LOW = 1;
    case MEDIUM = 2;
    case HIGH = 3;
    case URGENT = 4;
}

$priority = Priority::HIGH;
echo "Priority: {$priority->name}, Value: {$priority->value}\n";  // Priority: HIGH, Value: 3

// 从整数创建枚举
$priority2 = Priority::from(2);
echo "Priority: {$priority2->name}\n";  // Priority: MEDIUM

// 枚举值比较
if ($priority === Priority::HIGH) {
    echo "High priority task\n";
}
```

**输出**：

```
Priority: HIGH, Value: 3
Priority: MEDIUM
High priority task
```

**说明**：
- 整数枚举使用 `int` 作为类型
- 可以存储到数据库的整数字段
- 提供了类型安全的优先级管理

### 示例 4：枚举方法

```php
<?php
declare(strict_types=1);

enum OrderStatus: string
{
    case PENDING = 'pending';
    case PROCESSING = 'processing';
    case SHIPPED = 'shipped';
    case DELIVERED = 'delivered';
    case CANCELLED = 'cancelled';
    
    // 实例方法
    public function canCancel(): bool
    {
        return $this === self::PENDING || $this === self::PROCESSING;
    }
    
    public function canShip(): bool
    {
        return $this === self::PROCESSING;
    }
    
    public function getNextStatus(): ?self
    {
        return match ($this) {
            self::PENDING => self::PROCESSING,
            self::PROCESSING => self::SHIPPED,
            self::SHIPPED => self::DELIVERED,
            default => null,
        };
    }
    
    // 静态方法
    public static function getActiveStatuses(): array
    {
        return [
            self::PENDING,
            self::PROCESSING,
            self::SHIPPED
        ];
    }
    
    public function getDisplayName(): string
    {
        return match ($this) {
            self::PENDING => '待处理',
            self::PROCESSING => '处理中',
            self::SHIPPED => '已发货',
            self::DELIVERED => '已送达',
            self::CANCELLED => '已取消',
        };
    }
}

$status = OrderStatus::PENDING;

echo "Can cancel: " . ($status->canCancel() ? "Yes" : "No") . "\n";  // Can cancel: Yes
echo "Can ship: " . ($status->canShip() ? "Yes" : "No") . "\n";  // Can ship: No
echo "Display name: " . $status->getDisplayName() . "\n";  // Display name: 待处理

$nextStatus = $status->getNextStatus();
if ($nextStatus !== null) {
    echo "Next status: {$nextStatus->name}\n";  // Next status: PROCESSING
}

$activeStatuses = OrderStatus::getActiveStatuses();
echo "Active statuses: " . count($activeStatuses) . "\n";  // Active statuses: 3
```

**输出**：

```
Can cancel: Yes
Can ship: No
Display name: 待处理
Next status: PROCESSING
Active statuses: 3
```

**说明**：
- 枚举可以定义实例方法，使用 `$this` 引用当前枚举值
- 枚举可以定义静态方法，通过 `EnumName::methodName()` 调用
- 枚举方法提供了丰富的功能，可以封装与枚举值相关的逻辑

### 示例 5：枚举实现接口

```php
<?php
declare(strict_types=1);

interface Colorful
{
    public function getColor(): string;
}

enum Color: string implements Colorful
{
    case RED = 'red';
    case GREEN = 'green';
    case BLUE = 'blue';
    case YELLOW = 'yellow';
    
    public function getColor(): string
    {
        return $this->value;
    }
    
    public function isWarm(): bool
    {
        return $this === self::RED || $this === self::YELLOW;
    }
}

$color = Color::RED;
echo "Color: {$color->getColor()}\n";  // Color: red
echo "Is warm: " . ($color->isWarm() ? "Yes" : "No") . "\n";  // Is warm: Yes

// 可以作为接口类型使用
function displayColor(Colorful $colorful): void
{
    echo "Displaying color: {$colorful->getColor()}\n";
}

displayColor($color);
```

**输出**：

```
Color: red
Is warm: Yes
Displaying color: red
```

**说明**：
- 枚举可以实现接口
- 实现了接口的枚举可以作为接口类型使用
- 提供了更灵活的扩展方式

### 示例 6：枚举与 match 表达式配合

```php
<?php
declare(strict_types=1);

enum PaymentStatus: string
{
    case PENDING = 'pending';
    case COMPLETED = 'completed';
    case FAILED = 'failed';
    case REFUNDED = 'refunded';
}

function getStatusMessage(PaymentStatus $status): string
{
    return match ($status) {
        PaymentStatus::PENDING => '支付待处理',
        PaymentStatus::COMPLETED => '支付成功',
        PaymentStatus::FAILED => '支付失败',
        PaymentStatus::REFUNDED => '已退款',
    };
}

function getStatusColor(PaymentStatus $status): string
{
    return match ($status) {
        PaymentStatus::PENDING => 'yellow',
        PaymentStatus::COMPLETED => 'green',
        PaymentStatus::FAILED => 'red',
        PaymentStatus::REFUNDED => 'gray',
    };
}

$status = PaymentStatus::COMPLETED;
echo "Message: " . getStatusMessage($status) . "\n";  // Message: 支付成功
echo "Color: " . getStatusColor($status) . "\n";  // Color: green
```

**输出**：

```
Message: 支付成功
Color: green
```

**说明**：
- 枚举与 match 表达式（详细说明见 [2.6.5 match 表达式](../../stage-02-language/chapter-06-expressions/section-05-match-expression.md)）配合使用，提供类型安全的模式匹配
- match 表达式会检查是否覆盖所有枚举值
- 比 switch 语句更简洁和安全

### 示例 7：枚举在类中的使用

```php
<?php
declare(strict_types=1);

enum UserRole: string
{
    case ADMIN = 'admin';
    case USER = 'user';
    case MODERATOR = 'moderator';
    case GUEST = 'guest';
    
    public function hasPermission(string $permission): bool
    {
        return match ($this) {
            self::ADMIN => true,  // 管理员有所有权限
            self::MODERATOR => in_array($permission, ['read', 'write', 'moderate']),
            self::USER => in_array($permission, ['read', 'write']),
            self::GUEST => $permission === 'read',
        };
    }
}

class User
{
    public function __construct(
        private string $name,
        private UserRole $role
    ) {}
    
    public function canAccess(string $permission): bool
    {
        return $this->role->hasPermission($permission);
    }
    
    public function getRole(): UserRole
    {
        return $this->role;
    }
}

$admin = new User("Admin User", UserRole::ADMIN);
$user = new User("Regular User", UserRole::USER);

echo "Admin can delete: " . ($admin->canAccess('delete') ? "Yes" : "No") . "\n";  // Yes
echo "User can delete: " . ($user->canAccess('delete') ? "Yes" : "No") . "\n";  // No
echo "User can write: " . ($user->canAccess('write') ? "Yes" : "No") . "\n";  // Yes
```

**输出**：

```
Admin can delete: Yes
User can delete: No
User can write: Yes
```

**说明**：
- 枚举可以用作类的属性类型
- 枚举方法可以封装与枚举值相关的业务逻辑
- 提供了类型安全的角色和权限管理

## 使用场景

### 场景 1：状态管理

枚举非常适合用于对象状态管理。

**示例**：订单状态

```php
<?php
declare(strict_types=1);

enum OrderStatus: string
{
    case PENDING = 'pending';
    case PROCESSING = 'processing';
    case SHIPPED = 'shipped';
    case DELIVERED = 'delivered';
    case CANCELLED = 'cancelled';
    
    public function canTransitionTo(self $newStatus): bool
    {
        return match ($this) {
            self::PENDING => in_array($newStatus, [self::PROCESSING, self::CANCELLED]),
            self::PROCESSING => in_array($newStatus, [self::SHIPPED, self::CANCELLED]),
            self::SHIPPED => $newStatus === self::DELIVERED,
            self::DELIVERED => false,  // 已送达不能再转换
            self::CANCELLED => false,  // 已取消不能再转换
        };
    }
}

class Order
{
    public function __construct(
        private int $id,
        private OrderStatus $status
    ) {}
    
    public function changeStatus(OrderStatus $newStatus): void
    {
        if (!$this->status->canTransitionTo($newStatus)) {
            throw new InvalidArgumentException("Cannot transition from {$this->status->value} to {$newStatus->value}");
        }
        $this->status = $newStatus;
    }
}
```

### 场景 2：配置选项

枚举可以用于配置选项，提供类型安全。

**示例**：应用配置

```php
<?php
declare(strict_types=1);

enum Environment: string
{
    case DEVELOPMENT = 'development';
    case STAGING = 'staging';
    case PRODUCTION = 'production';
    
    public function isProduction(): bool
    {
        return $this === self::PRODUCTION;
    }
    
    public function getDebugLevel(): string
    {
        return match ($this) {
            self::DEVELOPMENT => 'debug',
            self::STAGING => 'info',
            self::PRODUCTION => 'error',
        };
    }
}

class Config
{
    public function __construct(
        private Environment $environment
    ) {}
    
    public function isDebug(): bool
    {
        return !$this->environment->isProduction();
    }
}
```

### 场景 3：替代常量类

枚举可以替代传统的常量类，提供更好的类型安全。

**示例**：HTTP 方法

```php
<?php
declare(strict_types=1);

enum HttpMethod: string
{
    case GET = 'GET';
    case POST = 'POST';
    case PUT = 'PUT';
    case DELETE = 'DELETE';
    case PATCH = 'PATCH';
    
    public function isReadOnly(): bool
    {
        return $this === self::GET;
    }
    
    public function requiresBody(): bool
    {
        return in_array($this, [self::POST, self::PUT, self::PATCH]);
    }
}

// 传统常量类方式（不推荐）
// class HttpMethod
// {
//     public const GET = 'GET';
//     public const POST = 'POST';
//     // ...
// }
```

## 注意事项

### 枚举值的唯一性

- 枚举值名称必须唯一
- Backed Enums 的标量值也必须唯一

**示例**：

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
    // case DUPLICATE = 'active';  // 错误：值 'active' 已经存在
}
```

### Backed Enums 的类型限制

- Backed Enums 只能使用 `int` 或 `string` 类型
- 不能使用其他类型（如 `float`、`bool`、`array`）

**示例**：

```php
<?php
declare(strict_types=1);

// 正确：使用 string
enum Color: string
{
    case RED = 'red';
}

// 正确：使用 int
enum Priority: int
{
    case LOW = 1;
}

// 错误：不能使用 float
// enum Temperature: float
// {
//     case HOT = 37.5;
// }
```

### 枚举值比较

- 使用 `===` 进行严格比较
- 相同枚举值的实例是同一个对象

**示例**：

```php
<?php
declare(strict_types=1);

enum Status
{
    case ACTIVE;
}

$status1 = Status::ACTIVE;
$status2 = Status::ACTIVE;

var_dump($status1 === $status2);  // true（同一个对象）
```

### 枚举不能实例化

- 不能使用 `new` 关键字创建枚举实例
- 只能通过枚举值访问

**示例**：

```php
<?php
declare(strict_types=1);

enum Status
{
    case ACTIVE;
}

// 正确：使用枚举值
$status = Status::ACTIVE;

// 错误：不能实例化
// $status = new Status();  // 错误：Cannot instantiate enum
```

## 常见问题

### 问题 1：从值创建枚举失败

**错误信息**：`ValueError: X is not a valid backing value for enum Y`

**原因**：使用 `from()` 方法创建枚举时，提供的值不存在

**解决方案**：
- 使用 `tryFrom()` 方法进行安全创建
- 检查返回值是否为 `null`
- 或使用 try-catch 捕获异常

**示例**：

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

// 方案 1：使用 tryFrom（推荐）
$status = Status::tryFrom('invalid');
if ($status === null) {
    echo "Invalid status\n";
}

// 方案 2：使用 try-catch
try {
    $status = Status::from('invalid');
} catch (ValueError $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### 问题 2：枚举值类型错误

**错误信息**：`TypeError` 或类型不匹配

**原因**：传递了错误的类型给枚举参数

**解决方案**：
- 确保传递正确的枚举类型
- 使用类型声明确保类型安全

**示例**：

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case ACTIVE = 'active';
}

function processStatus(Status $status): void
{
    echo "Status: {$status->value}\n";
}

// 错误：传递字符串而不是枚举
// processStatus('active');  // TypeError

// 正确：传递枚举值
processStatus(Status::ACTIVE);

// 正确：从值创建枚举
processStatus(Status::from('active'));
```

### 问题 3：枚举值在 match 表达式中未完全覆盖

**错误信息**：`UnhandledMatchError: Unhandled match value`

**原因**：match 表达式没有覆盖所有枚举值

**解决方案**：
- 在 match 表达式中覆盖所有枚举值
- 使用 `default` 分支处理未覆盖的情况

**示例**：

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
    case PENDING = 'pending';
}

function getMessage(Status $status): string
{
    // 错误：没有覆盖所有枚举值
    // return match ($status) {
    //     Status::ACTIVE => 'Active',
    //     Status::INACTIVE => 'Inactive',
    //     // 缺少 Status::PENDING
    // };
    
    // 正确：覆盖所有枚举值
    return match ($status) {
        Status::ACTIVE => 'Active',
        Status::INACTIVE => 'Inactive',
        Status::PENDING => 'Pending',
    };
    
    // 或使用 default 分支
    // return match ($status) {
    //     Status::ACTIVE => 'Active',
    //     default => 'Other',
    // };
}
```

## 最佳实践

### 1. 使用枚举替代常量类

枚举提供了更好的类型安全和功能。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用枚举
enum UserRole: string
{
    case ADMIN = 'admin';
    case USER = 'user';
}

// 不推荐：使用常量类
// class UserRole
// {
//     public const ADMIN = 'admin';
//     public const USER = 'user';
// }
```

### 2. 为枚举添加方法封装逻辑

将与枚举值相关的逻辑封装在枚举方法中。

**示例**：

```php
<?php
declare(strict_types=1);

enum Priority: int
{
    case LOW = 1;
    case MEDIUM = 2;
    case HIGH = 3;
    
    public function getColor(): string
    {
        return match ($this) {
            self::LOW => 'green',
            self::MEDIUM => 'yellow',
            self::HIGH => 'red',
        };
    }
    
    public function isHighPriority(): bool
    {
        return $this === self::HIGH;
    }
}
```

### 3. 使用 Backed Enums 存储到数据库

Backed Enums 可以方便地存储到数据库。

**示例**：

```php
<?php
declare(strict_types=1);

enum OrderStatus: string
{
    case PENDING = 'pending';
    case COMPLETED = 'completed';
}

// 存储到数据库：使用 value 属性
$statusValue = $order->status->value;  // 'pending'

// 从数据库读取：使用 from() 方法
$status = OrderStatus::from($row['status']);
```

### 4. 与 match 表达式配合使用

枚举与 match 表达式配合，提供类型安全的模式匹配。

**示例**：

```php
<?php
declare(strict_types=1);

enum Status: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

function getStatusMessage(Status $status): string
{
    return match ($status) {
        Status::ACTIVE => '系统运行中',
        Status::INACTIVE => '系统已停用',
    };
}
```

### 5. 实现接口提供扩展性

枚举可以实现接口，提供更灵活的扩展方式。

**示例**：

```php
<?php
declare(strict_types=1);

interface Describable
{
    public function getDescription(): string;
}

enum Status: string implements Describable
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
    
    public function getDescription(): string
    {
        return match ($this) {
            self::ACTIVE => '系统运行中',
            self::INACTIVE => '系统已停用',
        };
    }
}
```

## 对比分析

### 枚举 vs 常量类

| 特性         | 枚举（PHP 8.1+）              | 常量类                         |
|:-------------|:-------------------------------|:-------------------------------|
| **类型安全**  | ✅ 编译时类型检查               | ❌ 运行时检查                   |
| **方法支持**  | ✅ 可以定义方法                 | ❌ 不能定义方法                 |
| **接口实现**  | ✅ 可以实现接口                 | ❌ 不能实现接口                 |
| **IDE 支持**  | ✅ 更好的代码提示               | ❌ 有限的代码提示               |
| **值检查**    | ✅ 只能使用定义的枚举值          | ❌ 可以使用任意值               |
| **序列化**    | ✅ 支持序列化                   | ❌ 需要手动处理                 |

### 基础枚举 vs Backed Enums

| 特性         | 基础枚举                       | Backed Enums                   |
|:-------------|:-------------------------------|:-------------------------------|
| **标量值**    | ❌ 没有标量值                   | ✅ 有标量值（int 或 string）    |
| **数据库存储**| ❌ 不适合直接存储               | ✅ 适合存储到数据库             |
| **from()**    | ❌ 不支持                       | ✅ 支持从值创建                 |
| **value 属性**| ❌ 没有                         | ✅ 有 value 属性                |
| **适用场景**  | 纯状态标识                      | 需要存储值的场景                |

### 枚举 vs 类常量

| 特性         | 枚举                           | 类常量                         |
|:-------------|:-------------------------------|:-------------------------------|
| **类型**      | 独立类型                        | 标量值（string/int）            |
| **类型检查**  | 编译时检查                      | 运行时检查（容易出错）          |
| **方法**      | 可以定义方法                    | 不能定义方法                    |
| **值限制**    | 只能使用定义的枚举值            | 可以使用任意值                  |
| **IDE 支持**  | 更好的支持                      | 有限的支持                      |

## 练习任务

1. **创建状态枚举**：定义一个 `OrderStatus` 枚举，包含订单的各种状态（PENDING、PROCESSING、SHIPPED、DELIVERED、CANCELLED），添加 `canCancel()` 和 `canShip()` 方法。

2. **创建 Backed Enum**：定义一个 `Priority` 整数枚举（LOW=1, MEDIUM=2, HIGH=3），添加 `getColor()` 方法返回优先级对应的颜色。

3. **枚举实现接口**：创建一个 `Colorful` 接口，定义一个 `Color` 字符串枚举实现该接口，提供 `getColor()` 方法。

4. **枚举与 match 表达式**：使用枚举和 match 表达式创建一个函数，根据订单状态返回对应的消息和颜色。

5. **数据库存储练习**：创建一个 `UserRole` 字符串枚举，演示如何将枚举值存储到数据库，以及如何从数据库读取并转换为枚举。

## 相关章节

- **[2.6.5 match 表达式](../../stage-02-language/chapter-06-expressions/section-05-match-expression.md)**：了解 match 表达式的详细用法
- **[3.1.1 类与对象基础](../chapter-01-classes/section-01-basics.md)**：回顾类的基础知识
- **[3.3.2 接口（Interface）](../chapter-03-oop-features/section-02-interfaces.md)**：了解接口的实现规则
