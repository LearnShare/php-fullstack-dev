# 3.1.3 构造函数与析构函数

## 概述

构造函数和析构函数是管理对象生命周期的重要机制。构造函数（Constructor）在对象创建时自动调用，用于初始化对象的属性；析构函数（Destructor）在对象销毁时自动调用，用于清理资源和释放内存。

理解构造函数和析构函数对于掌握面向对象编程至关重要。构造函数确保对象在创建时处于有效状态，析构函数确保资源得到正确释放，避免内存泄漏和资源泄漏。PHP 8.0+ 引入了构造器属性提升（Constructor Property Promotion），可以简化属性定义和初始化的代码。

**主要内容**：
- 构造函数的定义、调用时机和作用
- 构造器属性提升（PHP 8.0+）的语法和使用
- 析构函数的定义、调用时机和作用
- 资源管理（文件、数据库连接等）
- 对象初始化最佳实践
- 析构函数的使用场景和注意事项

## 特性

- **构造函数 `__construct`**：对象创建时自动调用，用于初始化对象属性
- **析构函数 `__destruct`**：对象销毁时自动调用，用于清理资源
- **构造器属性提升**：PHP 8.0+ 特性，简化属性定义和初始化
- **自动调用**：构造函数和析构函数都是自动调用，不需要手动调用
- **生命周期管理**：构造函数和析构函数共同管理对象的完整生命周期

## 语法/定义

### 构造函数

**语法**：`public function __construct(参数列表) { ... }`

**组成部分**：
- `__construct`：构造函数的特殊方法名，使用双下划线前缀
- 参数列表：可以包含任意数量的参数，支持类型声明和默认值
- 方法体：包含初始化逻辑

**特点**：
- 在对象创建时自动调用
- 方法名必须是 `__construct`（PHP 5.0+）
- 可以定义多个参数，支持类型声明
- 可以使用可见性修饰符（通常是 `public`）

### 构造器属性提升（PHP 8.0+）

**语法**：`public function __construct(public Type $property) { ... }`

**组成部分**：
- 可见性修饰符（`public`、`protected`、`private`）
- 类型声明（可选）
- 属性名
- 参数可以包含默认值

**特点**：
- PHP 8.0+ 引入的新特性
- 同时完成属性声明、参数定义和属性初始化
- 可以混合使用提升属性和普通属性
- 支持所有可见性修饰符

### 析构函数

**语法**：`public function __destruct() { ... }`

**组成部分**：
- `__destruct`：析构函数的特殊方法名
- 无参数：析构函数不能定义参数
- 方法体：包含清理资源的逻辑

**特点**：
- 在对象销毁时自动调用
- 方法名必须是 `__destruct`
- 不能定义参数
- 主要用于释放资源

## 基本用法

### 示例 1：基本构造函数

```php
<?php
declare(strict_types=1);

class User
{
    private string $name;
    private int $age;
    private string $email;
    
    public function __construct(string $name, int $age, string $email)
    {
        $this->name = $name;
        $this->age = $age;
        $this->email = $email;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
}

// 创建对象时自动调用构造函数
$user = new User("John Doe", 25, "john@example.com");
echo "Name: " . $user->getName() . "\n";
echo "Age: " . $user->getAge() . "\n";
echo "Email: " . $user->getEmail() . "\n";
```

**输出**：

```
Name: John Doe
Age: 25
Email: john@example.com
```

**说明**：
- 使用 `new User(...)` 创建对象时，自动调用构造函数
- 构造函数参数用于初始化对象的属性
- 确保对象创建时属性已经被正确初始化

### 示例 2：构造器属性提升（PHP 8.0+）

```php
<?php
declare(strict_types=1);

// 传统方式
class UserTraditional
{
    private string $name;
    private int $age;
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }
}

// 构造器属性提升方式（PHP 8.0+）
class User
{
    public function __construct(
        private string $name,
        private int $age
    ) {}
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
}

$user = new User("Jane Doe", 30);
echo "Name: " . $user->getName() . "\n";
echo "Age: " . $user->getAge() . "\n";
```

**输出**：

```
Name: Jane Doe
Age: 30
```

**说明**：
- 构造器属性提升简化了代码，不需要单独声明属性
- 参数自动成为类的属性
- 可以使用可见性修饰符（`public`、`protected`、`private`）

### 示例 3：混合使用提升属性和普通属性

```php
<?php
declare(strict_types=1);

class Product
{
    private float $discount = 0.0;  // 普通属性，带默认值
    
    public function __construct(
        private string $name,
        private float $price,
        private int $stock = 0  // 提升属性，带默认值
    ) {
        // 可以在构造函数体中执行额外逻辑
        if ($this->price < 0) {
            throw new InvalidArgumentException("Price cannot be negative");
        }
    }
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getPrice(): float
    {
        return $this->price;
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

$product = new Product("Laptop", 999.99, 10);
echo "Product: " . $product->getName() . "\n";
echo "Price: " . $product->getPrice() . "\n";

$product->applyDiscount(0.1);  // 10% 折扣
echo "Final Price: " . $product->getFinalPrice() . "\n";
```

**输出**：

```
Product: Laptop
Price: 999.99
Final Price: 899.991
```

**说明**：
- 可以混合使用提升属性和普通属性
- 提升属性可以在参数列表中使用可见性修饰符
- 构造函数体中可以执行额外的初始化逻辑

### 示例 4：构造函数中的参数验证

```php
<?php
declare(strict_types=1);

class Email
{
    private string $address;
    
    public function __construct(string $address)
    {
        if (!filter_var($address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email address: {$address}");
        }
        $this->address = $address;
    }
    
    public function getAddress(): string
    {
        return $this->address;
    }
}

// 有效邮箱
$email = new Email("user@example.com");
echo "Email: " . $email->getAddress() . "\n";

// 无效邮箱（会抛出异常）
try {
    $invalidEmail = new Email("invalid-email");
} catch (InvalidArgumentException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

**输出**：

```
Email: user@example.com
Error: Invalid email address: invalid-email
```

**说明**：
- 构造函数中可以添加参数验证
- 如果验证失败，可以抛出异常
- 构造函数抛出异常会阻止对象创建

### 示例 5：基本析构函数

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

echo "=== Creating ResourceManager ===\n";
$manager = new ResourceManager();
$manager->acquireResource('Database');
$manager->acquireResource('Cache');
echo "=== End of script ===\n";
// 脚本结束时自动调用析构函数
```

**输出**：

```
=== Creating ResourceManager ===
ResourceManager created
Resource 'Database' acquired
Resource 'Cache' acquired
=== End of script ===
Resource 'Database' released
Resource 'Cache' released
ResourceManager destroyed
```

**说明**：
- 对象销毁时（脚本结束或引用计数为 0）自动调用析构函数
- 析构函数用于清理资源
- 可以在此处执行清理操作

### 示例 6：文件资源管理

```php
<?php
declare(strict_types=1);

class FileHandler
{
    private $handle;
    private string $filename;
    
    public function __construct(string $filename, string $mode = 'r')
    {
        $this->filename = $filename;
        $this->handle = fopen($filename, $mode);
        
        if ($this->handle === false) {
            throw new RuntimeException("Cannot open file: {$filename}");
        }
        
        echo "File '{$filename}' opened\n";
    }
    
    public function read(int $length = 1024): string|false
    {
        if (!is_resource($this->handle)) {
            return false;
        }
        return fread($this->handle, $length);
    }
    
    public function close(): void
    {
        if (is_resource($this->handle)) {
            fclose($this->handle);
            $this->handle = null;
            echo "File '{$this->filename}' closed manually\n";
        }
    }
    
    public function __destruct()
    {
        if (is_resource($this->handle)) {
            fclose($this->handle);
            echo "File '{$this->filename}' closed in destructor\n";
        }
    }
}

// 创建文件处理器
$file = new FileHandler('php://memory', 'w+');
fwrite($file->handle, "Hello, World!");

// 手动关闭（可选）
// $file->close();

// 脚本结束时会自动调用析构函数关闭文件
echo "Script ending...\n";
```

**输出**：

```
File 'php://memory' opened
Script ending...
File 'php://memory' closed in destructor
```

**说明**：
- 析构函数确保文件资源得到释放
- 即使忘记手动关闭，析构函数也会自动关闭
- 这是资源管理的最佳实践

## 使用场景

### 场景 1：对象初始化

构造函数用于确保对象创建时属性已经被正确初始化。

**示例**：用户对象初始化

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        private string $name,
        private string $email,
        private int $age
    ) {
        // 验证年龄
        if ($age < 0 || $age > 150) {
            throw new InvalidArgumentException("Invalid age: {$age}");
        }
    }
    
    public function getName(): string { return $this->name; }
    public function getEmail(): string { return $this->email; }
    public function getAge(): int { return $this->age; }
}

$user = new User("John", "john@example.com", 25);
```

### 场景 2：资源获取

构造函数用于获取资源（文件句柄、数据库连接等）。

**示例**：数据库连接

```php
<?php
declare(strict_types=1);

class DatabaseConnection
{
    private ?PDO $connection = null;
    
    public function __construct(
        private string $dsn,
        private string $username,
        private string $password
    ) {
        try {
            $this->connection = new PDO($dsn, $username, $password);
            $this->connection->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (PDOException $e) {
            throw new RuntimeException("Database connection failed: " . $e->getMessage());
        }
    }
    
    public function getConnection(): PDO
    {
        return $this->connection;
    }
    
    public function __destruct()
    {
        $this->connection = null;  // 关闭连接
    }
}
```

### 场景 3：资源释放

析构函数用于释放资源，防止资源泄漏。

**示例**：临时文件清理

```php
<?php
declare(strict_types=1);

class TemporaryFile
{
    private string $filename;
    
    public function __construct(string $content)
    {
        $this->filename = tempnam(sys_get_temp_dir(), 'tmp_');
        file_put_contents($this->filename, $content);
        echo "Temporary file created: {$this->filename}\n";
    }
    
    public function getFilename(): string
    {
        return $this->filename;
    }
    
    public function __destruct()
    {
        if (file_exists($this->filename)) {
            unlink($this->filename);
            echo "Temporary file deleted: {$this->filename}\n";
        }
    }
}

$tempFile = new TemporaryFile("Hello, World!");
// 脚本结束时会自动删除临时文件
```

## 注意事项

### 构造函数注意事项

#### 1. 构造函数调用时机

- 构造函数在对象创建时自动调用
- 不能手动调用构造函数
- 如果构造函数抛出异常，对象创建会失败

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    public function __construct()
    {
        echo "Constructor called\n";
    }
}

$obj = new Example();  // 自动调用构造函数
// $obj->__construct();  // 错误：不能手动调用构造函数
```

#### 2. 构造函数中的异常

构造函数中抛出异常会阻止对象创建，必须处理这些异常。

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(private string $email)
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email");
        }
    }
}

try {
    $user = new User("invalid-email");  // 抛出异常
} catch (InvalidArgumentException $e) {
    echo "Failed to create user: " . $e->getMessage() . "\n";
}
```

#### 3. 构造器属性提升的限制

- 提升属性不能有默认值（除了参数默认值）
- 提升属性不能使用 `readonly` 修饰符（在 PHP 8.2+ 中可以）
- 提升属性必须同时声明类型

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    // 正确：带默认值的参数
    public function __construct(
        private string $name = "Default"
    ) {}
    
    // 错误：不能这样写
    // private string $name = "Default";  // 提升属性不能在类体中初始化
}
```

### 析构函数注意事项

#### 1. 析构函数调用时机

- 对象引用计数为 0 时调用
- 脚本结束时调用
- 调用顺序不确定（不要依赖调用顺序）

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    private string $name;
    
    public function __construct(string $name)
    {
        $this->name = $name;
        echo "Created: {$this->name}\n";
    }
    
    public function __destruct()
    {
        echo "Destroyed: {$this->name}\n";
    }
}

$obj1 = new Example("Object 1");
$obj2 = new Example("Object 2");

// 手动释放引用
unset($obj1);  // 析构函数可能在这里调用
echo "After unset\n";

// 脚本结束时，$obj2 的析构函数会被调用
```

**输出**（顺序可能不同）：

```
Created: Object 1
Created: Object 2
Destroyed: Object 1
After unset
Destroyed: Object 2
```

#### 2. 析构函数中不应执行耗时操作

析构函数在对象销毁时调用，执行耗时操作会影响性能。

**示例**：

```php
<?php
declare(strict_types=1);

class BadExample
{
    public function __destruct()
    {
        // 不好：在析构函数中执行耗时操作
        sleep(5);  // 阻塞 5 秒
        // 应该避免这样的操作
    }
}

class GoodExample
{
    private array $logs = [];
    
    public function log(string $message): void
    {
        $this->logs[] = $message;
    }
    
    public function save(): void
    {
        // 好的做法：在需要时手动保存，而不是在析构函数中
        file_put_contents('logs.txt', implode("\n", $this->logs));
    }
    
    public function __destruct()
    {
        // 只在必要时保存（简单操作）
        if (!empty($this->logs)) {
            file_put_contents('logs.txt', implode("\n", $this->logs), FILE_APPEND);
        }
    }
}
```

#### 3. 析构函数中的异常处理

析构函数中抛出的异常可能导致问题，应该避免或捕获异常。

**示例**：

```php
<?php
declare(strict_types=1);

class SafeDestructor
{
    public function __destruct()
    {
        try {
            // 可能抛出异常的操作
            $this->cleanup();
        } catch (Exception $e) {
            // 记录错误，但不抛出异常
            error_log("Destructor error: " . $e->getMessage());
        }
    }
    
    private function cleanup(): void
    {
        // 清理逻辑
    }
}
```

## 常见问题

### 问题 1：资源泄漏

**症状**：文件句柄、数据库连接等资源未释放

**原因**：
- 忘记在析构函数中释放资源
- 析构函数未被调用（循环引用）

**解决方案**：
- 在析构函数中释放所有资源
- 提供手动释放资源的方法
- 使用 try-finally 确保资源释放

**示例**：

```php
<?php
declare(strict_types=1);

class ResourceHandler
{
    private $resource;
    
    public function __construct()
    {
        $this->resource = fopen('php://memory', 'r+');
    }
    
    public function close(): void
    {
        if (is_resource($this->resource)) {
            fclose($this->resource);
            $this->resource = null;
        }
    }
    
    public function __destruct()
    {
        $this->close();  // 确保资源释放
    }
}
```

### 问题 2：构造函数异常处理

**症状**：构造函数抛出异常导致对象创建失败

**原因**：
- 构造函数中的验证失败
- 资源获取失败

**解决方案**：
- 使用 try-catch 捕获异常
- 使用工厂方法封装对象创建逻辑
- 提供错误处理机制

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(private string $email)
    {
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email");
        }
    }
    
    // 工厂方法：提供更友好的错误处理
    public static function create(string $email): ?self
    {
        try {
            return new self($email);
        } catch (InvalidArgumentException $e) {
            error_log("Failed to create user: " . $e->getMessage());
            return null;
        }
    }
}

$user = User::create("invalid-email");  // 返回 null，不抛出异常
```

### 问题 3：构造器属性提升的可见性混淆

**症状**：不理解提升属性的可见性修饰符

**原因**：
- 提升属性的可见性修饰符影响属性的访问权限
- 混淆参数可见性和属性可见性

**解决方案**：
- 理解可见性修饰符的作用（详细说明见 [3.1.2 可见性修饰符](section-02-visibility.md)）
- 根据需求选择合适的可见性

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    // public 提升属性：外部可以访问
    public function __construct(
        public string $publicProp
    ) {}
    
    // protected 提升属性：子类可以访问
    public function __construct(
        protected string $protectedProp
    ) {}
    
    // private 提升属性：只有类内部可以访问
    public function __construct(
        private string $privateProp
    ) {}
}
```

### 问题 4：析构函数未被调用

**症状**：析构函数中的清理代码未执行

**原因**：
- 循环引用导致引用计数不为 0
- 对象被全局变量引用

**解决方案**：
- 避免循环引用
- 手动释放引用（`unset()`）
- 使用弱引用（WeakReference）打破循环引用

**示例**：

```php
<?php
declare(strict_types=1);

class ParentClass
{
    private ?ChildClass $child = null;
    
    public function setChild(ChildClass $child): void
    {
        $this->child = $child;
    }
    
    public function __destruct()
    {
        echo "Parent destructed\n";
        // 手动断开引用，避免循环引用
        $this->child = null;
    }
}

class ChildClass
{
    private ?ParentClass $parent = null;
    
    public function setParent(ParentClass $parent): void
    {
        $this->parent = $parent;
    }
    
    public function __destruct()
    {
        echo "Child destructed\n";
    }
}

$parent = new ParentClass();
$child = new ChildClass();
$parent->setChild($child);
$child->setParent($parent);

// 手动断开引用
unset($parent, $child);  // 确保析构函数被调用
```

## 最佳实践

### 1. 使用构造器属性提升简化代码

构造器属性提升（PHP 8.0+）可以显著简化代码，优先使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的做法：使用构造器属性提升
class User
{
    public function __construct(
        private string $name,
        private string $email,
        private int $age
    ) {}
}

// 避免：传统方式（除非需要额外逻辑）
class UserTraditional
{
    private string $name;
    private string $email;
    private int $age;
    
    public function __construct(string $name, string $email, int $age)
    {
        $this->name = $name;
        $this->email = $email;
        $this->age = $age;
    }
}
```

### 2. 在构造函数中进行参数验证

确保对象创建时数据有效，在构造函数中进行验证。

**示例**：

```php
<?php
declare(strict_types=1);

class Product
{
    public function __construct(
        private string $name,
        private float $price,
        private int $stock = 0
    ) {
        if (empty(trim($this->name))) {
            throw new InvalidArgumentException("Product name cannot be empty");
        }
        
        if ($this->price < 0) {
            throw new InvalidArgumentException("Price cannot be negative");
        }
        
        if ($this->stock < 0) {
            throw new InvalidArgumentException("Stock cannot be negative");
        }
    }
}
```

### 3. 在析构函数中释放资源

确保所有资源在对象销毁时得到释放。

**示例**：

```php
<?php
declare(strict_types=1);

class FileLogger
{
    private $fileHandle;
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
        $this->fileHandle = fopen($filename, 'a');
        
        if ($this->fileHandle === false) {
            throw new RuntimeException("Cannot open log file: {$filename}");
        }
    }
    
    public function log(string $message): void
    {
        fwrite($this->fileHandle, $message . "\n");
    }
    
    public function __destruct()
    {
        if (is_resource($this->fileHandle)) {
            fclose($this->fileHandle);
        }
    }
}
```

### 4. 提供手动资源释放方法

除了析构函数，提供手动释放资源的方法，允许提前释放。

**示例**：

```php
<?php
declare(strict_types=1);

class DatabaseConnection
{
    private ?PDO $connection = null;
    
    public function __construct(string $dsn, string $user, string $pass)
    {
        $this->connection = new PDO($dsn, $user, $pass);
    }
    
    public function close(): void
    {
        $this->connection = null;  // 手动关闭连接
    }
    
    public function __destruct()
    {
        $this->close();  // 确保在析构函数中也关闭
    }
}
```

### 5. 避免在析构函数中执行关键操作

析构函数调用时机不确定，不应依赖析构函数执行关键操作（如保存数据）。

**示例**：

```php
<?php
declare(strict_types=1);

// 不好的做法：依赖析构函数保存数据
class BadDataSaver
{
    private array $data = [];
    
    public function addData(string $item): void
    {
        $this->data[] = $item;
    }
    
    public function __destruct()
    {
        // 不好：依赖析构函数保存数据
        file_put_contents('data.txt', implode("\n", $this->data));
    }
}

// 好的做法：提供明确的保存方法
class GoodDataSaver
{
    private array $data = [];
    
    public function addData(string $item): void
    {
        $this->data[] = $item;
    }
    
    public function save(): void
    {
        // 好的：明确调用保存方法
        file_put_contents('data.txt', implode("\n", $this->data));
    }
    
    public function __destruct()
    {
        // 只在必要时保存（作为后备）
        if (!empty($this->data)) {
            file_put_contents('data.txt', implode("\n", $this->data), FILE_APPEND);
        }
    }
}
```

## 对比分析

### 构造函数 vs 普通方法

| 特性         | 构造函数                     | 普通方法                       |
|:-------------|:-----------------------------|:-------------------------------|
| **调用方式**  | 自动调用（对象创建时）        | 手动调用                        |
| **方法名**    | 必须是 `__construct`          | 自定义名称                      |
| **调用时机**  | 对象创建时                    | 任何时候                        |
| **返回值**    | 不能返回值（隐式返回对象）    | 可以返回值                      |
| **用途**      | 初始化对象                    | 执行操作                        |

### 构造器属性提升 vs 传统方式

| 特性         | 构造器属性提升（PHP 8.0+）    | 传统方式                        |
|:-------------|:-------------------------------|:--------------------------------|
| **代码量**    | 更少                           | 更多                            |
| **可读性**    | 更好                           | 较差                            |
| **灵活性**    | 限制更多                       | 更灵活                          |
| **适用场景**  | 简单属性初始化                 | 需要额外初始化逻辑               |

**示例对比**：

```php
<?php
declare(strict_types=1);

// 构造器属性提升方式（简洁）
class UserModern
{
    public function __construct(
        private string $name,
        private int $age
    ) {}
}

// 传统方式（需要更多代码）
class UserTraditional
{
    private string $name;
    private int $age;
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }
}
```

### 构造函数 vs 析构函数

| 特性         | 构造函数                       | 析构函数                        |
|:-------------|:-------------------------------|:--------------------------------|
| **调用时机**  | 对象创建时                     | 对象销毁时                      |
| **参数**      | 可以有参数                     | 不能有参数                      |
| **用途**      | 初始化对象                     | 清理资源                        |
| **异常**      | 可以抛出异常                   | 应避免抛出异常                  |
| **返回值**    | 隐式返回对象                   | 无返回值                        |

## 练习任务

1. **创建图书类**：定义一个 `Book` 类，使用构造函数初始化 `title`、`author`、`price`、`stock` 属性，在构造函数中验证价格和库存不能为负数。

2. **创建文件处理器类**：定义一个 `FileProcessor` 类，构造函数打开文件，析构函数关闭文件，确保文件资源得到正确释放。

3. **使用构造器属性提升**：重构一个使用传统构造函数的类，改为使用构造器属性提升（PHP 8.0+），比较代码差异。

4. **资源管理练习**：创建一个 `ResourcePool` 类，在构造函数中获取多个资源，在析构函数中释放所有资源。

5. **异常处理练习**：创建一个 `Email` 类，构造函数验证邮箱格式，如果格式无效则抛出异常。编写代码处理这个异常。

## 相关章节

- **[3.1.1 类与对象基础](section-01-basics.md)**：回顾类与对象的基础知识
- **[3.1.2 可见性修饰符](section-02-visibility.md)**：了解属性的可见性修饰符
- **[3.2.1 构造器属性提升](../chapter-02-modern-oop/section-01-constructor-promotion.md)**：深入了解构造器属性提升的详细特性
