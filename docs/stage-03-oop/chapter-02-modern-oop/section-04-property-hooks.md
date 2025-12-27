# 3.2.4 属性钩子与静态方法

## 概述

属性钩子（Property Hooks）是 PHP 8.4 引入的特性，允许为属性定义 getter 和 setter 逻辑，无需单独的方法。这一特性类似于其他语言（如 C#、Python）的属性访问器，可以简化代码，减少样板代码。本节详细介绍属性钩子的语法、不对称可见性、与传统的 Getter/Setter 方法的对比，以及静态方法的合理使用。

属性钩子提供了在属性访问时执行自定义逻辑的能力，如验证、格式化、计算等。这对于需要控制属性访问的场景非常有用。不对称可见性允许属性的读写操作具有不同的可见性，提供了更灵活的访问控制。

本节还介绍静态方法的合理使用。静态方法是属于类而不是对象实例的方法，不需要创建对象就可以调用。理解何时使用静态方法、何时使用实例方法，以及避免静态属性的滥用，对于编写高质量的代码非常重要。

**主要内容**：
- 属性钩子的基础语法和使用（PHP 8.4+）
- Getter 和 Setter 钩子的定义
- 不对称可见性的语法和使用（PHP 8.4+）
- 属性钩子与传统 Getter/Setter 的对比
- 静态方法的合理使用
- 静态工厂方法
- 静态属性滥用的风险
- 使用场景和最佳实践

## 特性

- **属性钩子**：PHP 8.4+ 特性，属性的 getter 和 setter
- **不对称可见性**：PHP 8.4+ 特性，属性的读写可见性可以不同
- **代码简化**：减少 Getter/Setter 方法的样板代码
- **灵活控制**：可以在属性访问时执行自定义逻辑
- **静态方法**：属于类的方法，不需要对象实例

## 语法/定义

### 属性钩子（Property Hooks）

**语法**：`private Type $propertyName { get { ... } set(Type $value) { ... } }`

**组成部分**：
- 属性声明：`private Type $propertyName`
- `get` 钩子：定义读取属性时的逻辑
- `set` 钩子：定义写入属性时的逻辑（参数类型必须与属性类型匹配）

**特点**：
- PHP 8.4+ 引入的特性
- 可以在属性访问时执行自定义逻辑
- 减少 Getter/Setter 方法的样板代码
- `get` 和 `set` 钩子都是可选的

### 不对称可见性（Asymmetric Visibility）

**语法**：`public(get) private(set) Type $propertyName` 或 `public Type $propertyName { get; private set; }`

**组成部分**：
- `get` 可见性：定义读取属性的可见性
- `set` 可见性：定义写入属性的可见性

**特点**：
- PHP 8.4+ 引入的特性
- 允许属性的读写操作具有不同的可见性
- 可以实现只读属性、只写属性等模式

### 静态方法

**语法**：`public static function methodName(): ReturnType { ... }`

**调用方式**：
- 类内部：`self::methodName()` 或 `static::methodName()`
- 类外部：`ClassName::methodName()`

**特点**：
- 属于类而不是对象实例
- 不需要对象实例就可以调用
- 不能使用 `$this`

## 基本用法

### 示例 1：基础属性钩子

```php
<?php
declare(strict_types=1);

class User
{
    private string $name {
        get {
            return $this->name ?? 'Unknown';
        }
        set(string $value) {
            if (empty(trim($value))) {
                throw new InvalidArgumentException("Name cannot be empty");
            }
            $this->name = trim($value);
        }
    }
    
    private int $age {
        get {
            return $this->age;
        }
        set(int $value) {
            if ($value < 0 || $value > 150) {
                throw new InvalidArgumentException("Age must be between 0 and 150");
            }
            $this->age = $value;
        }
    }
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;  // 调用 setter 钩子
        $this->age = $age;    // 调用 setter 钩子
    }
    
    public function getName(): string
    {
        return $this->name;  // 调用 getter 钩子
    }
    
    public function getAge(): int
    {
        return $this->age;  // 调用 getter 钩子
    }
}

$user = new User("  John Doe  ", 25);
echo "Name: " . $user->getName() . "\n";  // Name: John Doe（自动 trim）
echo "Age: " . $user->getAge() . "\n";    // Age: 25
```

**输出**：

```
Name: John Doe
Age: 25
```

**说明**：
- 属性钩子允许在属性访问时执行自定义逻辑
- `set` 钩子可以添加验证和格式化逻辑
- `get` 钩子可以返回计算值或默认值

### 示例 2：计算属性

```php
<?php
declare(strict_types=1);

class Temperature
{
    private float $celsius;
    
    // 计算属性：Fahrenheit
    public float $fahrenheit {
        get => ($this->celsius * 9/5) + 32;
        set(float $fahrenheit) {
            $this->celsius = ($fahrenheit - 32) * 5/9;
        }
    }
    
    public function __construct(float $celsius)
    {
        $this->celsius = $celsius;
    }
    
    public function getCelsius(): float
    {
        return $this->celsius;
    }
}

$temp = new Temperature(25);
echo "Celsius: {$temp->getCelsius()}\n";           // Celsius: 25
echo "Fahrenheit: {$temp->fahrenheit}\n";          // Fahrenheit: 77（自动计算）

$temp->fahrenheit = 86;
echo "Celsius after setting Fahrenheit: {$temp->getCelsius()}\n";  // Celsius: 30（自动转换）
```

**输出**：

```
Celsius: 25
Fahrenheit: 77
Celsius after setting Fahrenheit: 30
```

**说明**：
- 属性钩子可以实现计算属性
- `get` 钩子可以返回计算值，不存储实际属性
- `set` 钩子可以将值转换为内部存储格式

### 示例 3：不对称可见性

```php
<?php
declare(strict_types=1);

class BankAccount
{
    // 余额：外部可以读取，但不能直接写入
    public float $balance {
        get => $this->balance;
        private set;
    }
    
    // 账户号：只读
    public string $accountNumber {
        get;
        private set;
    }
    
    private float $balance = 0.0;
    
    public function __construct(string $accountNumber)
    {
        $this->accountNumber = $accountNumber;
    }
    
    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Amount must be positive");
        }
        $this->balance += $amount;  // 内部可以写入
    }
    
    public function withdraw(float $amount): bool
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException("Amount must be positive");
        }
        if ($this->balance >= $amount) {
            $this->balance -= $amount;
            return true;
        }
        return false;
    }
}

$account = new BankAccount("ACC001");
$account->deposit(1000);
echo "Balance: {$account->balance}\n";  // Balance: 1000（可以读取）

// $account->balance = 9999;  // 错误：Cannot access private setter

$account->withdraw(300);
echo "Balance after withdrawal: {$account->balance}\n";  // Balance: 700
```

**输出**：

```
Balance: 1000
Balance after withdrawal: 700
```

**说明**：
- 不对称可见性允许属性的读写操作具有不同的可见性
- 外部可以读取 `balance`，但不能直接写入
- 只能通过类的方法（`deposit`、`withdraw`）修改余额

### 示例 4：只读属性（使用不对称可见性）

```php
<?php
declare(strict_types=1);

class User
{
    // ID：只读，外部可以读取，但不能写入
    public string $id {
        get;
        private set;
    }
    
    private string $id;
    
    public function __construct(string $id)
    {
        $this->id = $id;
    }
    
    public function getId(): string
    {
        return $this->id;  // 可以通过方法访问
    }
}

$user = new User("user123");
echo "ID: {$user->id}\n";         // ID: user123（可以读取）
echo "ID: {$user->getId()}\n";    // ID: user123（也可以通过方法）

// $user->id = "user456";  // 错误：Cannot access private setter
```

**输出**：

```
ID: user123
ID: user123
```

**说明**：
- 使用不对称可见性可以实现只读属性
- 外部可以读取，但不能写入
- 类似 `readonly` 属性的效果，但提供了更多的灵活性

### 示例 5：属性钩子 vs 传统 Getter/Setter

```php
<?php
declare(strict_types=1);

// 传统方式：使用 Getter/Setter 方法
class UserTraditional
{
    private string $name;
    
    public function getName(): string
    {
        return $this->name ?? 'Unknown';
    }
    
    public function setName(string $name): void
    {
        if (empty(trim($name))) {
            throw new InvalidArgumentException("Name cannot be empty");
        }
        $this->name = trim($name);
    }
}

// PHP 8.4+ 方式：使用属性钩子
class UserModern
{
    private string $name {
        get {
            return $this->name ?? 'Unknown';
        }
        set(string $value) {
            if (empty(trim($value))) {
                throw new InvalidArgumentException("Name cannot be empty");
            }
            $this->name = trim($value);
        }
    }
    
    public function getName(): string
    {
        return $this->name;  // 调用 getter 钩子
    }
    
    public function setName(string $name): void
    {
        $this->name = $name;  // 调用 setter 钩子
    }
}

$user = new UserModern();
$user->setName("  John  ");
echo "Name: " . $user->getName() . "\n";  // Name: John（自动 trim）
```

**输出**：

```
Name: John
```

**说明**：
- 属性钩子减少了样板代码
- 逻辑更加集中，属性定义和访问逻辑在一起
- 仍然可以通过方法访问，提供向后兼容

### 示例 6：静态方法的使用

```php
<?php
declare(strict_types=1);

class MathUtils
{
    // 静态工具方法
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
    
    public static function multiply(int $a, int $b): int
    {
        return $a * $b;
    }
    
    public static function factorial(int $n): int
    {
        if ($n <= 1) {
            return 1;
        }
        return $n * self::factorial($n - 1);
    }
}

// 不需要创建对象，直接调用静态方法
echo "5 + 3 = " . MathUtils::add(5, 3) . "\n";
echo "5 * 3 = " . MathUtils::multiply(5, 3) . "\n";
echo "5! = " . MathUtils::factorial(5) . "\n";
```

**输出**：

```
5 + 3 = 8
5 * 3 = 15
5! = 120
```

**说明**：
- 静态方法不需要对象实例就可以调用
- 适合工具类和不依赖对象状态的方法
- 使用 `ClassName::methodName()` 调用

### 示例 7：静态工厂方法

```php
<?php
declare(strict_types=1);

class DateTimeHelper
{
    private function __construct(
        private DateTime $datetime
    ) {}
    
    // 静态工厂方法：从时间戳创建
    public static function fromTimestamp(int $timestamp): self
    {
        return new self(new DateTime("@{$timestamp}"));
    }
    
    // 静态工厂方法：从字符串创建
    public static function fromString(string $datetime): self
    {
        return new self(new DateTime($datetime));
    }
    
    // 静态工厂方法：当前时间
    public static function now(): self
    {
        return new self(new DateTime());
    }
    
    public function format(string $format): string
    {
        return $this->datetime->format($format);
    }
}

$dt1 = DateTimeHelper::fromTimestamp(time());
$dt2 = DateTimeHelper::fromString('2024-01-15');
$dt3 = DateTimeHelper::now();

echo "From timestamp: " . $dt1->format('Y-m-d H:i:s') . "\n";
echo "From string: " . $dt2->format('Y-m-d') . "\n";
echo "Now: " . $dt3->format('Y-m-d H:i:s') . "\n";
```

**输出**：

```
From timestamp: 2025-01-15 12:00:00
From string: 2024-01-15
Now: 2025-01-15 12:00:00
```

**说明**：
- 静态工厂方法提供灵活的对象创建方式
- 可以有多个工厂方法，提供不同的创建方式
- 构造函数可以是私有的，强制使用工厂方法

## 使用场景

### 场景 1：属性验证和格式化

属性钩子适合需要验证和格式化属性的场景。

**示例**：邮箱验证

```php
<?php
declare(strict_types=1);

class User
{
    private string $email {
        get => $this->email;
        set(string $value) {
            $normalized = strtolower(trim($value));
            if (!filter_var($normalized, FILTER_VALIDATE_EMAIL)) {
                throw new InvalidArgumentException("Invalid email address: {$value}");
            }
            $this->email = $normalized;
        }
    }
    
    public function __construct(string $email)
    {
        $this->email = $email;  // 自动验证和格式化
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
}

$user = new User("  USER@EXAMPLE.COM  ");
echo "Email: {$user->getEmail()}\n";  // Email: user@example.com（自动转换为小写和 trim）
```

### 场景 2：计算属性

属性钩子可以实现计算属性，不存储实际值。

**示例**：订单总价计算

```php
<?php
declare(strict_types=1);

class OrderItem
{
    private float $price;
    private int $quantity;
    
    public float $total {
        get => $this->price * $this->quantity;
        set(float $value) {
            // 根据总价反推单价
            if ($this->quantity > 0) {
                $this->price = $value / $this->quantity;
            }
        }
    }
    
    public function __construct(float $price, int $quantity)
    {
        $this->price = $price;
        $this->quantity = $quantity;
    }
    
    public function getPrice(): float { return $this->price; }
    public function getQuantity(): int { return $this->quantity; }
}

$item = new OrderItem(10.0, 3);
echo "Total: {$item->total}\n";  // Total: 30（自动计算）

$item->total = 50;
echo "Price after setting total: {$item->getPrice()}\n";  // Price: 16.666...（自动计算）
```

### 场景 3：只读属性模式

使用不对称可见性实现只读属性。

**示例**：配置对象

```php
<?php
declare(strict_types=1);

class DatabaseConfig
{
    public string $host {
        get;
        private set;
    }
    
    public int $port {
        get;
        private set;
    }
    
    private string $host;
    private int $port;
    
    public function __construct(string $host, int $port)
    {
        $this->host = $host;
        $this->port = $port;
    }
}

$config = new DatabaseConfig("localhost", 3306);
echo "Host: {$config->host}, Port: {$config->port}\n";

// $config->host = "other";  // 错误：Cannot access private setter
```

### 场景 4：工具类使用静态方法

静态方法适合不需要对象状态的工具类。

**示例**：字符串工具类

```php
<?php
declare(strict_types=1);

class StringUtils
{
    public static function capitalize(string $str): string
    {
        return ucfirst(strtolower($str));
    }
    
    public static function slug(string $str): string
    {
        $str = strtolower($str);
        $str = preg_replace('/[^a-z0-9]+/', '-', $str);
        return trim($str, '-');
    }
    
    public static function camelCase(string $str): string
    {
        $words = explode('_', $str);
        $result = array_shift($words);
        foreach ($words as $word) {
            $result .= ucfirst($word);
        }
        return $result;
    }
}

echo StringUtils::capitalize("hello world") . "\n";
echo StringUtils::slug("Hello, World!") . "\n";
echo StringUtils::camelCase("hello_world") . "\n";
```

## 注意事项

### 属性钩子的限制

#### 1. 版本要求

- **PHP 版本**：需要 PHP 8.4 或更高版本
- **兼容性**：旧版本 PHP 不支持此特性

#### 2. 性能考虑

- 属性钩子有性能开销
- 每次属性访问都会执行钩子逻辑
- 只在需要时使用，避免过度使用

#### 3. 存储属性

- 属性钩子仍然需要存储属性
- 不能完全替代存储属性
- `get` 钩子可以返回计算值，但不推荐完全避免存储

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    // 错误：没有存储属性
    // public string $name {
    //     get {
    //         return "Computed";  // 错误：需要存储属性
    //     }
    // }
    
    // 正确：有存储属性
    private string $name {
        get {
            return $this->name ?? 'Default';
        }
        set(string $value) {
            $this->name = $value;
        }
    }
}
```

### 静态方法的限制

#### 1. 不能使用 $this

- 静态方法没有对象实例
- 不能使用 `$this`
- 不能访问实例属性

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    private string $name = "Test";
    
    public static function staticMethod(): void
    {
        // echo $this->name;  // 错误：不能使用 $this
        // echo self::$name;  // 错误：$name 不是静态属性
    }
}
```

#### 2. 静态属性的共享

- 静态属性在所有对象间共享
- 可能导致意外的状态共享
- 应该谨慎使用

### 静态属性滥用的风险

#### 问题 1：全局状态

静态属性创建全局状态，可能导致：
- 测试困难
- 线程安全问题
- 代码耦合

**示例**：

```php
<?php
declare(strict_types=1);

// 不好的设计：使用静态属性存储全局状态
class BadDesign
{
    public static string $currentUser = "";
    
    public static function setCurrentUser(string $user): void
    {
        self::$currentUser = $user;
    }
}

// 好的设计：使用依赖注入
class GoodDesign
{
    private string $currentUser;
    
    public function __construct(string $user)
    {
        $this->currentUser = $user;
    }
}
```

## 常见问题

### 问题 1：属性钩子版本不兼容

**错误信息**：`Parse error: syntax error`

**原因**：使用了低于 PHP 8.4 的版本

**解决方案**：
- 升级 PHP 版本到 8.4 或更高
- 检查 PHP 版本：`php -v`
- 使用传统 Getter/Setter 方法作为替代

### 问题 2：属性钩子中的递归

**错误信息**：`Fatal error: Maximum function nesting level reached`

**原因**：在 `get` 或 `set` 钩子中访问同一个属性导致无限递归

**解决方案**：
- 使用私有存储属性
- 避免在钩子中访问同一属性

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    private string $name;  // 私有存储属性
    
    public string $name {
        get {
            return $this->name;  // 访问私有存储属性，不会递归
        }
        set(string $value) {
            $this->name = $value;  // 设置私有存储属性
        }
    }
}
```

### 问题 3：静态方法中使用 $this

**错误信息**：`Fatal error: Using $this when not in object context`

**原因**：在静态方法中使用了 `$this`

**解决方案**：
- 静态方法不能使用 `$this`
- 如果需要访问实例数据，使用实例方法
- 如果需要共享数据，使用静态属性

## 最佳实践

### 1. 合理使用属性钩子

- **适合场景**：需要验证、格式化、计算的属性
- **不适合场景**：简单属性（直接访问即可）
- **性能考虑**：注意性能开销

**示例**：

```php
<?php
declare(strict_types=1);

// 好的使用：需要验证和格式化
class GoodExample
{
    private string $email {
        get => $this->email;
        set(string $value) {
            // 验证和格式化逻辑
            $this->email = strtolower(trim($value));
        }
    }
}

// 不好的使用：简单属性不需要钩子
class BadExample
{
    private string $name {
        get => $this->name;
        set(string $value) {
            $this->name = $value;  // 没有额外逻辑，不需要钩子
        }
    }
}
```

### 2. 使用不对称可见性控制访问

- 实现只读属性模式
- 提供更灵活的访问控制
- 替代传统的 Getter/Setter 方法

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    // 只读 ID
    public string $id {
        get;
        private set;
    }
    
    private string $id;
    
    public function __construct(string $id)
    {
        $this->id = $id;
    }
}
```

### 3. 静态方法用于工具类

- 工具类使用静态方法
- 不需要对象状态的方法使用静态方法
- 提供清晰的 API

**示例**：

```php
<?php
declare(strict_types=1);

class MathUtils
{
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
    
    public static function max(int ...$numbers): int
    {
        return max($numbers);
    }
}
```

### 4. 避免静态属性滥用

- 避免使用静态属性存储全局状态
- 优先使用依赖注入
- 静态属性只用于真正的类级别数据

**示例**：

```php
<?php
declare(strict_types=1);

// 不好的设计
class BadExample
{
    public static ?User $currentUser = null;
}

// 好的设计：使用依赖注入
class GoodExample
{
    private ?User $currentUser;
    
    public function __construct(?User $user = null)
    {
        $this->currentUser = $user;
    }
}
```

### 5. 使用静态工厂方法

静态工厂方法提供灵活的对象创建方式。

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    private function __construct(
        private string $name,
        private string $email
    ) {}
    
    public static function create(string $name, string $email): self
    {
        return new self($name, $email);
    }
    
    public static function fromArray(array $data): self
    {
        return new self($data['name'] ?? '', $data['email'] ?? '');
    }
}
```

## 对比分析

### 属性钩子 vs 传统 Getter/Setter

| 特性         | 属性钩子（PHP 8.4+）            | 传统 Getter/Setter              |
|:-------------|:--------------------------------|:--------------------------------|
| **代码量**    | 更少                            | 更多                            |
| **可读性**    | 更好（逻辑集中）                 | 较差（分散在不同方法）           |
| **性能**      | 稍慢（有钩子开销）               | 稍快（直接访问）                 |
| **版本要求**  | PHP 8.4+                        | 所有 PHP 版本                    |
| **IDE 支持**  | 较好                            | 很好                            |
| **适用场景**  | 需要验证、计算的属性             | 简单属性或需要向后兼容           |

### 不对称可见性 vs readonly 属性

| 特性         | 不对称可见性（PHP 8.4+）        | readonly 属性（PHP 8.1+）       |
|:-------------|:--------------------------------|:--------------------------------|
| **灵活性**    | 更灵活（可以有不同的可见性）     | 较固定（只读）                   |
| **语法**      | `public(get) private(set)`      | `public readonly`                |
| **版本要求**  | PHP 8.4+                        | PHP 8.1+                        |
| **适用场景**  | 需要细粒度控制                   | 完全不可变                       |

### 静态方法 vs 实例方法

| 特性         | 静态方法                         | 实例方法                         |
|:-------------|:--------------------------------|:--------------------------------|
| **调用方式**  | `ClassName::method()`            | `$object->method()`              |
| **对象实例**  | 不需要                           | 需要                             |
| **$this**    | 不能使用                         | 可以使用                         |
| **适用场景**  | 工具类、不需要状态的方法          | 需要访问对象状态的方法            |
| **性能**      | 稍快（不需要对象）               | 稍慢（需要对象）                 |

## 练习任务

1. **属性钩子练习**（PHP 8.4+）：创建一个 `User` 类，使用属性钩子为 `email` 属性添加验证和格式化逻辑（转换为小写、trim）。

2. **计算属性练习**（PHP 8.4+）：创建一个 `Rectangle` 类，使用属性钩子实现 `area` 计算属性，设置 `area` 时自动计算 `width` 或 `height`。

3. **不对称可见性练习**（PHP 8.4+）：创建一个 `Product` 类，使用不对称可见性实现 `price` 属性（外部可读，只能通过 `setPrice()` 方法写入）。

4. **静态工具类**：创建一个 `StringHelper` 类，提供静态方法进行字符串处理（`capitalize`、`slug`、`camelCase`）。

5. **静态工厂方法**：创建一个 `User` 类，提供多个静态工厂方法（`create`、`fromArray`、`fromJson`）创建对象。

## 相关章节

- **[3.1.2 可见性修饰符](../chapter-01-classes/section-02-visibility.md)**：了解可见性修饰符的基础知识
- **[3.1.5 静态属性与方法、魔术方法](../chapter-01-classes/section-05-static-magic.md)**：回顾静态成员的知识
- **[2.14.3 PHP 8.4 新特性](../../stage-02-language/chapter-14-php-versions/section-03-php84.md)**：了解 PHP 8.4 的其他新特性
