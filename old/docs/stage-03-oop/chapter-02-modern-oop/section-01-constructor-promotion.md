# 3.2.1 构造器属性提升

## 概述

构造器属性提升（Constructor Property Promotion）是 PHP 8.0+ 引入的语法糖，可以大幅减少样板代码，提升代码可读性。

## 基础语法

### 传统写法 vs 构造器属性提升

```php
// 传统写法
class User
{
    public int $id;
    public string $name;
    public string $email;

    public function __construct(int $id, string $name, string $email)
    {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
    }
}

// PHP 8.0+ 构造器属性提升
class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {
    }
}
```

### 语法说明

- **语法**：在构造函数参数前添加可见性修饰符（`public`、`protected`、`private`）
- PHP 会自动创建属性并赋值
- 减少样板代码，提升可读性

### 优势

1. **代码简洁**：减少重复的属性声明和赋值代码
2. **可读性强**：属性定义和初始化在一起
3. **类型安全**：支持类型声明

## 可见性修饰符

### 支持所有可见性

```php
class BankAccount
{
    public function __construct(
        public int $accountNumber,
        protected float $balance,
        private string $pin
    ) {
    }

    public function getBalance(): float
    {
        return $this->balance;
    }

    public function verifyPin(string $input): bool
    {
        return $this->pin === $input;
    }
}
```

### 可见性示例

```php
class Product
{
    public function __construct(
        public string $name,        // 公开属性
        protected float $price,     // 受保护属性
        private int $stock = 0      // 私有属性，带默认值
    ) {
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getPrice(): float
    {
        return $this->price;
    }

    public function getStock(): int
    {
        return $this->stock;
    }
}
```

## 混合使用

### 提升属性与普通属性

可以混合使用提升属性和普通属性：

```php
class Product
{
    private float $discount = 0.0;

    public function __construct(
        public string $name,
        public float $price,
        public int $stock = 0
    ) {
        // 可以在这里执行额外逻辑
        if ($this->price < 0) {
            throw new InvalidArgumentException('Price cannot be negative');
        }
    }

    public function applyDiscount(float $rate): void
    {
        $this->discount = $rate;
    }

    public function getFinalPrice(): float
    {
        return $this->price * (1 - $this->discount);
    }
}
```

## 默认值

### 参数默认值

```php
class Config
{
    public function __construct(
        public string $environment = 'production',
        public bool $debug = false,
        public int $timeout = 30
    ) {
    }
}

$config1 = new Config();
$config2 = new Config('development', true, 60);
```

## 类型声明

### 支持所有类型

```php
class User
{
    public function __construct(
        public int $id,
        public string $name,
        public array $tags = [],
        public ?string $email = null,
        public bool $active = true
    ) {
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Order
{
    private float $total = 0.0;

    public function __construct(
        public int $id,
        public string $customerName,
        public array $items = [],
        public string $status = 'pending'
    ) {
        $this->calculateTotal();
    }

    private function calculateTotal(): void
    {
        foreach ($this->items as $item) {
            $this->total += $item['price'] * $item['quantity'];
        }
    }

    public function getTotal(): float
    {
        return $this->total;
    }
}

$order = new Order(
    1,
    'Alice',
    [
        ['name' => 'Product A', 'price' => 10.0, 'quantity' => 2],
        ['name' => 'Product B', 'price' => 20.0, 'quantity' => 1]
    ]
);

echo $order->customerName; // Alice
echo $order->getTotal(); // 40.0
```

## 注意事项

1. **属性自动创建**：使用构造器属性提升时，属性会自动创建，无需单独声明。

2. **类型声明**：建议始终使用类型声明，提高代码安全性。

3. **默认值**：可以在参数中设置默认值，简化对象创建。

4. **构造函数体**：可以在构造函数体中执行额外逻辑，如验证、计算等。

5. **可见性**：根据需求选择合适的可见性修饰符。

## 练习

1. 使用构造器属性提升重构一个现有的类，比较代码行数的减少。

2. 创建一个 `Product` 类，使用构造器属性提升定义 `name`、`price`、`stock` 属性。

3. 实现一个 `User` 类，混合使用提升属性和普通属性（如 `passwordHash` 设为私有）。

4. 创建一个 `Config` 类，使用构造器属性提升，所有属性都有默认值。

5. 实现一个 `Order` 类，使用构造器属性提升，在构造函数中计算订单总额。
