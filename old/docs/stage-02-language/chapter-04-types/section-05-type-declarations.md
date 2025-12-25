# 2.4.5 类型声明与联合类型

## 概述

PHP 7.0+ 引入了类型声明（Type Declarations）功能，允许在函数参数和返回值中指定类型。PHP 8.0+ 进一步支持联合类型（Union Types）和其他高级类型特性。

## 基本类型声明

### 函数参数类型

#### 语法

```php
function functionName(Type $parameter): ReturnType
{
    // 函数体
}
```

#### 示例

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

function greet(string $name): string
{
    return "Hello, {$name}";
}

function calculate(float $price, int $quantity): float
{
    return $price * $quantity;
}
```

### 返回值类型

```php
<?php
declare(strict_types=1);

function getAge(): int
{
    return 25;
}

function getName(): string
{
    return "Alice";
}

function isActive(): bool
{
    return true;
}
```

## 可选参数与默认值

### 可选参数

```php
<?php
declare(strict_types=1);

function greet(string $name, ?string $title = null): string
{
    if ($title !== null) {
        return "Hello, {$title} {$name}";
    }
    return "Hello, {$name}";
}

echo greet("Alice") . "\n";           // Hello, Alice
echo greet("Alice", "Ms.") . "\n";   // Hello, Ms. Alice
```

### 可空类型（Nullable Types）

使用 `?Type` 表示类型可以为 `null`：

```php
<?php
declare(strict_types=1);

function findUser(int $id): ?User
{
    // 如果找到返回 User 对象，否则返回 null
    return $users[$id] ?? null;
}

$user = findUser(1);
if ($user !== null) {
    echo $user->name;
}
```

## 联合类型（PHP 8.0+）

### 基本语法

联合类型允许参数或返回值接受多种类型：

```php
function process(int|string $value): int|string
{
    return $value;
}
```

### 示例

```php
<?php
declare(strict_types=1);

function formatId(int|string $id): string
{
    return (string) $id;
}

function parseValue(int|string|float $value): mixed
{
    if (is_int($value)) {
        return $value * 2;
    } elseif (is_string($value)) {
        return strtoupper($value);
    } else {
        return round($value, 2);
    }
}

echo formatId(42) . "\n";      // 42
echo formatId("123") . "\n";   // 123
```

### 联合类型与 null

```php
<?php
declare(strict_types=1);

// 方式 1：使用 ? 前缀（推荐）
function findUser1(int $id): ?User
{
    return $users[$id] ?? null;
}

// 方式 2：使用联合类型
function findUser2(int $id): User|null
{
    return $users[$id] ?? null;
}

// 方式 3：多个类型
function processValue(int|string|null $value): void
{
    if ($value === null) {
        echo "No value\n";
    } elseif (is_int($value)) {
        echo "Integer: {$value}\n";
    } else {
        echo "String: {$value}\n";
    }
}
```

## 属性类型声明（PHP 7.4+）

### 基本语法

```php
class ClassName
{
    public Type $property;
    private Type $property;
    protected Type $property;
}
```

### 示例

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;
    public int $age;
    public bool $isActive;
    public ?string $email;
    
    public function __construct(string $name, int $age, bool $isActive = true, ?string $email = null)
    {
        $this->name = $name;
        $this->age = $age;
        $this->isActive = $isActive;
        $this->email = $email;
    }
}

$user = new User('Alice', 25, true, 'alice@example.com');
echo $user->name . "\n";
```

### 只读属性（PHP 8.1+）

```php
<?php
declare(strict_types=1);

class Config
{
    public readonly string $appName;
    public readonly string $version;
    
    public function __construct(string $appName, string $version)
    {
        $this->appName = $appName;
        $this->version = $version;
    }
}

$config = new Config('My App', '1.0.0');
// $config->appName = 'New Name';  // 错误：只读属性不能修改
```

## 伪类型

### mixed

`mixed` 类型表示可以接受任何类型（PHP 8.0+）：

```php
<?php
declare(strict_types=1);

function process(mixed $value): mixed
{
    if (is_string($value)) {
        return strtoupper($value);
    } elseif (is_int($value)) {
        return $value * 2;
    }
    return $value;
}
```

### iterable

`iterable` 表示可迭代类型（数组或实现了 `Traversable` 的对象）：

```php
<?php
declare(strict_types=1);

function processItems(iterable $items): array
{
    $result = [];
    foreach ($items as $item) {
        $result[] = $item;
    }
    return $result;
}

processItems([1, 2, 3]);
processItems(new ArrayIterator([1, 2, 3]));
```

### callable

`callable` 表示可调用类型：

```php
<?php
declare(strict_types=1);

function execute(callable $callback, mixed $data): mixed
{
    return $callback($data);
}

$result = execute(fn($x) => $x * 2, 5);
echo $result . "\n";  // 10
```

## 严格模式

### declare(strict_types=1)

在文件开头使用 `declare(strict_types=1);` 启用严格类型检查：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

// 严格模式下会报错
// add("5", 3);  // TypeError

// 必须显式转换
add((int)"5", 3);  // 正确
```

### 严格模式的影响

1. **类型必须完全匹配**：不允许隐式类型转换
2. **必须显式转换**：需要类型转换时，必须使用类型转换操作符
3. **提高代码安全性**：减少因类型错误导致的 bug

## 类型推断（PHP 8.0+）

PHP 8.0+ 改进了类型推断能力：

```php
<?php
declare(strict_types=1);

function process(int|string $value): int
{
    if (is_string($value)) {
        return (int) $value;
    }
    
    // PHP 8.0+ 知道这里 $value 是 int
    return $value;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Product
{
    public function __construct(
        public string $name,
        public float $price,
        public int $stock = 0,
        public ?string $description = null
    ) {}
    
    public function isAvailable(): bool
    {
        return $this->stock > 0;
    }
    
    public function getPriceWithTax(float $taxRate = 0.1): float
    {
        return $this->price * (1 + $taxRate);
    }
}

class Order
{
    private array $items = [];
    
    public function addItem(Product $product, int $quantity): void
    {
        if (!$product->isAvailable()) {
            throw new RuntimeException("Product {$product->name} is not available");
        }
        
        $this->items[] = [
            'product' => $product,
            'quantity' => $quantity
        ];
    }
    
    public function calculateTotal(float $taxRate = 0.1): float
    {
        $total = 0.0;
        foreach ($this->items as $item) {
            $product = $item['product'];
            $quantity = $item['quantity'];
            $total += $product->getPriceWithTax($taxRate) * $quantity;
        }
        return $total;
    }
    
    public function getItems(): array
    {
        return $this->items;
    }
}

// 使用示例
$product1 = new Product('Laptop', 999.99, 10);
$product2 = new Product('Mouse', 29.99, 50);

$order = new Order();
$order->addItem($product1, 1);
$order->addItem($product2, 2);

echo "Total: $" . number_format($order->calculateTotal(), 2) . "\n";
```

## 注意事项

1. **严格模式**：始终在文件开头使用 `declare(strict_types=1);`

2. **类型提示位置**：类型提示只能用于函数参数和返回值，不能用于变量

3. **联合类型限制**：不能使用 `false` 或 `true` 作为联合类型的一部分（PHP 8.2+ 支持 `true` 和 `false` 作为独立类型）

4. **性能影响**：类型声明在运行时检查，对性能影响很小

5. **向后兼容**：类型声明不会破坏向后兼容性，未提供类型时仍可正常工作

## 练习

1. 创建一个 `Calculator` 类，使用类型声明实现基本的数学运算方法。

2. 编写一个函数 `formatValue(int|string|float $value): string`，根据类型格式化输出。

3. 实现一个 `Validator` 类，使用类型声明验证不同类型的输入值。

4. 创建一个 `DataProcessor` 类，使用联合类型处理多种数据类型的输入。

5. 编写一个函数，演示严格模式和非严格模式下的类型转换差异。
