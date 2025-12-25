# 3.1.3 构造函数与析构函数

## 概述

构造函数和析构函数是对象的生命周期管理方法。构造函数在对象创建时自动调用，用于初始化；析构函数在对象销毁时自动调用，用于清理资源。

## 构造函数（Constructor）

### `__construct` 方法

- **语法**：`public function __construct(参数列表) { ... }`
- 对象创建时自动调用，用于初始化对象属性。
- 支持参数类型声明与默认值。

```php
class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {
        // 构造函数体（可选）
    }
}

$user = new User(1, 'Alice', 'alice@example.com');
```

### 构造器属性提升（PHP 8.0+）

PHP 8.0+ 支持构造器属性提升，简化代码：

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

### 混合使用

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
}
```

### 可见性修饰符

构造器属性提升支持所有可见性修饰符：

```php
class BankAccount
{
    public function __construct(
        public int $accountNumber,
        protected float $balance,
        private string $pin
    ) {
    }
}
```

### 构造函数示例

```php
class Product
{
    private string $name;
    private float $price;
    private int $stock;

    public function __construct(string $name, float $price, int $stock = 0)
    {
        $this->name = $name;
        $this->price = max(0, $price);
        $this->stock = max(0, $stock);
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

$product = new Product('Laptop', 999.99, 10);
```

### 构造函数中的验证

```php
class Email
{
    private string $address;

    public function __construct(string $address)
    {
        if (!filter_var($address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: {$address}");
        }
        $this->address = $address;
    }

    public function getAddress(): string
    {
        return $this->address;
    }
}
```

## 析构函数（Destructor）

### `__destruct` 方法

- **语法**：`public function __destruct() { ... }`
- 对象被销毁时自动调用（引用计数为 0 或脚本结束时）。
- 常用于释放资源（关闭文件、断开连接等）。

```php
class FileHandler
{
    private $handle;

    public function __construct(string $filename)
    {
        $this->handle = fopen($filename, 'r');
        if ($this->handle === false) {
            throw new RuntimeException("Cannot open file: {$filename}");
        }
    }

    public function __destruct()
    {
        if (is_resource($this->handle)) {
            fclose($this->handle);
        }
    }

    public function read(): string
    {
        return fread($this->handle, 8192);
    }
}
```

### 析构函数使用场景

```php
class DatabaseConnection
{
    private ?PDO $connection = null;

    public function __construct(string $dsn, string $user, string $pass)
    {
        $this->connection = new PDO($dsn, $user, $pass);
    }

    public function __destruct()
    {
        $this->connection = null;  // 关闭连接
    }

    public function query(string $sql): array
    {
        return $this->connection->query($sql)->fetchAll();
    }
}
```

### 析构函数注意事项

```php
class Logger
{
    private array $logs = [];

    public function log(string $message): void
    {
        $this->logs[] = $message;
    }

    public function __destruct()
    {
        // 在对象销毁时保存日志
        if (!empty($this->logs)) {
            file_put_contents('app.log', implode("\n", $this->logs), FILE_APPEND);
        }
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ResourceManager
{
    private array $resources = [];

    public function __construct()
    {
        echo "ResourceManager created\n";
    }

    public function acquireResource(string $name): void
    {
        $this->resources[] = $name;
        echo "Resource '{$name}' acquired\n";
    }

    public function __destruct()
    {
        foreach ($this->resources as $resource) {
            echo "Resource '{$resource}' released\n";
        }
        echo "ResourceManager destroyed\n";
    }
}

// 使用
$manager = new ResourceManager();
$manager->acquireResource('Database');
$manager->acquireResource('Cache');
// 脚本结束时自动调用析构函数
```

## 注意事项

1. **构造函数调用时机**：对象创建时自动调用，不能手动调用。

2. **析构函数调用时机**：对象引用计数为 0 或脚本结束时自动调用。

3. **资源管理**：析构函数用于清理资源，但不应该依赖析构函数进行关键操作（如保存数据）。

4. **异常处理**：析构函数中抛出的异常可能导致问题，应该避免在析构函数中抛出异常。

5. **性能考虑**：析构函数会增加对象销毁的开销，只在必要时使用。

## 练习

1. 创建一个 `FileReader` 类，在构造函数中打开文件，在析构函数中关闭文件。

2. 实现一个 `SessionManager` 类，在构造函数中启动会话，在析构函数中保存会话数据。

3. 创建一个 `Cache` 类，在析构函数中将缓存数据写入文件。

4. 实现一个 `DatabaseTransaction` 类，在构造函数中开始事务，在析构函数中根据状态提交或回滚事务。

5. 创建一个 `Logger` 类，在析构函数中将日志缓冲区的内容写入日志文件。
