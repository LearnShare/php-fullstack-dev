# 3.1.5 静态属性与方法、魔术方法

## 概述

静态成员和魔术方法是 PHP 面向对象编程的重要特性。静态属性与方法属于类而不是对象实例，所有对象共享同一份静态成员。魔术方法（Magic Methods）是 PHP 提供的特殊方法，以双下划线（`__`）开头，在特定时机自动调用，允许自定义对象的行为。本节详细介绍静态属性与方法、静态工厂方法、常用魔术方法（`__toString`、`__get`、`__set`、`__call`、`__invoke` 等）的语法、使用场景和最佳实践。

理解静态成员和魔术方法对于掌握 PHP 面向对象编程至关重要。静态成员适用于不需要对象实例的场景，如工具类、计数器、配置类等。魔术方法提供了强大的元编程能力，允许自定义对象的访问、调用、字符串转换等行为。

**主要内容**：
- 静态属性的定义、访问和使用
- 静态方法的定义、调用和使用
- 静态成员的作用域和生命周期
- 静态工厂方法
- 常用魔术方法的语法和使用场景
- 魔术方法的调用时机
- 静态成员和魔术方法的最佳实践

## 特性

- **静态成员**：属于类而不是对象实例，所有对象共享
- **静态属性**：类的属性，所有对象共享同一份
- **静态方法**：类的方法，不需要对象实例就可以调用
- **魔术方法**：以 `__` 开头的特殊方法，在特定时机自动调用
- **工厂方法**：使用静态方法创建对象的模式

## 语法/定义

### 静态属性

**语法**：`public static Type $propertyName;`

**访问方式**：
- 类内部：`self::$propertyName` 或 `static::$propertyName`
- 类外部：`ClassName::$propertyName`
- 对象访问（不推荐）：`$object::$propertyName`

**特点**：
- 所有对象共享同一份静态属性
- 属于类，不属于对象实例
- 可以使用可见性修饰符（`public`、`protected`、`private`）

### 静态方法

**语法**：`public static function methodName(): ReturnType { ... }`

**调用方式**：
- 类内部：`self::methodName()` 或 `static::methodName()`
- 类外部：`ClassName::methodName()`
- 对象调用（不推荐）：`$object::methodName()`

**特点**：
- 不需要对象实例就可以调用
- 不能使用 `$this`（因为没有对象实例）
- 可以使用可见性修饰符

### 魔术方法

**命名规则**：以双下划线（`__`）开头

**常用魔术方法**：
- `__toString()`：对象转换为字符串时调用
- `__get(string $name)`：访问不存在的属性时调用
- `__set(string $name, mixed $value)`：设置不存在的属性时调用
- `__call(string $name, array $arguments)`：调用不存在的方法时调用
- `__invoke(...$arguments)`：对象作为函数调用时调用
- `__construct()`：构造函数（已介绍）
- `__destruct()`：析构函数（已介绍）
- `__clone()`：对象克隆时调用（已介绍）

## 基本用法

### 示例 1：静态属性

```php
<?php
declare(strict_types=1);

class Counter
{
    public static int $count = 0;  // 静态属性
    
    public function __construct()
    {
        self::$count++;  // 在构造器中增加计数
    }
    
    public static function getCount(): int
    {
        return self::$count;
    }
}

echo "Initial count: " . Counter::getCount() . "\n";  // 0

$counter1 = new Counter();
echo "After creating counter1: " . Counter::getCount() . "\n";  // 1

$counter2 = new Counter();
echo "After creating counter2: " . Counter::getCount() . "\n";  // 2

// 直接访问静态属性
Counter::$count = 10;
echo "After setting count: " . Counter::getCount() . "\n";  // 10
```

**输出**：

```
Initial count: 0
After creating counter1: 1
After creating counter2: 2
After setting count: 10
```

**说明**：
- 静态属性 `$count` 属于类，所有对象共享
- 使用 `self::$count` 在类内部访问
- 使用 `Counter::$count` 在类外部访问

### 示例 2：静态方法

```php
<?php
declare(strict_types=1);

class Math
{
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
    
    public static function subtract(int $a, int $b): int
    {
        return $a - $b;
    }
    
    public static function multiply(int $a, int $b): int
    {
        return $a * $b;
    }
    
    public static function divide(int $a, int $b): float
    {
        if ($b === 0) {
            throw new DivisionByZeroError("Division by zero");
        }
        return $a / $b;
    }
}

// 不需要创建对象，直接调用静态方法
echo "3 + 4 = " . Math::add(3, 4) . "\n";
echo "10 - 5 = " . Math::subtract(10, 5) . "\n";
echo "6 * 7 = " . Math::multiply(6, 7) . "\n";
echo "15 / 3 = " . Math::divide(15, 3) . "\n";
```

**输出**：

```
3 + 4 = 7
10 - 5 = 5
6 * 7 = 42
15 / 3 = 5
```

**说明**：
- 静态方法不需要对象实例就可以调用
- 使用 `ClassName::methodName()` 调用
- 适合工具类和不需要状态的方法

### 示例 3：静态工厂方法

```php
<?php
declare(strict_types=1);

class User
{
    private function __construct(
        private string $name,
        private string $email,
        private int $age
    ) {}
    
    // 静态工厂方法：创建用户
    public static function create(string $name, string $email, int $age): self
    {
        // 可以在这里添加验证逻辑
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email address");
        }
        
        return new self($name, $email, $age);
    }
    
    // 静态工厂方法：从数组创建
    public static function fromArray(array $data): self
    {
        return new self(
            $data['name'] ?? '',
            $data['email'] ?? '',
            $data['age'] ?? 0
        );
    }
    
    // 静态工厂方法：创建管理员用户
    public static function createAdmin(string $name, string $email): self
    {
        $user = new self($name, $email, 0);
        // 可以设置管理员相关属性
        return $user;
    }
    
    public function getName(): string { return $this->name; }
    public function getEmail(): string { return $this->email; }
    public function getAge(): int { return $this->age; }
}

// 使用静态工厂方法创建对象
$user1 = User::create("John", "john@example.com", 25);
$user2 = User::fromArray(['name' => 'Jane', 'email' => 'jane@example.com', 'age' => 30]);
$admin = User::createAdmin("Admin", "admin@example.com");
```

**输出**：

```
（无输出，示例展示创建对象的方式）
```

**说明**：
- 静态工厂方法提供灵活的对象创建方式
- 可以在工厂方法中添加验证逻辑
- 可以提供多种创建方式（如 `fromArray`、`createAdmin`）

### 示例 4：__toString 魔术方法

```php
<?php
declare(strict_types=1);

class Product
{
    public function __construct(
        private string $name,
        private float $price
    ) {}
    
    public function __toString(): string
    {
        return "Product: {$this->name}, Price: \${$this->price}";
    }
}

$product = new Product("Laptop", 999.99);

// 对象转换为字符串时自动调用 __toString()
echo $product . "\n";  // Product: Laptop, Price: $999.99

// 在字符串上下文中也会调用
$message = "Item: " . $product;
echo $message . "\n";  // Item: Product: Laptop, Price: $999.99
```

**输出**：

```
Product: Laptop, Price: $999.99
Item: Product: Laptop, Price: $999.99
```

**说明**：
- `__toString()` 在对象转换为字符串时自动调用
- 必须返回字符串
- 常用于调试和日志记录

### 示例 5：__get 和 __set 魔术方法

```php
<?php
declare(strict_types=1);

class DynamicProperties
{
    private array $data = [];
    
    // 访问不存在的属性时调用
    public function __get(string $name): mixed
    {
        if (array_key_exists($name, $this->data)) {
            return $this->data[$name];
        }
        
        // 可以返回默认值或抛出异常
        return null;
    }
    
    // 设置不存在的属性时调用
    public function __set(string $name, mixed $value): void
    {
        $this->data[$name] = $value;
    }
    
    // 检查属性是否存在
    public function __isset(string $name): bool
    {
        return isset($this->data[$name]);
    }
    
    // 删除属性
    public function __unset(string $name): void
    {
        unset($this->data[$name]);
    }
}

$obj = new DynamicProperties();

// 动态设置属性
$obj->name = "John";
$obj->age = 25;
$obj->email = "john@example.com";

// 动态访问属性
echo "Name: " . $obj->name . "\n";
echo "Age: " . $obj->age . "\n";
echo "Email: " . $obj->email . "\n";

// 检查属性是否存在
if (isset($obj->name)) {
    echo "Name is set\n";
}

// 删除属性
unset($obj->email);
echo "Email after unset: " . ($obj->email ?? "not set") . "\n";
```

**输出**：

```
Name: John
Age: 25
Email: john@example.com
Name is set
Email after unset: not set
```

**说明**：
- `__get()` 在访问不存在的属性时调用
- `__set()` 在设置不存在的属性时调用
- `__isset()` 在使用 `isset()` 检查属性时调用
- `__unset()` 在使用 `unset()` 删除属性时调用

### 示例 6：__call 魔术方法

```php
<?php
declare(strict_types=1);

class MagicCall
{
    private array $methods = [];
    
    // 调用不存在的方法时调用
    public function __call(string $name, array $arguments): mixed
    {
        // 模拟方法调用
        $methodName = "method_" . $name;
        
        if (method_exists($this, $methodName)) {
            return $this->$methodName(...$arguments);
        }
        
        // 可以记录调用或抛出异常
        echo "Calling undefined method: {$name}\n";
        echo "Arguments: " . implode(", ", $arguments) . "\n";
        
        return null;
    }
    
    // 静态方法调用（调用不存在的静态方法时调用）
    public static function __callStatic(string $name, array $arguments): mixed
    {
        echo "Calling undefined static method: {$name}\n";
        echo "Arguments: " . implode(", ", $arguments) . "\n";
        return null;
    }
    
    private function method_hello(string $name): string
    {
        return "Hello, {$name}!";
    }
}

$obj = new MagicCall();

// 调用不存在的方法，会触发 __call()
$obj->greet("John");
$obj->sayGoodbye("Jane");

// 如果方法存在，可以转发调用
echo $obj->hello("World") . "\n";

// 调用不存在的静态方法
MagicCall::staticMethod("test");
```

**输出**：

```
Calling undefined method: greet
Arguments: John
Calling undefined method: sayGoodbye
Arguments: Jane
Hello, World!
Calling undefined static method: staticMethod
Arguments: test
```

**说明**：
- `__call()` 在调用不存在的实例方法时调用
- `__callStatic()` 在调用不存在的静态方法时调用
- 可以用于实现方法重载、代理模式等

### 示例 7：__invoke 魔术方法

```php
<?php
declare(strict_types=1);

class CallableClass
{
    public function __construct(
        private string $name
    ) {}
    
    // 对象作为函数调用时调用
    public function __invoke(string $message): string
    {
        return "{$this->name} says: {$message}";
    }
}

$callable = new CallableClass("John");

// 对象可以像函数一样调用
echo $callable("Hello, World!") . "\n";

// 可以作为回调函数使用
$callback = $callable;
echo $callback("How are you?") . "\n";

// 可以用于函数式编程
$result = array_map($callable, ["message1", "message2"]);
print_r($result);
```

**输出**：

```
John says: Hello, World!
John says: How are you?
Array
(
    [0] => John says: message1
    [1] => John says: message2
)
```

**说明**：
- `__invoke()` 使对象可以像函数一样调用
- 对象实现了 `__invoke()` 后，就是可调用的（callable）
- 可以用于回调函数、函数式编程等场景

## 使用场景

### 场景 1：工具类

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
    
    public static function camelCase(string $str): string
    {
        $words = explode('_', $str);
        $result = array_shift($words);
        foreach ($words as $word) {
            $result .= ucfirst($word);
        }
        return $result;
    }
    
    public static function slug(string $str): string
    {
        $str = strtolower($str);
        $str = preg_replace('/[^a-z0-9]+/', '-', $str);
        return trim($str, '-');
    }
}

echo StringUtils::capitalize("hello world") . "\n";
echo StringUtils::camelCase("hello_world") . "\n";
echo StringUtils::slug("Hello, World!") . "\n";
```

### 场景 2：计数器

静态属性适合实现计数器，所有对象共享计数。

**示例**：对象计数器

```php
<?php
declare(strict_types=1);

class InstanceCounter
{
    private static int $instanceCount = 0;
    private static array $instances = [];
    
    public function __construct()
    {
        self::$instanceCount++;
        self::$instances[] = $this;
    }
    
    public static function getInstanceCount(): int
    {
        return self::$instanceCount;
    }
    
    public static function getAllInstances(): array
    {
        return self::$instances;
    }
}

$obj1 = new InstanceCounter();
$obj2 = new InstanceCounter();
$obj3 = new InstanceCounter();

echo "Total instances: " . InstanceCounter::getInstanceCount() . "\n";
```

### 场景 3：配置类

静态属性适合存储配置信息。

**示例**：应用配置

```php
<?php
declare(strict_types=1);

class Config
{
    private static array $config = [
        'app_name' => 'My App',
        'version' => '1.0.0',
        'debug' => false
    ];
    
    public static function get(string $key, mixed $default = null): mixed
    {
        return self::$config[$key] ?? $default;
    }
    
    public static function set(string $key, mixed $value): void
    {
        self::$config[$key] = $value;
    }
    
    public static function all(): array
    {
        return self::$config;
    }
}

echo Config::get('app_name') . "\n";
Config::set('debug', true);
var_dump(Config::get('debug'));
```

### 场景 4：动态属性访问

使用 `__get` 和 `__set` 实现动态属性访问。

**示例**：数据容器

```php
<?php
declare(strict_types=1);

class DataContainer
{
    private array $data = [];
    
    public function __get(string $name): mixed
    {
        return $this->data[$name] ?? null;
    }
    
    public function __set(string $name, mixed $value): void
    {
        $this->data[$name] = $value;
    }
    
    public function toArray(): array
    {
        return $this->data;
    }
}

$container = new DataContainer();
$container->name = "John";
$container->age = 25;
print_r($container->toArray());
```

## 注意事项

### 静态成员的限制

#### 1. 静态方法不能使用 $this

静态方法没有对象实例，不能使用 `$this`。

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
        echo "Static method\n";
    }
    
    public function instanceMethod(): void
    {
        echo $this->name . "\n";  // 正确：实例方法可以使用 $this
    }
}
```

#### 2. 静态属性在所有对象间共享

静态属性属于类，所有对象共享同一份。

**示例**：

```php
<?php
declare(strict_types=1);

class Counter
{
    public static int $count = 0;
    
    public function increment(): void
    {
        self::$count++;
    }
}

$counter1 = new Counter();
$counter2 = new Counter();

$counter1->increment();
echo Counter::$count . "\n";  // 1

$counter2->increment();
echo Counter::$count . "\n";  // 2（共享计数）
```

#### 3. 静态成员的生命周期

静态成员在脚本执行期间一直存在，直到脚本结束。

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    public static int $counter = 0;
    
    public function __construct()
    {
        self::$counter++;
    }
}

function testFunction(): void
{
    $obj = new Example();
    echo "Counter in function: " . Example::$counter . "\n";
}

testFunction();
echo "Counter after function: " . Example::$counter . "\n";  // 计数仍然保留
```

### 魔术方法的注意事项

#### 1. 性能考虑

魔术方法有性能开销，应该谨慎使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 性能较差：使用魔术方法
class SlowClass
{
    private array $data = [];
    
    public function __get(string $name): mixed
    {
        return $this->data[$name] ?? null;
    }
}

// 性能较好：直接属性访问
class FastClass
{
    public string $name;
    public int $age;
}
```

#### 2. 调试困难

魔术方法使代码行为不那么明显，可能增加调试难度。

**示例**：

```php
<?php
declare(strict_types=1);

class MagicClass
{
    public function __call(string $name, array $arguments): mixed
    {
        // 方法调用被拦截，调试时不容易发现
        return null;
    }
}
```

## 常见问题

### 问题 1：静态方法中使用 $this

**错误信息**：`Fatal error: Using $this when not in object context`

**原因**：静态方法没有对象实例，不能使用 `$this`

**解决方案**：
- 静态方法不能使用 `$this`
- 如果需要访问实例数据，应该使用实例方法
- 如果需要共享数据，使用静态属性

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    private string $name = "Test";
    
    // 错误：静态方法不能使用 $this
    // public static function badMethod(): void
    // {
    //     echo $this->name;
    // }
    
    // 正确：使用实例方法
    public function goodMethod(): void
    {
        echo $this->name;
    }
    
    // 正确：使用静态属性
    private static string $staticName = "Static";
    
    public static function staticMethod(): void
    {
        echo self::$staticName;  // 使用静态属性
    }
}
```

### 问题 2：静态属性共享导致的问题

**症状**：修改一个对象的静态属性影响所有对象

**原因**：静态属性在所有对象间共享

**解决方案**：
- 理解静态属性的共享特性
- 如果需要独立状态，使用实例属性
- 如果需要共享状态，使用静态属性

**示例**：

```php
<?php
declare(strict_types=1);

// 错误理解：以为每个对象有独立的计数
class BadCounter
{
    public static int $count = 0;
    
    public function __construct()
    {
        self::$count++;
    }
}

// 正确：使用实例属性（如果需要独立计数）
class GoodCounter
{
    private int $count = 0;
    
    public function increment(): void
    {
        $this->count++;
    }
    
    public function getCount(): int
    {
        return $this->count;
    }
}
```

### 问题 3：魔术方法性能问题

**症状**：使用魔术方法的代码运行较慢

**原因**：魔术方法有性能开销

**解决方案**：
- 只在必要时使用魔术方法
- 对于频繁调用的代码，考虑使用直接属性访问
- 可以使用缓存优化魔术方法

## 最佳实践

### 1. 合理使用静态成员

- **工具类**：使用静态方法提供工具函数
- **配置类**：使用静态属性存储配置
- **工厂方法**：使用静态方法创建对象
- **避免过度使用**：不要将所有方法都设为静态

**示例**：

```php
<?php
declare(strict_types=1);

// 好的做法：工具类使用静态方法
class MathUtils
{
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
}

// 不好的做法：有状态的对象使用静态方法
class User
{
    private string $name;
    
    // 不好：应该使用实例方法
    // public static function getName(): string { ... }
    
    // 好：使用实例方法
    public function getName(): string
    {
        return $this->name;
    }
}
```

### 2. 理解魔术方法的调用时机

- `__toString()`：对象转换为字符串时
- `__get()` / `__set()`：访问/设置不存在的属性时
- `__call()` / `__callStatic()`：调用不存在的方法时
- `__invoke()`：对象作为函数调用时

### 3. 使用静态工厂方法

静态工厂方法提供灵活的对象创建方式。

**示例**：

```php
<?php
declare(strict_types=1);

class DateTime
{
    private function __construct(private int $timestamp) {}
    
    public static function now(): self
    {
        return new self(time());
    }
    
    public static function fromTimestamp(int $timestamp): self
    {
        return new self($timestamp);
    }
    
    public static function fromString(string $datetime): self
    {
        return new self(strtotime($datetime));
    }
}
```

### 4. 避免在魔术方法中执行耗时操作

魔术方法可能被频繁调用，应该保持高效。

**示例**：

```php
<?php
declare(strict_types=1);

class EfficientMagic
{
    private array $cache = [];
    private array $data = [];
    
    public function __get(string $name): mixed
    {
        // 使用缓存避免重复计算
        if (isset($this->cache[$name])) {
            return $this->cache[$name];
        }
        
        $value = $this->data[$name] ?? null;
        $this->cache[$name] = $value;
        return $value;
    }
}
```

### 5. 在文档中说明魔术方法的行为

对于使用魔术方法的类，应该在文档中说明行为。

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 数据容器类
 * 
 * 使用 __get 和 __set 实现动态属性访问。
 * 所有属性存储在内部的 $data 数组中。
 */
class DataContainer
{
    private array $data = [];
    
    public function __get(string $name): mixed
    {
        return $this->data[$name] ?? null;
    }
    
    public function __set(string $name, mixed $value): void
    {
        $this->data[$name] = $value;
    }
}
```

## 对比分析

### 静态方法 vs 实例方法

| 特性         | 静态方法                       | 实例方法                       |
|:-------------|:-------------------------------|:-------------------------------|
| **调用方式**  | `ClassName::method()`           | `$object->method()`             |
| **对象实例**  | 不需要                         | 需要                           |
| **$this**    | 不能使用                       | 可以使用                       |
| **适用场景**  | 工具类、不需要状态的方法        | 需要访问对象状态的方法          |
| **性能**      | 稍快（不需要创建对象）          | 稍慢（需要对象实例）            |

### 静态属性 vs 实例属性

| 特性         | 静态属性                       | 实例属性                       |
|:-------------|:-------------------------------|:-------------------------------|
| **存储位置**  | 类级别（所有对象共享）          | 对象级别（每个对象独立）        |
| **访问方式**  | `ClassName::$property`          | `$object->property`             |
| **生命周期**  | 脚本执行期间                   | 对象生命周期                   |
| **适用场景**  | 计数器、配置、共享状态          | 对象状态、实例数据              |

### 魔术方法 vs 普通方法

| 特性         | 魔术方法                       | 普通方法                       |
|:-------------|:-------------------------------|:-------------------------------|
| **命名**      | 以 `__` 开头                    | 自定义名称                      |
| **调用**      | 自动调用（特定时机）            | 手动调用                        |
| **性能**      | 有性能开销                     | 性能较好                        |
| **用途**      | 自定义对象行为                  | 实现业务逻辑                    |

## 练习任务

1. **静态计数器**：创建一个 `Counter` 类，使用静态属性实现对象计数，每次创建对象时计数加 1。

2. **工具类**：创建一个 `StringHelper` 类，提供静态方法进行字符串处理（如：`capitalize`、`slug`、`camelCase`）。

3. **__toString 实现**：创建一个 `Product` 类，实现 `__toString()` 方法，返回产品的详细信息字符串。

4. **动态属性**：创建一个 `Config` 类，使用 `__get` 和 `__set` 实现动态属性访问，所有配置存储在内部数组中。

5. **可调用对象**：创建一个 `Logger` 类，实现 `__invoke()` 方法，使对象可以像函数一样调用进行日志记录。

## 相关章节

- **[3.1.1 类与对象基础](section-01-basics.md)**：回顾类与对象的基础知识
- **[3.1.3 构造函数与析构函数](section-03-constructors-destructors.md)**：了解其他魔术方法（`__construct`、`__destruct`）
- **[2.3.4 全局与静态](../../stage-02-language/chapter-03-variables/section-04-scope-static.md)**：回顾静态变量的概念
