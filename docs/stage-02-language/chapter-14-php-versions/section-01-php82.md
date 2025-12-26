# 2.14.1 PHP 8.2 新特性与示例

## 概述

PHP 8.2 引入了多项新特性，提升了语言的功能和性能。本节详细介绍 Readonly 类、独立类型、枚举常量表达式、敏感参数、全新 Random 扩展等重点能力。

理解 PHP 8.2 的新特性对于编写现代化、高性能的 PHP 代码非常重要。掌握这些新特性可以帮助提高代码质量、类型安全性和开发效率。

## 特性

- **Readonly 类**：整个类只读，所有属性自动为 readonly
- **独立类型**：更灵活的类型声明，支持交集和并集类型组合
- **枚举常量表达式**：枚举中使用常量表达式
- **敏感参数**：标记敏感参数，避免在堆栈跟踪中泄露
- **Random 扩展**：全新的随机数生成扩展，提供更好的性能和功能

## 语法/定义

### Readonly 类

**语法**：`readonly class ClassName { ... }`

**要求**：PHP 8.2+

**特点**：
- 类的所有属性自动为 readonly
- 属性必须在构造器中初始化
- 不能有非 readonly 属性

### 独立类型（DNF Types）

**语法**：`(A&B)|C` 或 `A&(B|C)`

**要求**：PHP 8.2+

**特点**：
- 支持交集类型（`&`）和并集类型（`|`）的组合
- 必须使用括号明确优先级
- 提供更灵活的类型声明

### 枚举常量表达式

**语法**：在枚举中使用常量表达式

**要求**：PHP 8.2+

**特点**：
- 枚举值可以使用常量表达式
- 可以使用 `self::` 引用其他枚举值
- 支持计算枚举值

### 敏感参数标记

**语法**：`#[\SensitiveParameter]`

**要求**：PHP 8.2+

**特点**：
- 标记敏感参数（如密码、密钥）
- 在堆栈跟踪中隐藏参数值
- 提高安全性

## 基本用法

### 示例 1：Readonly 类

```php
<?php
declare(strict_types=1);

// Readonly 类
readonly class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $email
    ) {}
}

// 使用
$user = new User('John', 25, 'john@example.com');
echo $user->name . "\n";  // John

// 错误：不能修改属性
// $user->name = 'Jane';  // Fatal error
```

### 示例 2：独立类型

```php
<?php
declare(strict_types=1);

interface A {}
interface B {}
class C {}

// 独立类型：交集和并集的组合
function process((A&B)|C $value): void
{
    // $value 必须是 (A 和 B 的交集) 或 C
}

// 使用类型别名提高可读性
type AB = A&B;
type ABC = AB|C;

function process2(ABC $value): void
{
    // 等价于 (A&B)|C
}
```

### 示例 3：枚举常量表达式

```php
<?php
declare(strict_types=1);

enum Status: int
{
    case ACTIVE = 1;
    case INACTIVE = 2;
    case PENDING = self::ACTIVE->value + 2;  // 3
    case SUSPENDED = self::INACTIVE->value * 2;  // 4
}

// 使用
$status = Status::PENDING;
echo $status->value . "\n";  // 3
```

### 示例 4：敏感参数

```php
<?php
declare(strict_types=1);

function login(
    string $username,
    #[\SensitiveParameter] string $password
): bool
{
    // 密码参数在堆栈跟踪中会被隐藏
    return authenticate($username, $password);
}

// 使用
login('user', 'secret123');
// 如果发生异常，堆栈跟踪中不会显示密码值
```

### 示例 5：Random 扩展

```php
<?php
declare(strict_types=1);

use Random\Randomizer;

$randomizer = new Randomizer();

// 生成随机整数
$number = $randomizer->getInt(1, 100);

// 生成随机字符串
$string = $randomizer->getBytes(16);

// 打乱数组
$array = [1, 2, 3, 4, 5];
$shuffled = $randomizer->shuffleArray($array);

// 选择随机元素
$element = $randomizer->pickArrayKeys($array, 2);
```

## 完整代码示例

### 示例 1：Readonly 类实现

```php
<?php
declare(strict_types=1);

readonly class Point
{
    public function __construct(
        public float $x,
        public float $y
    ) {}
    
    public function distanceTo(Point $other): float
    {
        $dx = $this->x - $other->x;
        $dy = $this->y - $other->y;
        return sqrt($dx * $dx + $dy * $dy);
    }
}

// 使用
$p1 = new Point(0, 0);
$p2 = new Point(3, 4);
echo $p1->distanceTo($p2) . "\n";  // 5.0
```

### 示例 2：独立类型使用

```php
<?php
declare(strict_types=1);

interface Countable {}
interface Iterator {}

class Collection implements Countable, Iterator {}

// 独立类型：必须是 Countable 和 Iterator 的交集，或者是 Collection
function process((Countable&Iterator)|Collection $value): void
{
    if ($value instanceof Collection) {
        echo "Collection\n";
    } else {
        echo "Countable and Iterator\n";
    }
}

// 使用
$collection = new Collection();
process($collection);  // Collection
```

### 示例 3：枚举常量表达式

```php
<?php
declare(strict_types=1);

enum HttpStatus: int
{
    case OK = 200;
    case CREATED = 201;
    case BAD_REQUEST = 400;
    case UNAUTHORIZED = 401;
    case NOT_FOUND = 404;
    case SERVER_ERROR = 500;
    
    // 使用常量表达式
    case ACCEPTED = self::CREATED->value + 1;  // 202
    case FORBIDDEN = self::UNAUTHORIZED->value + 2;  // 403
}

// 使用
$status = HttpStatus::ACCEPTED;
echo $status->value . "\n";  // 202
```

### 示例 4：敏感参数使用

```php
<?php
declare(strict_types=1);

class AuthService
{
    public function authenticate(
        string $username,
        #[\SensitiveParameter] string $password
    ): bool
    {
        // 认证逻辑
        return $this->verifyCredentials($username, $password);
    }
    
    public function changePassword(
        string $username,
        #[\SensitiveParameter] string $oldPassword,
        #[\SensitiveParameter] string $newPassword
    ): bool
    {
        // 修改密码逻辑
        return true;
    }
}

// 使用
$auth = new AuthService();
$auth->authenticate('user', 'secret123');
// 如果发生异常，堆栈跟踪中不会显示密码
```

## 使用场景

### Readonly 类

- **不可变对象**：创建不可变的数据对象
- **值对象**：实现值对象模式
- **数据传输**：用于数据传输对象（DTO）

### 独立类型

- **复杂类型约束**：需要复杂类型约束的场景
- **接口组合**：需要多个接口组合的场景
- **类型安全**：提高类型安全性

### 枚举常量表达式

- **计算枚举值**：需要基于其他枚举值计算新值
- **枚举关系**：表达枚举值之间的关系
- **动态枚举值**：动态生成枚举值

### 敏感参数

- **密码处理**：处理密码、密钥等敏感信息
- **安全函数**：安全相关的函数
- **API 密钥**：处理 API 密钥等敏感数据

## 注意事项

### 版本要求

- **PHP 8.2+**：所有新特性都需要 PHP 8.2 或更高版本
- **兼容性**：注意向后兼容性
- **迁移**：评估迁移成本

### Readonly 类限制

- **属性初始化**：所有属性必须在构造器中初始化
- **不能修改**：创建后不能修改属性
- **继承**：readonly 类可以继承，但子类也必须是 readonly

### 独立类型复杂度

- **可读性**：复杂类型表达式可能影响可读性
- **类型别名**：使用类型别名提高可读性
- **工具支持**：确保工具支持独立类型

## 常见问题

### 问题 1：Readonly 类属性初始化

**症状**：属性未初始化错误

**原因**：readonly 类属性必须在构造器中初始化

**错误示例**：

```php
<?php
declare(strict_types=1);

readonly class User
{
    public string $name;  // 错误：必须初始化
    
    public function setName(string $name): void
    {
        $this->name = $name;  // 错误：不能在构造器外赋值
    }
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public string $name  // 正确：在构造器中初始化
    ) {}
}
```

### 问题 2：独立类型语法错误

**症状**：类型声明语法错误

**原因**：独立类型必须使用括号

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：缺少括号
function process(A&B|C $value): void
{
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 正确：使用括号
function process((A&B)|C $value): void
{
}

// 或使用类型别名
type AB = A&B;
function process2(AB|C $value): void
{
}
```

## 最佳实践

### 新特性使用

- **了解场景**：了解新特性的适用场景
- **评估成本**：评估迁移成本
- **逐步采用**：逐步采用新特性
- **参考文档**：参考官方文档

### 代码质量

- **类型安全**：利用新特性提高类型安全性
- **代码可读性**：使用类型别名提高可读性
- **安全性**：使用敏感参数标记提高安全性

## 相关章节

- **2.4.5 类型声明**：了解类型声明基础
- **阶段三：面向对象编程基础**：了解类和枚举
- **2.13.1 PSR 标准**：了解编码标准

## 练习任务

1. **Readonly 类练习**：
   - 创建 readonly 类
   - 实现不可变对象
   - 测试属性访问
   - 理解限制

2. **独立类型练习**：
   - 使用独立类型声明
   - 创建类型别名
   - 测试类型检查
   - 理解类型组合

3. **枚举常量表达式练习**：
   - 在枚举中使用常量表达式
   - 创建计算枚举值
   - 测试枚举使用
   - 理解表达式规则

4. **实际应用练习**：
   - 在项目中使用新特性
   - 重构现有代码
   - 测试兼容性
   - 评估效果

5. **综合练习**：
   - 创建一个使用 PHP 8.2 新特性的项目
   - 实现各种新特性
   - 进行代码审查
   - 测试性能改进
