# 3.2.4 属性钩子与静态方法

## 概述

属性钩子（Property Hooks）是 PHP 8.4+ 引入的新特性，允许在属性的读取和写入操作中定义自定义逻辑。本章还介绍静态方法的合理使用。

## 属性钩子（Property Hooks）（PHP 8.4+）

### 基础语法

- **语法**：在属性定义中使用 `get` 和 `set` 钩子
- **说明**：允许在属性的读取和写入操作中定义自定义逻辑，减少样板代码。

```php
<?php
declare(strict_types=1);

class Temperature
{
    private float $celsius;
    
    // 使用属性钩子自动转换
    public float $fahrenheit
    {
        get => ($this->celsius * 9/5) + 32;
        set (float $fahrenheit) {
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
echo $temp->fahrenheit; // 77
$temp->fahrenheit = 86;
echo $temp->getCelsius(); // 30
```

### 不对称可见性（PHP 8.4+）

- 允许属性的读取和写入具有不同的可见性修饰符。

```php
<?php
declare(strict_types=1);

class BankAccount
{
    private float $balance = 0.0;
    
    // 公有读取，私有写入
    public float $balance
    {
        get => $this->balance;
        private set;
    }
    
    public function deposit(float $amount): void
    {
        $this->balance = $this->balance + $amount; // 可以设置（在类内部）
    }
    
    public function withdraw(float $amount): bool
    {
        if ($this->balance >= $amount) {
            $this->balance = $this->balance - $amount;
            return true;
        }
        return false;
    }
}

$account = new BankAccount();
$account->deposit(100);
echo $account->balance; // 可以读取：100
// $account->balance = 200; // 错误：无法从外部设置
```

### 属性钩子示例

```php
<?php
declare(strict_types=1);

class OrderItem
{
    private int $quantity = 0;
    private float $unitPrice = 0.0;
    
    // 自动计算总价
    public float $total
    {
        get => $this->quantity * $this->unitPrice;
    }
    
    public function __construct(float $unitPrice, int $quantity = 1)
    {
        $this->unitPrice = $unitPrice;
        $this->quantity = $quantity;
    }
    
    public function setQuantity(int $quantity): void
    {
        $this->quantity = $quantity;
    }
    
    public function getQuantity(): int
    {
        return $this->quantity;
    }
}

$item = new OrderItem(10.0, 3);
echo $item->total; // 30.0
$item->setQuantity(5);
echo $item->total; // 50.0
```

## 静态属性与方法

### 静态属性滥用风险

- 静态属性是全局状态，可能导致：
  - 测试困难（难以隔离状态）
  - 并发问题（多线程/多进程环境）
  - 隐藏依赖（难以追踪数据流）

```php
// 不推荐：使用静态属性存储全局状态
class Config
{
    public static string $databaseHost = 'localhost';
    public static string $databaseName = 'app';
}

// 推荐：使用依赖注入
class DatabaseConnection
{
    public function __construct(
        private string $host,
        private string $database
    ) {
    }
}
```

### 静态方法的合理使用

- 适用于：
  - 工具类方法（如 `Math::add()`）
  - 工厂方法（`User::create()`）
  - 单例模式（需谨慎使用）

```php
class DateTimeHelper
{
    public static function formatTimestamp(int $timestamp, string $format = 'Y-m-d H:i:s'): string
    {
        return date($format, $timestamp);
    }

    public static function isWeekend(int $timestamp): bool
    {
        $dayOfWeek = (int) date('w', $timestamp);
        return $dayOfWeek === 0 || $dayOfWeek === 6;
    }
}

echo DateTimeHelper::formatTimestamp(time());
```

### 静态工厂方法

```php
class Email
{
    private function __construct(
        public readonly string $address
    ) {
        if (!filter_var($address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: {$address}");
        }
    }

    public static function fromString(string $address): self
    {
        return new self($address);
    }

    public static function fromArray(array $data): self
    {
        return new self($data['email'] ?? throw new InvalidArgumentException('Email required'));
    }
}

$email = Email::fromString('user@example.com');
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Currency
{
    private float $amount;
    private string $baseCurrency = 'USD';
    
    // 使用属性钩子实现自动转换
    public float $eur
    {
        get => $this->convert('EUR');
        set (float $eur) {
            $this->amount = $this->convertFrom('EUR', $eur);
        }
    }
    
    public float $gbp
    {
        get => $this->convert('GBP');
        set (float $gbp) {
            $this->amount = $this->convertFrom('GBP', $gbp);
        }
    }
    
    public function __construct(float $amount, string $currency = 'USD')
    {
        $this->amount = $amount;
        $this->baseCurrency = $currency;
    }
    
    private function convert(string $to): float
    {
        $rates = [
            'USD' => 1.0,
            'EUR' => 0.85,
            'GBP' => 0.73,
        ];
        return $this->amount * ($rates[$to] / $rates[$this->baseCurrency]);
    }
    
    private function convertFrom(string $from, float $amount): float
    {
        $rates = [
            'USD' => 1.0,
            'EUR' => 0.85,
            'GBP' => 0.73,
        ];
        return $amount * ($rates[$this->baseCurrency] / $rates[$from]);
    }
    
    public function getAmount(string $currency = 'USD'): float
    {
        return $currency === $this->baseCurrency ? $this->amount : $this->convert($currency);
    }
}

$money = new Currency(100, 'USD');
echo $money->eur; // 85
$money->gbp = 100;
echo $money->getAmount('USD'); // 约 136.99
```

## 注意事项

1. **属性钩子限制**：PHP 8.4+ 特性，需要 PHP 8.4 或更高版本。

2. **性能考虑**：属性钩子会增加属性访问的开销，只在必要时使用。

3. **静态属性风险**：避免使用静态属性存储全局状态，优先使用依赖注入。

4. **静态方法适用**：静态方法适用于工具类和工厂方法，不适合需要状态的操作。

5. **测试友好**：避免静态全局状态，提高代码的可测试性。

## 练习

1. **PHP 8.4+**：使用属性钩子创建一个 `Currency` 类，支持多种货币之间的自动转换。

2. **PHP 8.4+**：使用不对称可见性创建一个 `User` 类，`email` 属性可以读取但只能通过验证方法设置。

3. **PHP 8.4+**：使用属性钩子实现一个自动计算总价的 `OrderItem` 类。

4. 创建一个 `Math` 工具类，提供静态方法进行数学运算。

5. 实现一个 `UserFactory` 类，提供静态工厂方法创建不同类型的用户。
